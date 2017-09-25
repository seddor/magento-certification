# Describe the EAV load-and-save process and its differences from the regular load-and-save process

## load

### Загрузка обычных моделей

```
$flat = Mage::getModel('foo_bar/baz')->load(1);
```

Модель грузится посредством **Mage_Core_Model_Resource_Db_Abstract->load()**

  1. Получются основные поля
  2. Создаётся запрос (Varien_Db_Select):
  ```
  SELECT `foo_bar_baz_table`.* FROM `foo_bar_baz_table` WHERE (`foo_bar_baz_table`.`id` = 1)
  ```

  3. $read->fetchRow($select);

### Загрузка EAV

```
$eav = Mage::getModel('foo_eav/bar')->load(1);
```

Модель грузится посредством **Mage_Eav_Model_Entity_Abstract->load($object, $entityId, $attributes = array())**

  1. Получются основные полей из `bar_entity`
    $select = $this->\_getLoadRowSelect($object, $entityId); $row = $this->\_getReadAdapter()->fetchRow($select);
  2. проверка если парамтер с аттрибутами пустов, то должны загрузится все аттрибуты.
  3. Загрузка аттрибутов в $object: $this->\_loadModelAttributes($object)

## Save

  * Flat tables save to 1 row in Mage_Core_Model_Resource_Db_Abstract->save($object)
  * EAV tables save to multiple tables with Mage_Eav_Model_Entity_Abstract->save($object)
