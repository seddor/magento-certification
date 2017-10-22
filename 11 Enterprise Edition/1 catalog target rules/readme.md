# Describe how to customize, extend, and troubleshoot Enterprise Edition catalog target rules

## What additional possibilities does the Enterprise target rule module provide over the Mage_CatalogRule?

Target rule позволяет находить автоматически привязывать апселы, релейтеды или крос селы к продуктам, в правиле указывается с какими продуктами должно совпадать правило и какие продукты должны отображаться для этих продуктов



## How does the module store the rules in the database?

Инофрмация поделена среди следующих таблиц

  - enterprise_targetrule — Основная таблица содержащие правила
  - enterprise_targetrule_customersegment — соответские правил кастомер сегментам
  - enterprise_targetrule_product — хранит соответствия продуктов и правил
  - enterprise_targetrule_index
  - enterprise_targetrule_index_related
  - enterprise_targetrule_index_crosssell
  - enterprise_targetrule_index_upsell
  - enterprise_targetrule_index_related_product
  - enterprise_targetrule_index_upsell_product
  - enterprise_targetrule_index_crosssell_product

## Which indexer processes the rules, and how is the index used to realize faster reads?

 - Класс модели индекса — Enterprise_TargetRule_Model_Index
 - Ресрусная модель — Enterprise_TargetRule_Model_Resource_Index

 Продукты получаемые в правилах выводятся через соответствующие блоки. По сути выводится коллекция продуктов отфильтрованная по ИД

 ```php
 <?
 /**
 * Get target rule products by ids
 *
 * @param array $ids
 * @param int $limit
 * @return array
 */
protected function _getTargetRuleProductsByIds(array $ids, $limit)
{
    /** @var $collection Mage_Catalog_Model_Resource_Eav_Mysql4_Product_Collection */
    $collection = Mage::getResourceModel('catalog/product_collection');
    $collection->addFieldToFilter('entity_id', array('in' => $ids));
    $collection->addAttributeToFilter('status', Mage_Catalog_Model_Product_Status::STATUS_ENABLED);
    $this->_addProductAttributesAndPrices($collection);

    $collection->setFlag('is_link_collection', true);
    Mage::getSingleton('catalog/product_visibility')->addVisibleInCatalogFilterToCollection($collection);
    $collection->setPageSize($limit)->setFlag('do_not_use_category_id', true);
    $items = array();
    foreach ($collection as $item) {
        $items[$item->getEntityId()] = $item;
    }
    return $items;
}

/**
 * Get target rule collection ids
 *
 * @param null|int $limit
 * @return array
 */
protected function _getTargetRuleProductIds($limit = null)
{
    $excludeProductIds = $this->getExcludeProductIds();
    if (!is_null($this->_items)) {
        $excludeProductIds = array_merge(array_keys($this->_items), $excludeProductIds);
    }
    $indexModel = $this->_getTargetRuleIndex()
        ->setType($this->getProductListType())
        ->setProduct($this->getProduct())
        ->setExcludeProductIds($excludeProductIds);
    if (!is_null($limit)) {
        $indexModel->setLimit($limit);
    }

    return $indexModel->getProductIds();
}
```

getProductIds в `Enterprise_TargetRule_Model_Index` вызывает метод `getProductIds($object)` у  `Enterprise_TargetRule_Model_Resource_Index`

```php
<?
public function getProductIds($object)
{
    $adapter = $this->_getReadAdapter();
    $select  = $adapter->select()
        ->from($this->getMainTable(), 'customer_segment_id')
        ->where('type_id = :type_id')
        ->where('entity_id = :entity_id')
        ->where('store_id = :store_id')
        ->where('customer_group_id = :customer_group_id');

    $rotationMode = $this->_factory->getHelper('enterprise_targetrule')->getRotationMode($object->getType());

    $segmentsIds = array_merge(array(0), $this->_getSegmentsIdsFromCurrentCustomer());
    $bind = array(
        ':type_id'              => $object->getType(),
        ':entity_id'            => $object->getProduct()->getEntityId(),
        ':store_id'             => $object->getStoreId(),
        ':customer_group_id'    => $object->getCustomerGroupId()
    );

    $segmentsList = $adapter->fetchAll($select, $bind);

    $foundSegmentIndexes = array();
    foreach ($segmentsList as $segment) {
        $foundSegmentIndexes[] = $segment['customer_segment_id'];
    }

    $productIds = array();
    foreach ($segmentsIds as $segmentId) {
        if (in_array($segmentId, $foundSegmentIndexes)) {
            $productIds = array_merge($productIds,
                $this->getTypeIndex($object->getType())->loadProductIdsBySegmentId($object, $segmentId));
        } else {
            $matchedProductIds = $this->_matchProductIdsBySegmentId($object, $segmentId);
            $productIds = array_merge($matchedProductIds, $productIds);
            $this->getTypeIndex($object->getType())
                ->saveResultForCustomerSegments($object, $segmentId, $matchedProductIds);
            $this->saveFlag($object, $segmentId);
        }
    }
    $productIds = array_diff(array_unique($productIds), $object->getExcludeProductIds());

    if ($rotationMode == Enterprise_TargetRule_Model_Rule::ROTATION_SHUFFLE) {
         shuffle($productIds);
    }
    return $productIds;
}

public function getTypeIndex($type)
{
    switch ($type) {
        case Enterprise_TargetRule_Model_Rule::RELATED_PRODUCTS:
            $model = 'related';
            break;

        case Enterprise_TargetRule_Model_Rule::UP_SELLS:
            $model = 'upsell';
            break;

        case Enterprise_TargetRule_Model_Rule::CROSS_SELLS:
            $model = 'crosssell';
            break;

        default:
            Mage::throwException(
                Mage::helper('enterprise_targetrule')->__('Undefined Catalog Product List Type')
            );
    }

    return Mage::getResourceSingleton('enterprise_targetrule/index_' . $model);
}
```

Все индексные модели для конкретного типа наследуются от `Enterprise_TargetRule_Model_Resource_Index_Abstract` в нём вызывется метод `loadProductIdsBySegmentId` который загружает сохранёные в индексе продукты для конкретного сегмента кастомеров
