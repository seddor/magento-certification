# Describe the programmatic structure of payment methods and how to implement new methods

## Создание кастомного платежного метода

1. Конфиг в config.xml

```xml
<default>
  <payment>
    <mycheckout>
      <model>foo_checkout/standard</model>
      <active>1</active>
      <title>Foo</title>
      <sort_order>100</sort_order>
      <sallowspecific>0</sallowspecific>
    </mycheckout>
  </payment>
</default>
```

2. Системный конфиг в `system.xml`

```xml
<config>
    <sections>
        <payment>
            <groups>
                <foo_card translate="label">
                    <label>Foo Card</label>
                    <sort_order>1</sort_order>
                    <show_in_default>1</show_in_default>
                    <show_in_website>0</show_in_website>
                    <show_in_store>0</show_in_store>
                    <fields>
                        <active translate="label">
                            <label>Enabled</label>
                            <frontend_type>select</frontend_type>
                            <source_model>adminhtml/system_config_source_yesno</source_model>
                            <sort_order>1</sort_order>
                            <show_in_default>1</show_in_default>
                            <show_in_website>0</show_in_website>
                            <show_in_store>0</show_in_store>
                        </active>
                        <title translate="label">
                            <label>Title</label>
                            <frontend_type>text</frontend_type>
                            <sort_order>20</sort_order>
                            <show_in_default>1</show_in_default>
                            <show_in_website>0</show_in_website>
                            <show_in_store>0</show_in_store>
                        </title>
                        <sallowspecific translate="label">
                            <label>For selected countries only</label>
                            <frontend_type>select</frontend_type>
                            <frontend_class>shipping-applicable-country</frontend_class>
                            <source_model>adminhtml/system_config_source_shipping_allspecificcountries</source_model>
                            <sort_order>30</sort_order>
                            <show_in_default>1</show_in_default>
                            <show_in_website>0</show_in_website>
                            <show_in_store>0</show_in_store>
                        </sallowspecific>
                        <specificcountry translate="label">
                            <label>Ship to Specific Countries</label>
                            <frontend_type>multiselect</frontend_type>
                            <sort_order>31</sort_order>
                            <source_model>adminhtml/system_config_source_country</source_model>
                            <show_in_default>1</show_in_default>
                            <show_in_website>0</show_in_website>
                            <show_in_store>0</show_in_store>
                            <can_be_empty>1</can_be_empty>
                        </specificcountry>
                    </fields>
                </foo_card>
            </groups>
        </payment>
    </sections>
</config>
```

3. Модель, класс модели должен наследовать `Mage_Payment_Model_Method_Abstract`.

опции в метода определяются посредством переопределения значений полей
```
/**
 * Availability options
 */
 protected $_isGateway                   = false;
 protected $_canOrder                    = false;
 protected $_canAuthorize                = false;
 protected $_canCapture                  = false;
 protected $_canCapturePartial           = false;
 protected $_canCaptureOnce              = false;
 protected $_canRefund                   = false;
 protected $_canRefundInvoicePartial     = false;
 protected $_canVoid                     = false;
 protected $_canUseInternal              = true;
 protected $_canUseCheckout              = true;
 protected $_canUseForMultishipping      = true;
 protected $_isInitializeNeeded          = false;
 protected $_canFetchTransactionInfo     = false;
 protected $_canReviewPayment            = false;
 protected $_canCreateBillingAgreement   = false;
 protected $_canManageRecurringProfiles  = true;
```

помиомо этого `Mage_Payment_Model_Method_Abstract` продоставляет методы, которые используются в процесе оплаты маджента, и которые можно переопределить в кастомном метода. Такие как `cancel`, `refund`, `authorize`, `capture` и т.д.

## How can payment method behavior be customized (for example: whether to charge or authorize a credit card; controlling URL redirects; etc.)?

Переопределить в кастомной моделе соответствующий метод `Mage_Payment_Model_Method_Abstract`

## Which class is the basic class in the payment method hierarchy?

`Mage_Payment_Model_Method_Abstract`

## How can the stored data of payment methods be customized (credit card numbers, for example)?

1. В моделе метода оплаты можно переопределить метода `prepareSave()`
2. в метода `Mage_Sales_Model_Quote_Payment->importData(array $data)` созадется событие `sales_quote_payment_import_data_before`

```
Mage::dispatchEvent(
    $this->_eventPrefix . '_import_data_before',
    array(
        $this->_eventObject=>$this,
        'input'=>$data,
    )
);
```
3. В кастомном метода переопределить `assignData($data)`

## What is the difference between payment method and payment classes (such as order_payment, quote_payment, etc.)?

Методы оплаты отвечают за оплату и прочие прямые операции с оператором оплаты и управляются платжными классами order_payment, quote_payment, т.е. в этих классах получается конкретный инстанс метода оплаты, который вызвается внути класса при оплате и прочих действиях

## What is the typical structure of the payment method module?

1. конфиги config.xml и system.xml
2. Модель метода оплаты наследуемая от `Mage_Payment_Model_Method_Abstract`
3. При необходимости блоки для формы и информации о методе, их типы задаюстся в моделе метода оплаты
```
protected $_formBlockType = 'payment/form_cc';
protected $_infoBlockType = 'payment/info_cc';
```
4. При необходимости шаблоны блоков формы и информации об методе

## How do payment modules support billing agreements?

Из коробки маджента поддерживает измение блинговых соглашений если вызвать `$order->getPayment()->setBillingAgreementData($data)`. и билинговая информация используется по вызову метода '$order->getPayment()->place()'

```php
<?
/**
 * Generate billing agreement object if there is billing agreement data
 * Adds it to order as related object
 */
protected function _createBillingAgreement()
{
    if ($this->getBillingAgreementData()) {
        $order = $this->getOrder();
        $agreement = Mage::getModel('sales/billing_agreement')->importOrderPayment($this);
        if ($agreement->isValid()) {
            $message = Mage::helper('sales')->__('Created billing agreement #%s.', $agreement->getReferenceId());
            $order->addRelatedObject($agreement);
            $this->_billingAgreement = $agreement;
        } else {
            $message = Mage::helper('sales')->__('Failed to create billing agreement for this order.');
        }
        $comment = $order->addStatusHistoryComment($message);
        $order->addRelatedObject($comment);
    }
}
```

Если нужно добавить усправление billing agreements в платежный метод, то можно имплементировать `Mage_Payment_Model_Billing_Agreement_MethodInterface`

```php
<?interface Mage_Payment_Model_Billing_Agreement_MethodInterface
{
    /**
     * Init billing agreement
     *
     * @param Mage_Payment_Model_Billing_AgreementAbstract $agreement
     */
    public function initBillingAgreementToken(Mage_Payment_Model_Billing_AgreementAbstract $agreement);

    /**
     * Retrieve billing agreement details
     *
     * @param Mage_Payment_Model_Billing_AgreementAbstract $agreement
     */
    public function getBillingAgreementTokenInfo(Mage_Payment_Model_Billing_AgreementAbstract $agreement);

    /**
     * Create billing agreement
     *
     * @param Mage_Payment_Model_Billing_AgreementAbstract $agreement
     */
    public function placeBillingAgreement(Mage_Payment_Model_Billing_AgreementAbstract $agreement);

    /**
     * Update billing agreement status
     *
     * @param Mage_Payment_Model_Billing_AgreementAbstract $agreement
     */
    public function updateBillingAgreementStatus(Mage_Payment_Model_Billing_AgreementAbstract $agreement);
}
```
