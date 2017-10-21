# REST

https://habrahabr.ru/post/232603/

Для работы с RestFull api используется модуль `Mage_Api2`, конфиги содержатся в api2.xml файлах

```xml
<config>
    <api2>
        <resource_groups>
            <easy_interfacing translate="title" module="api2">
                <title>Easy Interfacing REST</title>
                <sort_order>30</sort_order>
                <children>
                    <easy_interfacing_orders translate="title" module="api2">
                        <title>Orders</title>
                        <sort_order>50</sort_order>
                    </easy_interfacing_orders>
                </children>
            </easy_interfacing>
        </resource_groups>
        <resources>
            <easy_interfacing_orders translate="title" module="api2">
                <group>easy_interfacing</group>
                <model>easy_interfacing/api2_order</model>
                <filter>easy_interfacing/api2_order_filter</filter>
                <title>Orders</title>
                <sort_order>10</sort_order>
                <versions>1</versions>
                <routes>
                    <route_collection>
                        <route>/easy_interfacing/order</route>
                        <action_type>collection</action_type>
                    </route_collection>
                </routes>
                <privileges>
                    <guest>
                        <update>1</update>
                    </guest>
                </privileges>
                <attributes translate="id shipment" module="easy_interfacing">
                    <id>Order ID</id>
                    <shipment>Shipment data</shipment>
                </attributes>
            </easy_interfacing_orders>
        </resources>
    </api2>
</config>
```

  - resource_groups — ответчает за ACL
  - model — элиса к модели класса ресурса, ресурсы наследуются от `Mage_Api2_Model_Resource`
  - routes — пути, action_type —  может быть entity или collection, для одиночных и множественных соответственно, например у ресрса продукта
```xml
<route_entity>
  <route>/products/:id</route>
  <action_type>entity</action_type>
</route_entity>
<route_collection>
  <route>/products</route>
  <action_type>collection</action_type>
</route_collection>
```
  - attributes — ответчает за фильтрацию аттрибутов, фильтрацию производит класс `Mage_Api2_Model_Acl_Filter`


Определение какой метод ресурса будет вызван происходит в методе `dispath`

```php
<?
public function dispatch()
    {
        switch ($this->getActionType() . $this->getOperation()) {
            /* Create */
            case self::ACTION_TYPE_ENTITY . self::OPERATION_CREATE:
                // Creation of objects is possible only when working with collection
                $this->_critical(self::RESOURCE_METHOD_NOT_IMPLEMENTED);
                break;
            case self::ACTION_TYPE_COLLECTION . self::OPERATION_CREATE:
                // If no of the methods(multi or single) is implemented, request body is not checked
                if (!$this->_checkMethodExist('_create') && !$this->_checkMethodExist('_multiCreate')) {
                    $this->_critical(self::RESOURCE_METHOD_NOT_IMPLEMENTED);
                }
                // If one of the methods(multi or single) is implemented, request body must not be empty
                $requestData = $this->getRequest()->getBodyParams();
                if (empty($requestData)) {
                    $this->_critical(self::RESOURCE_REQUEST_DATA_INVALID);
                }
                // The create action has the dynamic type which depends on data in the request body
                if ($this->getRequest()->isAssocArrayInRequestBody()) {
                    $this->_errorIfMethodNotExist('_create');
                    $filteredData = $this->getFilter()->in($requestData);
                    if (empty($filteredData)) {
                        $this->_critical(self::RESOURCE_REQUEST_DATA_INVALID);
                    }
                    $newItemLocation = $this->_create($filteredData);
                    $this->getResponse()->setHeader('Location', $newItemLocation);
                } else {
                    $this->_errorIfMethodNotExist('_multiCreate');
                    $filteredData = $this->getFilter()->collectionIn($requestData);
                    $this->_multiCreate($filteredData);
                    $this->_render($this->getResponse()->getMessages());
                    $this->getResponse()->setHttpResponseCode(Mage_Api2_Model_Server::HTTP_MULTI_STATUS);
                }
                break;
            /* Retrieve */
            case self::ACTION_TYPE_ENTITY . self::OPERATION_RETRIEVE:
                $this->_errorIfMethodNotExist('_retrieve');
                $retrievedData = $this->_retrieve();
                $filteredData  = $this->getFilter()->out($retrievedData);
                $this->_render($filteredData);
                break;
            case self::ACTION_TYPE_COLLECTION . self::OPERATION_RETRIEVE:
                $this->_errorIfMethodNotExist('_retrieveCollection');
                $retrievedData = $this->_retrieveCollection();
                $filteredData  = $this->getFilter()->collectionOut($retrievedData);
                $this->_render($filteredData);
                break;
            /* Update */
            case self::ACTION_TYPE_ENTITY . self::OPERATION_UPDATE:
                $this->_errorIfMethodNotExist('_update');
                $requestData = $this->getRequest()->getBodyParams();
                if (empty($requestData)) {
                    $this->_critical(self::RESOURCE_REQUEST_DATA_INVALID);
                }
                $filteredData = $this->getFilter()->in($requestData);
                if (empty($filteredData)) {
                    $this->_critical(self::RESOURCE_REQUEST_DATA_INVALID);
                }
                $this->_update($filteredData);
                break;
            case self::ACTION_TYPE_COLLECTION . self::OPERATION_UPDATE:
                $this->_errorIfMethodNotExist('_multiUpdate');
                $requestData = $this->getRequest()->getBodyParams();
                if (empty($requestData)) {
                    $this->_critical(self::RESOURCE_REQUEST_DATA_INVALID);
                }
                $filteredData = $this->getFilter()->collectionIn($requestData);
                $this->_multiUpdate($filteredData);
                $this->_render($this->getResponse()->getMessages());
                $this->getResponse()->setHttpResponseCode(Mage_Api2_Model_Server::HTTP_MULTI_STATUS);
                break;
            /* Delete */
            case self::ACTION_TYPE_ENTITY . self::OPERATION_DELETE:
                $this->_errorIfMethodNotExist('_delete');
                $this->_delete();
                break;
            case self::ACTION_TYPE_COLLECTION . self::OPERATION_DELETE:
                $this->_errorIfMethodNotExist('_multiDelete');
                $requestData = $this->getRequest()->getBodyParams();
                if (empty($requestData)) {
                    $this->_critical(self::RESOURCE_REQUEST_DATA_INVALID);
                }
                $this->_multiDelete($requestData);
                $this->getResponse()->setHttpResponseCode(Mage_Api2_Model_Server::HTTP_MULTI_STATUS);
                break;
            default:
                $this->_critical(self::RESOURCE_METHOD_NOT_IMPLEMENTED);
                break;
        }
    }
```
