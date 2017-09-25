# Identify basic concepts of price generation in Magento

В мадженте для работы с ценой продукта используется специальный класс `Mage_Catalog_Model_Product_Type_Price`, он отвечает за вывод и подсчёт цены продукта. Для каждого типа продукта можно определить свой класс для цены. Помимо просто цены, есть ещё специальная цена, tier цена(т.е. специальная цена при покупке нескольких экземпляров продукта), цена для группы пользователей

# Modify and adjust price generation for products

Можно изменить получение цены следующим образом

  * Для новых типов продуктов:
    * создать свой класс для цены и поставить в `<price_model>`
  * Для существующих типов:
    * сделать реврайт для нужного класса цены
    * в базовом классе `Mage_Catalog_Model_Product_Type_Price` в методе `getFinalPrice`
    диспачится ивент `catalog_product_get_final_price`
    ```
    Mage::dispatchEvent('catalog_product_get_final_price', array('product' => $product, 'qty' => $qty));
    ```

## Under what circumstances are product prices read from the index tables?

При использовании колеекции продуктов (`Mage_Catalog_Model_Resource_Product_Collection`), при получении информации по цене вызывается метод `_prepareStatisticsData`
```php
<?
protected function _prepareStatisticsData()
{
    $select = clone $this->getSelect();
    $priceExpression = $this->getPriceExpression($select) . ' ' . $this->getAdditionalPriceExpression($select);
    $sqlEndPart = ') * ' . $this->getCurrencyRate() . ', 2)';
    $select = $this->_getSelectCountSql($select, false);
    $select->columns(array(
        'max' => 'ROUND(MAX(' . $priceExpression . $sqlEndPart,
        'min' => 'ROUND(MIN(' . $priceExpression . $sqlEndPart,
        'std' => $this->getConnection()->getStandardDeviationSql('ROUND((' . $priceExpression . $sqlEndPart)
    ));
    $select->where($this->getPriceExpression($select) . ' IS NOT NULL');
    $row = $this->getConnection()->fetchRow($select, $this->_bindParams, Zend_Db::FETCH_NUM);
    $this->_pricesCount = (int)$row[0];
    $this->_maxPrice = (float)$row[1];
    $this->_minPrice = (float)$row[2];
    $this->_priceStandardDeviation = (float)$row[3];

    return $this;
}
```
в нём вызваается `getPriceExpression`
```php
<?
public function getPriceExpression($select)
{
    if (is_null($this->_priceExpression)) {
        $this->_preparePriceExpressionParameters($select);
    }
    return $this->_priceExpression;
}

/**
 * Prepare additional price expression sql part
 *
 * @param Varien_Db_Select $select
 * @return Mage_Catalog_Model_Resource_Product_Collection
 */
protected function _preparePriceExpressionParameters($select)
{
    // prepare response object for event
    $response = new Varien_Object();
    $response->setAdditionalCalculations(array());
    $tableAliases = array_keys($select->getPart(Zend_Db_Select::FROM));
    if (in_array(self::INDEX_TABLE_ALIAS, $tableAliases)) {
        $table = self::INDEX_TABLE_ALIAS;
    } else {
        $table = reset($tableAliases);
    }

    // prepare event arguments
    $eventArgs = array(
        'select'          => $select,
        'table'           => $table,
        'store_id'        => $this->getStoreId(),
        'response_object' => $response
    );

    Mage::dispatchEvent('catalog_prepare_price_select', $eventArgs);

    $additional   = join('', $response->getAdditionalCalculations());
    $this->_priceExpression = $table . '.min_price';
    $this->_additionalPriceExpression = $additional;
    $this->_catalogPreparePriceSelect = clone $select;

    return $this;
}
```

## From which modules do classes participate in price calculation?

`Mage_Catalog_Model_Product_Type_Price`

## Which ways exist to specify custom prices during runtime?

Отловить событие `catalog_product_get_final_price`

## How do custom product options influence price calculation?

options price применяется при получение финальной цены в `getFinalPrice` вызывается `_applyOptionsPrice`
```php
<?
protected function _applyOptionsPrice($product, $qty, $finalPrice)
{
    if ($optionIds = $product->getCustomOption('option_ids')) {
        $basePrice = $finalPrice;
        foreach (explode(',', $optionIds->getValue()) as $optionId) {
            if ($option = $product->getOptionById($optionId)) {
                $confItemOption = $product->getCustomOption('option_'.$option->getId());

                $group = $option->groupFactory($option->getType())
                    ->setOption($option)
                    ->setConfigurationItemOption($confItemOption);
                $finalPrice += $group->getOptionPrice($confItemOption->getValue(), $basePrice);
            }
        }
    }

    return $finalPrice;
}
```

## How are product tier prices implemented and displayed?

Tier price хранится в `catalog_product_entity_tier_price`, цена считается при получении финальной цены в `_applyTierPrice`

```php
<?
protected function _applyTierPrice($product, $qty, $finalPrice)
{
    if (is_null($qty)) {
        return $finalPrice;
    }

    $tierPrice  = $product->getTierPrice($qty);
    if (is_numeric($tierPrice)) {
        $finalPrice = min($finalPrice, $tierPrice);
    }
    return $finalPrice;
}
```

## What is the priority of the different prices that can be specified for products (price, special price, group price, tier price, etc.)?


Цена получается в
```php
<?
public function getFinalPrice($qty = null, $product)
{
    if (is_null($qty) && !is_null($product->getCalculatedFinalPrice())) {
        return $product->getCalculatedFinalPrice();
    }

    $finalPrice = $this->getBasePrice($product, $qty);
    $product->setFinalPrice($finalPrice);

    Mage::dispatchEvent('catalog_product_get_final_price', array('product' => $product, 'qty' => $qty));

    $finalPrice = $product->getData('final_price');
    $finalPrice = $this->_applyOptionsPrice($product, $qty, $finalPrice);
    $finalPrice = max(0, $finalPrice);
    $product->setFinalPrice($finalPrice);

    return $finalPrice;
}

public function getBasePrice($product, $qty = null)
{
    $price = (float)$product->getPrice();
    return min($this->_applyGroupPrice($product, $price), $this->_applyTierPrice($product, $qty, $price),
        $this->_applySpecialPrice($product, $price)
    );
}
```

Как видно в `getBasePrice` будет в итоге выбрана самая минимальная цена среди group, tier и special price.
