# Describe the process and configuration of class overrides in Magento

## 1. Модели, блоки, хелперы, ресурсные модели

Для реврайта нужно в соотвующей ноде в конфиге модуля прописать:
для моделей
```xml
<global>
  <models>
    <sales>
      <rewrite>
        <order>Certification_Sales_Model_Order</order>
      </rewrite>
    </sales>
  </models>
</global>
```
вместо models могут быть blocks, helpers или нода ресурснфых моделей

## 2. Контроллеров

Реврайт контролеров отличается, маджента позволяет заменить контролер, или использовать кастомный до/после реврайченого

### Замена

Нужно в config.xml модуля написать

```xml
<frontend>
    <routers>
        <catalog>
            <use>standard</use>
            <args>
                <module>Certification_Catalog</module>
                <frontName>catalog</frontName>
            </args>
        </catalog>
    </routers>
</frontend>
```

Для реврайта ProductController

* создать app/code/local/Certification/Catalog/controllers/ProductController.php
* создать класс `Certification_Catalog_ProductController` унаследованый от `Mage_Catalog_ProductController` в файле класса зареквайрить контролер от которого идёт наследование
```
require_once MAGENTO_ROOT . '/app/code/core/Mage/Catalog/controllers/ProductController.php';
```

### before/after
```
<frontend>
    <routers>
        <catalog>
            <args>
                <modules>
                    <colin_request before="Mage_Catalog">Certification_Catalog</colin_request>
                </modules>
            </args>
        </catalog>
    </routers>
</frontend>
```

дальше анологично реврайту из замемены
