# Describe the EAV entity structure and its difference from the standard core resource model

## Разные ресурсные модели

Flat Resource:

This sets the main table and field.

```php
<?
  class Foo_Bar_Model_Resource_Baz extends Mage_Core_Model_Resource_Db_Abstract
  {
      protected function _construct()
      {
          $this->_init('foo_bar/baz', 'id');
      }
  }
```
EAV Resource:

This sets the EAV type and connections.

```php
<?
class Foo_EAV_Model_Resource_Bar extends Mage_Eav_Model_Entity_Abstract
{
    protected function _construct()
    {
        $resource = Mage::getSingleton('core/resource');
        $this->setType('foo_eav_bar');
        $this->setConnection(
            $resource->getConnection('core_read'),
            $resource->getConnection('core_write')
        );
    }
}
```
## Разные коллекции

Flat Resource:

```php
<?
class Foo_Bar_Model_Resource_Baz_Collection extends Mage_Core_Model_Resource_Db_Collection_Abstract
{
    public function _construct()
    {
        $this->_init('foo_bar/baz');
    }
}
```

EAV Resource:

```php
class Foo_Eav_Model_Resource_Bar_Collection extends Mage_Eav_Model_Entity_Collection_Abstract
{
    protected function _construct()
    {
        $this->_init('foo_eav/bar');
    }
}
```

## Разные setup модели

  * Flat Resource: Extends from Mage_Core_Model_Resource_Setup
  * EAV Resource: Extends from Mage_Eav_Model_Entity_Setup

## Разные методы для фильтрации коллекций

  * Flat Resource: ->addFieldToFilter();
  * EAV Resource: ->addAttributeToFilter();
