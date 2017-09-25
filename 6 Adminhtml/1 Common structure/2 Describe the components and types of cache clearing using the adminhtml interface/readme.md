# Describe the components and types of cache clearing using the adminhtml interface

## At which moment does Magento check if a user is logged in or not?

В `Mage_Admin_Model_Observer->actionPreDispatchAdmin()`

## Which class do most Magento adminhtml blocks extend?

`Mage_Adminhtml_Block_Template`, он наследуется от `Mage_Core_Block_Template`. При рендере админских блоков генерируется ивент `adminhtml_block_html_before`

## What are the roles of adminhtml config?

Он содержит настройки ACL и админского меню

## What are the differences between the different cache types on the admin cache cleaning page?

Админка содержит следющие кеши:

| *Cache* | *Tag* |
| :------------- | :------------- |
| Configuration System | CONFIG |
| Layouts | LAYOUT_GENERAL_CACHE_TAG |
| Blocks HTML output | BLOCK_HTML |
| Translations | TRANSLATE |
| Collections Data | COLLECTION_DATA |
| EAV types and attributes | EAV |
| Web Services Configuration(api.xml) | CONFIG_API |
| Web Services Configuration(api2.xml) | CONFIG_API2 |
| Page Cache  | FPC |

## What is the difference between “Flush storage” and “Flush Magento Cache”?

Класс работы с кешем `Mage_Core_Model_Cache`

### Flush storage
```php
<?
public function flushAllAction()
{
    Mage::dispatchEvent('adminhtml_cache_flush_all');
    Mage::app()->getCacheInstance()->flush();
    $this->_getSession()->addSuccess(Mage::helper('adminhtml')->__("The cache storage has been flushed."));
    $this->_redirect('*/*');
}
```
Mage_Core_Model_Cache->flush()
### Flush Magento Cache
```php
<?
public function flushSystemAction()
{
    Mage::app()->cleanCache();
    Mage::dispatchEvent('adminhtml_cache_flush_system');
    $this->_getSession()->addSuccess(Mage::helper('adminhtml')->__("The Magento cache storage has been flushed."));
    $this->_redirect('*/*');
}
```
Zend_Cache_Core->clean()
## How you can clear the cache without using the UI?
```
Mage_Core_Model_Cache::flush
Mage_Core_Model_Cache::clean
Zend_Cache_Core::clean();
```
