# Magento API

http://alanstorm.com/series/magento-api/

## Структура api.xml

```xml
<config>
    <api>
        <resources>
            <customer translate="title" module="customer">
            <customer translate="title" module="customer">
              <model>customer/customer_api</model>
              <title>Customer API</title>
              <acl>customer</acl>
              <methods>
                  <list translate="title" module="customer">
                      <title>Retrieve customers</title>
                      <method>items</method>
                      <acl>customer/info</acl>
                  </list>
                  <create translate="title" module="customer">
                      <title>Create customer</title>
                      <acl>customer/create</acl>
                  </create>
                  <info translate="title" module="customer">
                      <title>Retrieve customer data</title>
                      <acl>customer/info</acl>
                  </info>
                  <update translate="title" module="customer">
                      <title>Update customer data</title>
                      <acl>customer/update</acl>
                  </update>
                  <delete translate="title" module="customer">
                      <title>Delete customer</title>
                      <acl>customer/delete</acl>
                  </delete>
              </methods>
              <faults module="customer">
                  <data_invalid>
                      <code>100</code>
                      <message>Invalid customer data. Details in error message.</message>
                  </data_invalid>
                  <filters_invalid>
                      <code>101</code>
                      <message>Invalid filters specified. Details in error message.</message>
                  </filters_invalid>
                  <not_exists>
                      <code>102</code>
                      <message>Customer not exists.</message>
                  </not_exists>
                  <not_deleted>
                      <code>103</code>
                      <message>Customer not deleted. Details in error message.</message>
                  </not_deleted>
              </faults>
          </customer>
            </customer>
        </resources>
    </api>
</config>
```

Нода `resources` содержит конфиг ресурса апи, в дополнение к `resource` конфиг может содержать `resource_alias` внутри которого содержатся элисасы для ресурсов, например

```xml
<config>
    <api>
        <resources_alias>
            <category>catalog_category</category>
        </resource_alias>
    </api>
</config>    
```

Позволяет вызывать ресурс так `category.info` и так `catalog_category.info`

  - model — содержит элиас к классу модели, которая будет использована для ресурса
  - methods — содержит методы модели, каждый метод апи соответствует методы класса модели, название метода указывается в `method` или, если не указано, то берётся название ноды из methods


Код вызова метода ресрса АПИ

```php
<?
public function call($sessionId, $apiPath, $args = array())
    {
        $this->_startSession($sessionId);

        if (!$this->_getSession()->isLoggedIn($sessionId)) {
            return $this->_fault('session_expired');
        }

        list($resourceName, $methodName) = explode('.', $apiPath);

        if (empty($resourceName) || empty($methodName)) {
            return $this->_fault('resource_path_invalid');
        }

        $resourcesAlias = $this->_getConfig()->getResourcesAlias();
        $resources      = $this->_getConfig()->getResources();
        if (isset($resourcesAlias->$resourceName)) {
            $resourceName = (string) $resourcesAlias->$resourceName;
        }

        if (!isset($resources->$resourceName)
            || !isset($resources->$resourceName->methods->$methodName)) {
            return $this->_fault('resource_path_invalid');
        }

        if (!isset($resources->$resourceName->public)
            && isset($resources->$resourceName->acl)
            && !$this->_isAllowed((string)$resources->$resourceName->acl)) {
            return $this->_fault('access_denied');

        }


        if (!isset($resources->$resourceName->methods->$methodName->public)
            && isset($resources->$resourceName->methods->$methodName->acl)
            && !$this->_isAllowed((string)$resources->$resourceName->methods->$methodName->acl)) {
            return $this->_fault('access_denied');
        }

        $methodInfo = $resources->$resourceName->methods->$methodName;

        try {
            $method = (isset($methodInfo->method) ? (string) $methodInfo->method : $methodName);

            $modelName = $this->_prepareResourceModelName((string) $resources->$resourceName->model);
            try {
                $model = Mage::getModel($modelName);
                if ($model instanceof Mage_Api_Model_Resource_Abstract) {
                    $model->setResourceConfig($resources->$resourceName);
                }
            } catch (Exception $e) {
                throw new Mage_Api_Exception('resource_path_not_callable');
            }

            if (method_exists($model, $method)) {
                $result = array();
                if (isset($methodInfo->arguments) && ((string)$methodInfo->arguments) == 'array') {
                    $result = $model->$method((is_array($args) ? $args : array($args)));
                } elseif (!is_array($args)) {
                    $result = $model->$method($args);
                } else {
                    $result = call_user_func_array(array(&$model, $method), $args);
                }
                return $this->processingMethodResult($result);
            } else {
                throw new Mage_Api_Exception('resource_path_not_callable');
            }
        } catch (Mage_Api_Exception $e) {
            return $this->_fault($e->getMessage(), $resourceName, $e->getCustomMessage());
        } catch (Exception $e) {
            Mage::logException($e);
            return $this->_fault('internal', null, $e->getMessage());
        }
    }
```  

## Архитектура API

Базовая архитектура маджентовского апи состоит в следующем: HTTP запрос направляется в мадженту, с помощью маджентовского роутинга попадает к модулю `Mage_Api`, в контроллере этого модуля инстанцируется Magento API сервер для соответствующего типа апи, затем у сервиса вызывается метод Run Класс сервера — `Mage_Api_Model_Server`

Иницилизация объекта сервера

```php
<?
public function initialize($adapterCode, $handler = null)
{
    /** @var $helper Mage_Api_Model_Config */
    $helper   = Mage::getSingleton('api/config');
    $adapters = $helper->getActiveAdapters();

    if (isset($adapters[$adapterCode])) {
        /** @var $adapterModel Mage_Api_Model_Server_Adapter_Interface */
        $adapterModel = Mage::getModel((string) $adapters[$adapterCode]->model);

        if (!($adapterModel instanceof Mage_Api_Model_Server_Adapter_Interface)) {
            Mage::throwException(Mage::helper('api')->__('Invalid webservice adapter specified.'));
        }
        $this->_adapter = $adapterModel;
        $this->_api     = $adapterCode;

        // get handler code from config if no handler passed as argument
        if (null === $handler && !empty($adapters[$adapterCode]->handler)) {
            $handler = (string) $adapters[$adapterCode]->handler;
        }
        $handlers = $helper->getHandlers();

        if (!isset($handlers->$handler)) {
            Mage::throwException(Mage::helper('api')->__('Invalid webservice handler specified.'));
        }
        $handlerClassName = Mage::getConfig()->getModelClassName((string) $handlers->$handler->model);

        $this->_adapter->setHandler($handlerClassName);
    } else {
        Mage::throwException(Mage::helper('api')->__('Invalid webservice adapter specified.'));
    }
    return $this;
}
```

Контролеры апи наследуются от 'Mage_Api_Controller_Action', в нём содержится переопределенный `preDispatch`

```php
<?
public function preDispatch()
{
    $this->getLayout()->setArea('adminhtml');
    Mage::app()->setCurrentStore('admin');
    $this->setFlag('', self::FLAG_NO_START_SESSION, 1); // Do not start standart session
    parent::preDispatch();
    return $this;
}
```
