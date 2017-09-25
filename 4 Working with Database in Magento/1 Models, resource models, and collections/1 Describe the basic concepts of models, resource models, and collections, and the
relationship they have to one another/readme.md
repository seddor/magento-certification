# Describe the basic concepts of models, resource models, and collections, and the relationship they have to one another

  * **Модель** — Содержит бизнес логику мадженты
  * **Ресурсная модель** — прослока меджду моделью и БД, здесь должна быть вся низкоуровневая рбаота с БД для одной модели
  * **Коллекция** — коллекция содержит несколько моделей, а также предоставляет основные действия по работе и фильтрации групп моделей

## Конфигурирование моделей

*app/code/local/Foo/Bar/etc/config.xml*

```xml
<global>
  <models>
    <foo_bar>
      <class>Foo_Bar_Model</class>
      <resourceModel>foo_bar_resource</resourceModel>
    </foo_bar>
    <foo_bar_resource>
      <class>Foo_Bar_Model_Resource</class>
      <entities>
        <baz>
          <table>baz</table>
        </baz>
      </entities>
    </foo_bar_resource>
  </models>
</global>
```

## Отношения

### Модель

```php
<?
class Foo_Bar_Model_Baz extends Mage_Core_Model_Abstract
{
    public function _construct()
    {
        parent::_construct();
        $this->_init('foo_bar/baz'); //resourceModel
    }
}
```

В методе $this->\_init($resourceModel) вызывается метод $this->\_setResourceModel($resourceModel):
```php
<?
protected function _setResourceModel($resourceName, $resourceCollectionName=null)
{
    $this->_resourceName = $resourceName;
    if (is_null($resourceCollectionName)) {
        $resourceCollectionName = $resourceName.'_collection';
    }
    $this->_resourceCollectionName = $resourceCollectionName;
}
```

### Ресурсная модель

```php
<?
class Foo_Bar_Model_Resource_Baz
    extends Mage_Core_Model_Resource_Db_Abstract
{
    public function _construct()
    {
        $this->_init('foo_bar/baz', 'id');//$tableName and $idFiledName
    }
}
```

### Коллекция

```php
<?
class Foo_Bar_Model_Resource_Baz_Collection
    extends Mage_Core_Model_Resource_Db_Collection_Abstract
{
    public function _construct()
    {
      $this->_init('foo_bar/baz');//$model
    }
}
```

```php
<?
protected function _init($model, $resourceModel = null)
{
    $this->setModel($model);
    if (is_null($resourceModel)) {
        $resourceModel = $model;
    }
    $this->setResourceModel($resourceModel);
    return $this;
}
```
