# Use the Magento API to implement third party integrations

Маджента поддерживает SOAP, SOAP v2, SOAP WSI, XML-RPC и RestFull API.

Основные настройки API находятся в api.xml и api2.XML-RPC

api.xml
```xml
<adapters>
  <api>
        <resources>
            <customer translate="title" module="customer">
                <model>customer/customer_api</model>
            </customer>
        </resources>
    </api>
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
```

## Адаптеры

Адаптеры наследуются от ` Mage_Api_Model_Server_Adapter_Interface`.

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

В самой мадженте реализованы такие адаптеры

  - Mage_Api_Model_Server_Adapter_Xmlrpc
  - Mage_Api_Model_Server_Adapter_Soap
  - Mage_Api_Model_Server_V2_Adapter_Soap
  - Mage_Api_Model_Server_WSI_Adapter_Soap
