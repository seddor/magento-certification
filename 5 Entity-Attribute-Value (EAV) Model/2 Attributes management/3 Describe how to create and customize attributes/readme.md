# Describe how to create and customize attributes.

## create
```php
<?
$installer->addAttribute(
    Mage_Catalog_Model_Category::ENTITY, 'category_brand',
    [
        'group'          => 'General Information',
        'input'          => 'select',
        'source'         => 'oggetto_catalog/source_brand',
        'type'           => 'int',
        'label'          => 'Brand',
        'visible'        => 1,
        'required'       => 0,
        'user_defined'   => 1,
        'global'         => Mage_Catalog_Model_Resource_Eav_Attribute::SCOPE_GLOBAL,
    ]
);
```
## update
```php
<?
$this->startSetup();
$helper = Mage::helper("core");
// Entity Type Code, Attribute Code, Attribute, Value
$this->updateAttribute("colin_eav_blog", "content", "frontend_model", "colin_eav/attribute_frontend_content");
$this->endSetup();
```
## delete

```php
<?
$setup = Mage::getResourceModel('catalog/setup','catalog_setup');
$setup->removeAttribute('catalog_product','attr_code');
```
