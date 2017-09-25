# Describe group save operations

Для сохранении изменений в группе объектов у коллекции вызвается метод Mage_Core_Model_Resource_Db_Collection_Abstract->save()

```php
<?
public function save()
{
    foreach ($this->getItems() as $item) {
      /* @var $item Mage_Core_Model_Abstract
        $item->save();
    }
    return $this;
}
```
