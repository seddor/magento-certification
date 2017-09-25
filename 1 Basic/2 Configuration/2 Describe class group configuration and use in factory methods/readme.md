# Describe class group configuration and use in factory methods

Фабричные методы:
```
 $helper                 = Mage::helper('core');
 $model                  = Mage::getModel('sales/order');
 $resourceModel          = Mage::getResourceModel('catalog/product');
 $modelSingleton         = Mage::getSingleton('sales/order');
 $resourceModelSingleton = Mage::getResourceSingleton('catalog/product');
```

### helper

```php
<?
public static function helper($name)
{
    $registryKey = '_helper/' . $name;
    if (!self::registry($registryKey)) {
        $helperClass = self::getConfig()->getHelperClassName($name);
        self::register($registryKey, new $helperClass);
    }
    return self::registry($registryKey);
}
```

примечание:
Если через слеш не задано точное имя хелепера, то в качестве имени будет использоваться `/data` - т.е.  `Mage::helper('core')` == ` Mage::helper('core/data')` == `Mage_Core_Helper_Data`.

### getModel

```php
<?
public static function getModel($modelClass = '', $arguments = array())
{
    return self::getConfig()->getModelInstance($modelClass, $arguments);
}
```

### getResourceModel

```php
<?
public static function getResourceModel($modelClass, $arguments = array())
{
    return self::getConfig()->getResourceModelInstance($modelClass, $arguments);
}
```

### getSingleton

```php
<?
public static function getSingleton($modelClass='', array $arguments=array())
{
    $registryKey = '_singleton/'.$modelClass;
    if (!self::registry($registryKey)) {
        self::register($registryKey, self::getModel($modelClass, $arguments));
    }
    return self::registry($registryKey);
}
```

### getResourceSingleton
```php
<?
public static function getResourceSingleton($modelClass = '', array $arguments = array())
{
    $registryKey = '_resource_singleton/'.$modelClass;
    if (!self::registry($registryKey)) {
        self::register($registryKey, self::getResourceModel($modelClass, $arguments));
    }
    return self::registry($registryKey);
}
```
