# Describe how to implement the interface of attribute frontend, source, and backend models

## How do attribute models, attribute source models, attribute backend models and attribute frontend models relate to each other?

  * attribute model — базовая модель аттрибута, обычно представлена классом `Mage_Eav_Model_Entity_Attribute` наследует `Mage_Eav_Model_Entity_Attribute_Abstract` и содержит методы для получения остальных моделей
  * attribute source model — модель данных для селектов, эти модели наследуют `Mage_Eav_Model_Entity_Attribute_Source_Abstract`
  * attribute backend model — бекенд модели предназначены для сохранения данных аттрибутов, бекедные модели имплементируют интерфейс `Mage_Eav_Model_Entity_Attribute_Backend_Interface`.
  * attribute frontend model — вывод значений аттрибутов на фронтенд, модели наследуются от `Mage_Eav_Model_Entity_Attribute_Frontend_Abstract`

## Which methods have to be implemented in a custom source model (or frontend model or backend model)?

  * toOptionArray для админки
  * getAllOptions для фронтенда

## Can adminhtml system configuration source models also be used for EAV attributes?

Да.

## What is the default frontend model (and source and backend model) for EAV attributes?

`Mage_Eav_Model_Entity_Attribute_Frontend_Default`

## Does every attribute use a source model?

Нет.

## Which classes and methods are related to updating the EAV attribute values in the flat catalog tables?

```
Mage::getResourceModel('eav/entity_setup)->updateAttribute()
Mage::getResourceModel('eav/entity_setup)->addAttribute()
```

## What factors allow for attributes to be added to flat catalog tables?

За это отвечают source models. Они содержат методы:

  * getFlatIndexes()
  * getFlatColumns()
  * getFlatUpdateSelect()

Также статичные аттрибуты добавляются во флэт таблицы

## How are store-scoped entity attribute values stored when catalog flat storage is enabled for that entity type?

`catalog_category_flat_store_{{id}}`

## Which frontend, source, and backend models are available in a stock Magento installation?

## How do multi-lingual options for attributes work in Magento?

раздаляются по store_id

## How do you get a list of all options for an attribute for a specified store view in addition to the admin scope?

Mage::getModel('eav/entity_attribute')->load({id})->getSource()->getAllOptions()
