# Non-RESTful APIs

## Magento API Adapters and Handlers

Адраптеры ответчат за вызовы обработчиков запросов, вывод ошибок, адаптеры имплементируют `Mage_Api_Model_Server_Adapter_Interface`.

```php
<?
interface Mage_Api_Model_Server_Adapter_Interface
{
    /**
     * Set handler class name for webservice
     *
     * @param string $handler
     * @return Mage_Api_Model_Server_Adapter_Interface
     */
    function setHandler($handler);

    /**
     * Retrive handler class name for webservice
     *
     * @return string
     */
    function getHandler();

    /**
     * Set webservice api controller
     *
     * @param Mage_Api_Controller_Action $controller
     * @return Mage_Api_Model_Server_Adapter_Interface
     */
    function setController(Mage_Api_Controller_Action $controller);

    /**
     * Retrive webservice api controller
     *
     * @return Mage_Api_Controller_Action
     */
    function getController();

    /**
     * Run webservice
     *
     * @return Mage_Api_Model_Server_Adapter_Interface
     */
    function run();

    /**
     * Dispatch webservice fault
     *
     * @param int $code
     * @param string $message
     */
    function fault($code, $message);

}
```

Адаптеры доступные в мадженте:

```xml
<adapters>
  <soap>
    <model>api/server_adapter_soap</model>
    <handler>default</handler>
    <active>1</active>
    <required>
      <extensions>
        <soap />
      </extensions>
    </required>
  </soap>
  <soap_v2>
    <model>api/server_v2_adapter_soap</model>
    <handler>soap_v2</handler>
    <active>1</active>
    <required>
      <extensions>
        <soap />
      </extensions>
    </required>
  </soap_v2>
  <soap_wsi>
    <model>api/server_wsi_adapter_soap</model>
    <handler>soap_wsi</handler>
    <active>1</active>
    <required>
      <extensions>
        <soap />
      </extensions>
    </required>
  </soap_wsi>
  <xmlrpc>
    <model>api/server_adapter_xmlrpc</model>
    <handler>default</handler>
    <active>1</active>
  </xmlrpc>
  <default>
    <use>soap</use>
  </default>
</adapters>
```

Задача хендлера обработать запросы от адаптера, связть запросы с доступными ресурсами api, базовый класс хендела — `Mage_Api_Model_Server_Handler_Abstract`.

Хендлеры мадженты:

```xml
<handlers>
  <default>
    <model>api/server_handler</model>
  </default>
  <soap_v2>
    <model>api/server_v2_handler</model>
  </soap_v2>
  <soap_wsi>
    <model>api/server_wsi_handler</model>
  </soap_wsi>
</handlers>
```

## WSDL

Маджета позволяет использовать WSDL для soap если в запросе передан соответвующий параметр, wsdl используется для описаний стуктуры запросов и ответов в апи, конфиги для хранятся в `wsdl.xmk` файлах
