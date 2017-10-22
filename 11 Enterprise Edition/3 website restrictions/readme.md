# Describe how to implement, customize, and troubleshoot Enterprise Edition website restrictions

Website Restrictions используется для ограничений доступа к сайту, сайт можно сделать полностью недоступным, доступным тольо для зарегистированных пользователей (с регистрацией и без). Модуль — `Enterprise_WebsiteRestriction`

Модуль работает так:

1. на фронте отлавливаетя ивент `controller_action_predispatch`
```xml
<frontend>
  <events>
    <controller_action_predispatch>
      <observers>
        <enterprise_websiterestriction>
          <class>enterprise_websiterestriction/observer</class>
          <method>restrictWebsite</method>
        </enterprise_websiterestriction>
      </observers>
    </controller_action_predispatch>
  </frontend>
```

2. restrictWebsite

```php
<?
public function restrictWebsite($observer)
{
    /* @var $controller Mage_Core_Controller_Front_Action */
    $controller = $observer->getEvent()->getControllerAction();

    if (!Mage::app()->getStore()->isAdmin()) {
        $dispatchResult = new Varien_Object(array('should_proceed' => true, 'customer_logged_in' => false));
        Mage::dispatchEvent('websiterestriction_frontend', array(
            'controller' => $controller, 'result' => $dispatchResult
        ));
        if (!$dispatchResult->getShouldProceed()) {
            return;
        }
        if (!Mage::helper('enterprise_websiterestriction')->getIsRestrictionEnabled()) {
            return;
        }
        /* @var $request Mage_Core_Controller_Request_Http */
        $request    = $controller->getRequest();
        /* @var $response Mage_Core_Controller_Response_Http */
        $response   = $controller->getResponse();
        switch ((int)Mage::getStoreConfig(Enterprise_WebsiteRestriction_Helper_Data::XML_PATH_RESTRICTION_MODE)) {
            // show only landing page with 503 or 200 code
            case Enterprise_WebsiteRestriction_Model_Mode::ALLOW_NONE:
                if ($controller->getFullActionName() !== 'restriction_index_stub') {
                    $request->setModuleName('restriction')
                        ->setControllerName('index')
                        ->setActionName('stub')
                        ->setDispatched(false);
                    return;
                }
                $httpStatus = (int)Mage::getStoreConfig(
                    Enterprise_WebsiteRestriction_Helper_Data::XML_PATH_RESTRICTION_HTTP_STATUS
                );
                if (Enterprise_WebsiteRestriction_Model_Mode::HTTP_503 === $httpStatus) {
                    $response->setHeader('HTTP/1.1','503 Service Unavailable');
                }
                break;

            case Enterprise_WebsiteRestriction_Model_Mode::ALLOW_REGISTER:
                // break intentionally omitted

            // redirect to landing page/login
            case Enterprise_WebsiteRestriction_Model_Mode::ALLOW_LOGIN:
                if (!$dispatchResult->getCustomerLoggedIn() && !Mage::helper('customer')->isLoggedIn()) {
                    // see whether redirect is required and where
                    $redirectUrl = false;
                    $allowedActionNames = array_keys(Mage::getConfig()
                        ->getNode(Enterprise_WebsiteRestriction_Helper_Data::XML_NODE_RESTRICTION_ALLOWED_GENERIC)
                        ->asArray()
                    );
                    if (Mage::helper('customer')->isRegistrationAllowed()) {
                        foreach(array_keys(Mage::getConfig()
                            ->getNode(
                                Enterprise_WebsiteRestriction_Helper_Data::XML_NODE_RESTRICTION_ALLOWED_REGISTER
                            )
                            ->asArray()) as $fullActionName
                        ) {
                            $allowedActionNames[] = $fullActionName;
                        }
                    }

                    // to specified landing page
                    $restrictionRedirectCode = (int)Mage::getStoreConfig(
                        Enterprise_WebsiteRestriction_Helper_Data::XML_PATH_RESTRICTION_HTTP_REDIRECT
                    );
                    if (Enterprise_WebsiteRestriction_Model_Mode::HTTP_302_LANDING === $restrictionRedirectCode) {
                        $cmsPageViewAction = 'cms_page_view';
                        $allowedActionNames[] = $cmsPageViewAction;
                        $pageIdentifier = Mage::getStoreConfig(
                            Enterprise_WebsiteRestriction_Helper_Data::XML_PATH_RESTRICTION_LANDING_PAGE
                        );
                        // Restrict access to CMS pages too
                        if (!in_array($controller->getFullActionName(), $allowedActionNames)
                            || ($controller->getFullActionName() === $cmsPageViewAction
                                && $request->getAlias('rewrite_request_path') !== $pageIdentifier)
                        ) {
                            $redirectUrl = Mage::getUrl('', array('_direct' => $pageIdentifier));
                        }
                    } elseif (!in_array($controller->getFullActionName(), $allowedActionNames)) {
                        // to login form
                        $redirectUrl = Mage::getUrl('customer/account/login');
                    }

                    if ($redirectUrl) {
                        $response->setRedirect($redirectUrl);
                        $controller->setFlag('', Mage_Core_Controller_Varien_Action::FLAG_NO_DISPATCH, true);
                    }
                    if (Mage::getStoreConfigFlag(
                        Mage_Customer_Helper_Data::XML_PATH_CUSTOMER_STARTUP_REDIRECT_TO_DASHBOARD
                    )) {
                        $afterLoginUrl = Mage::helper('customer')->getDashboardUrl();
                    } else {
                        $afterLoginUrl = Mage::getUrl();
                    }
                    Mage::getSingleton('core/session')->setWebsiteRestrictionAfterLoginUrl($afterLoginUrl);
                } elseif (Mage::getSingleton('core/session')->hasWebsiteRestrictionAfterLoginUrl()) {
                    $response->setRedirect(
                        Mage::getSingleton('core/session')->getWebsiteRestrictionAfterLoginUrl(true)
                    );
                    $controller->setFlag('', Mage_Core_Controller_Varien_Action::FLAG_NO_DISPATCH, true);
                }
                break;
        }
    }
}
```
3. Настройки доступных роутов для неавторизированых пользователей выполняется через конфиг.

```xml
<global>
  <enterprise>
    <websiterestriction>
      <full_action_names>
        <generic>
          <captcha_refresh_index/>
          <customer_account_login/>
          <customer_account_loginPost/>
          <customer_account_forgotpassword/>
          <customer_account_forgotpasswordpost/>
          <customer_account_confirm/>
          <customer_account_confirmation/>
          <customer_account_resetpassword/>
          <customer_account_resetpasswordpost/>
          <customer_account_changeforgotten/>
          <core_index_noCookies/>
          <paypal_ipn_standard/>
          <paypal_ipn_express/>
          <paypal_ipn_direct/>
          <paypal_ipn_index/>
          <paypaluk_ipn_express/>
          <paypaluk_ipn_direct/>
          <paypal_payflow_silentPost/>
          <paypal_payflowadvanced_silentPost/>
          <amazonpayments_cba_callback/>
          <amazonpayments_cba_notification/>
          <amazonpayments_asp_notification/>
          <ideal_basic_notify/>
          <authorizenet_directpost_payment_response/>
          <enterprise_pbridge_PbridgeIpn_index/>
        </generic>
        <register>
          <customer_account_create/>
          <customer_account_createpost/>
        </register>
      </full_action_names>
    </websiterestriction>
  </enterprise>
</global>
```
