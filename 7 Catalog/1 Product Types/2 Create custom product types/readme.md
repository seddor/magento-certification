# Create custom product types from scratch or modify existing product types

## Конфиг

конфиг в config.xml

```xml
<global>
    <catalog>
        <product>
            <type>
              <accessory translate="label" module="foo_catalog">
                  <label>Accessory</label>
                  <model>foo_catalog/product_type_accessory</model>
                  <composite>0</composite>
                  <is_qty>1</is_qty>
                  <can_use_qty_decimals>0</can_use_qty_decimals>
                  <index_priority>100</index_priority>
                  <price_model>colin_catalog/product_price</price_model>
              </accessory>
            </type>
        </product>
    </catalog>
  </global>
```

  * model — класс модели типа, должен наследоваться от `Mage_Catalog_Model_Product_Type_Abstract`
  * composite — для продуктов, которые включают в себя другие продукты, например как групповые и конфигурируемые товары
  * can_use_qty_decimals — может ли продукт типа быть дробного количества
  * price_model — отвечает за подсчёт цены товара населдуется от `Mage_Catalog_Model_Product_Type_Price`

Если продукт составной (composite = 1) то ещё можно указать типы входящих в него продуктов:

```xml
<allow_product_types>
    <simple/>
</allow_product_types>
```

## Модель типа

класс наследуемый от `Mage_Catalog_Model_Product_Type_Abstract`
```php
class Foo_Catalog_Model_Product_Type_Accessory extends Mage_Catalog_Model_Product_Type_Abstract
{
}

```

в этом классе можно определить специфичные для типа методы, из объекта продукта можно получить с помощью `getTypeInstance()`

## Установка аттрибутов

В инсталяционном скрипте можно создать аттрибуты специфичные для типа и добавить их в новую группу:

```php
<?
$entityTypeId = $setup->getEntityTypeId('catalog_product');


    $attributeSet = Mage::getModel('eav/entity_attribute_set')
       ->setEntityTypeId($entityTypeId)
       ->setAttributeSetName('Accessory');
    $attributeSet->validate();
    $attributeSet->save();
    $attributeSet->initFromSkeleton($entityTypeId)->save();

   $setup->addAttributeGroup(Mage_Catalog_Model_Product::ENTITY, 'Accessory', 'Accessory Info');

   $setup->addAttribute($entityTypeId, 'accessory_brand', [
       'type'         => 'varchar',
       'input'        => 'multiselect',
       'label'        => 'Brands',
       'source'       => 'oggetto_catalog/product_attribute_source_brand',
       'backend'      => 'eav/entity_attribute_backend_array',
       'global'       => Mage_Catalog_Model_Resource_Eav_Attribute::SCOPE_GLOBAL,
       'required'     => false,
       'user_defined' => true,
       'filterable'   => 1,
       'used_in_product_listing' => true,
   ]);
   $setup->addAttributeToGroup(
       $entityTypeId,
       'Accessory',
       'Accessory Info',
       'accessory_brand'
   );

   //apply price
   $attributes = [
    'price', 'special_price',
    'group_price', 'tier_price',
    'special_from_date',
    'special_to_date'];
foreach ($attributes as $attributeCode) {
    $applyTo = explode(
        ',',
        $setup->getAttribute(
            Mage_Catalog_Model_Product::ENTITY,
            $attributeCode,
            'apply_to'
        )
    );
    if (!in_array('accessory', $applyTo)) {
        $applyTo[] = 'accessory';
    }
    $setup->updateAttribute(
        Mage_Catalog_Model_Product::ENTITY,
        $attributeCode,
        'apply_to',
        implode(',', $applyTo)
    );
}
```

## Price model

Класс может наследоватся от `Mage_Catalog_Model_Product_Type_Price` который имеет следующие паблик-методы

```
getPrice($product)
getBasePrice($product, $qty = null)
getFinalPrice($qty = null, $product)
getChildFinalPrice($product, $productQty, $childProduct, $childProductQty)
getGroupPrice($product)
getTierPrice($qty = null, $product)
getTierPriceCount($product)
getFormatedTierPrice($qty=null, $product)
getFormatedPrice($product)
```

## Дополнительно

http://www.divisionlab.com/solvingmagento/creating-a-custom-product-type-in-magento/
