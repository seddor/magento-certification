# Modify, extend, and troubleshoot the Magento layered (“filter”) navigation

Модели фильтров должны наследоваться от `Mage_Catalog_Model_Layer_Filter_Abstract`. В layered navigation отображаются фильтры по аттрибутам в соотвествии с настройкой в адмнинке(там можно указать какие атрибуты использовать в layered navigation).

## выжные методы Mage_Catalog_Model_Layer_Filter_Abstract

```php
<?
/**
 * Здесь происходит применение фильтров
 * запрашиваемое значение фильтра можно получить из объекта реквеста
 * $filter = $request->getParam($this->_requestVar)
 *
 * @param  Zend_Controller_Request_Abstract $request
 */
public function apply(Zend_Controller_Request_Abstract $request, $filterBlock)
{
    return $this;
}

/**
 * Возвращает данные для айтемов фильтра
 *
 * result array should have next structure:
 * array(
 *      $index => array(
 *          'label' => $label,
 *          'value' => $value,
 *          'count' => $count
 *      )
 * )
 *
 * @return array
 */
protected function _getItemsData()
{
    return array();
}

/**
 * Initialize filter items
 *
 * @return  Mage_Catalog_Model_Layer_Filter_Abstract
 */
protected function _initItems()
{
    $data = $this->_getItemsData();
    $items=array();
    foreach ($data as $itemData) {
        $items[] = $this->_createItem(
            $itemData['label'],
            $itemData['value'],
            $itemData['count']
        );
    }
    $this->_items = $items;
    return $this;
}

/**
 * Create filter item object
 *
 * @param   string $label
 * @param   mixed $value
 * @param   int $count
 * @return  Mage_Catalog_Model_Layer_Filter_Item
 */
protected function _createItem($label, $value, $count=0)
{
    return Mage::getModel('catalog/layer_filter_item')
        ->setFilter($this)
        ->setLabel($label)
        ->setValue($value)
        ->setCount($count);
}
```
