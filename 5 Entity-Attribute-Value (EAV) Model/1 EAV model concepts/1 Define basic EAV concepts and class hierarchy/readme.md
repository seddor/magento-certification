# Define basic EAV concepts and class hierarchy

EAV (Entity-Attribute-Value) — состоит из следующих частей:

  * Entity — (customer, catalog)
  * Attribute — (text, decimal, integer etc.)
  * Value — (Value)

## Типы

В мадженте имеется 8 EAV типов, их можно найти в таблице `eav_entity_type`

  * Catalog Product
  * Catalog Category
  * Customer
  * Customer Address
  * Order
  * Invoice
  * Credit Memo
  * Shipment

## Иерархия

Иерархия классов схожа с обычными core моделями, так же нужно 3 класса, для модели, ресурса и коллекции.

  * Модель наследуется от Mage_Core_Model_Abstract
  * Ресурс от Mage_Eav_Model_Entity_Abstract
  * Колекция от Mage_Eav_Model_Entity_Collection_Abstract


[подробнее про создание кастомного EAV типа](https://github.com/colinmurphy/magento-exam-notes/blob/master/5.%20EAV/1.%20EAV%20Concepts/1.%20Define%20basic%20EAV%20concepts%20and%20class%20hierarchy.md#3-create-a-eav-type)

http://www.divisionlab.com/solvingmagento/magento-eav-system/
