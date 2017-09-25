# Describe the load-and-save process for a regular entity

## Load моделей

Загрузка моделей происходит с помощью метода Mage_Core_Model_Abstract->load($id, $field=null) вызваного у специфичной модели наследуемой от Mage_Core_Model_Abstract

```php
<?
public function load($id, $field=null)
{
    $this->_beforeLoad($id, $field);
    $this->_getResource()->load($this, $id, $field);
    $this->_afterLoad();
    $this->setOrigData();
    $this->_hasDataChanges = false;
    return $this;
}
```

## Получение коллекций

Получить коллекцию можно следующими способами:

  * $collection = Mage::getModel('foo_bar/baz')->getCollection();
  * $collection = Mage::getResourceModel('foo_bar/baz_collection');

Результат будет одинаковым. Коллекция не будет грузится из БД пока явно или не явно не будет вызван метод load.


## Сохранение

Сохранение происохдит после вызова метода Mage_Core_Model_Abstract->save()

```php
<?
public function save()
{
    /**
     * Direct deleted items to delete method
     */
    if ($this->isDeleted()) {
        return $this->delete();
    }
    if (!$this->_hasModelChanged()) {
        return $this;
    }
    $this->_getResource()->beginTransaction();
    $dataCommited = false;
    try {
        $this->_beforeSave();
        if ($this->_dataSaveAllowed) {
            $this->_getResource()->save($this);
            $this->_afterSave();
        }
        $this->_getResource()->addCommitCallback(array($this, 'afterCommitCallback'))
            ->commit();
        $this->_hasDataChanges = false;
        $dataCommited = true;
    } catch (Exception $e) {
        $this->_getResource()->rollBack();
        $this->_hasDataChanges = true;
        throw $e;
    }
    if ($dataCommited) {
        $this->_afterSaveCommit();
    }
    return $this;
}
```
