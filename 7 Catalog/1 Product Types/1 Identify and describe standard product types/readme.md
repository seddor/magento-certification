# Identify and describe standard product types (simple, configurable, bundled, etc.)

В стандартной мадженте существует 6 типов продуктов:

  1. Simple — простой продукт, основной вид продуктов
  2. Grouped — группа продуктов, включается в себя насколько товаров разных типов (по умолчанию simple и virtual, это можно настроить через ноду `allow_product_types`)
  3. Configurable — настраиваемый продукт, имеет какой-либо настраиваемый атрибут, включает в себя несколько симлов с разными значениями этого аттриюута
  4. Virtual — виртуальный товар, например какая-нибудь карта магазина, не имеющая физического воплощения
  5. Downloadable — загружаемый товар
  6. Bundle — группа товаров в отличии от грповых товаров бандл нельзя купить по отдельности

Все типа продуктов настраиваются в `config.xml` в ноде global/catalog/product/type. Классы типов продуктов наследуются от `Mage_Catalog_Model_Product_Type_Abstract`.

## Классы типов продуктов

  1. Simple — Mage_Catalog_Model_Product_Type_Simple
  2. Grouped — Mage_Catalog_Model_Product_Type_Grouped
  3. Configurable — Mage_Catalog_Model_Product_Type_Configurable
  4. Virtual — Mage_Catalog_Model_Product_Type_Virtual
  5. Downloadable — Mage_Downloadable_Model_Product_Type наследует Mage_Catalog_Model_Product_Type_Virtual
  6. Bundle — Mage_Bundle_Model_Product_Type

Класс типа продукта можно получить с помощью `$product->getTypeInstance()`


## Особености некоторых типов

## Grouped

Таблица со связми продуктов в группы `catalog_product_link`

Example:

| link_id     | product_id     | linked_product_id     | link_type_id     |
| :------------- | :------------- | :------------- | :------------- |
| 1    | 3     | 1      | 3      |
| 2    | 3     | 2      | 3      |

  * product_id — групповой продукт
  * linked_product_id — симпл
  * link_type_id — тип ссылки из `catalog_product_link_type`. 3 обозначает `super`

В таблице `catalog_product_link` также хранятся отношения кроселов, апселов и релейтедов тип свзяи задаётся в — link_type_id

## Configurable

Таблица со связми продуктов в группы `catalog_product_super_link`

Example:

| link_id     | product_id     | parent_id     |
| :------------- | :------------- | :------------- |
| 1    | 1    | 4      |
| 1    | 2    | 2      |

Настраиваемый аттрибуты хранятся в `catalog_product_super_attribute`

Цены хранятся в `catalog_product_super_attribute_pricing`

### Downloadable

Линки для загрузки фалов хранятся в таблице `downloadable_link`, цены хранятся в `downloadable_link_price`
