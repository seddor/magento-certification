# Identify how catalog price rules are implemented in Magento

В мадженте есть два типа ценовых правил

  * Catalog Price Rules — применяются к продукту
  * Shopping Cart Rules — применяются к корзине (будет рассмотрено позже)

Продукты на которые будет действовать правило определяется его выражением, в нём указывается по каким критериям отбирать продукты, выражения можно комбинировать между собой.

Поддерживаются следующие выражения:
  * is
  * is not
  * greater than
  * equals or greater than
  * equals or less than
  * greater than
  * less than
  * contains
  * does not contain
  * is one of
  * is not one of

Действие которое будет происходить по правилу может быть 4х типов:
  * By Percentage of the Original Price — отнять процент от первоначальной цены
  * By Fixed Amount — отнять фиксированную сумму
  * To Percentage of the Original Price — процент от первоначальной цены
  * To Fixed Amount — фиксированная цена


## How are the catalog price rules related to the product prices?

Модуль **Mage_CatalogRule** отлавливает ивенты получения цены у продуктов:
```xml
<frontend>
  <events>
    <catalog_product_get_final_price>
      <observers>
        <catalogrule>
          <class>catalogrule/observer</class>
          <method>processFrontFinalPrice</method>
        </catalogrule>
      </observers>
    </catalog_product_get_final_price>
    <prepare_catalog_product_collection_prices>
      <observers>
        <catalogrule>
          <class>catalogrule/observer</class>
          <method>prepareCatalogProductCollectionPrices</method>
        </catalogrule>
      </observers>
    </prepare_catalog_product_collection_prices>
  </events>
</frontend>
```

в обзервере, вызвается метод `Mage::getResourceModel('catalogrule/rule')->getRulePrice()`
```php
<?
public function getRulePrice($date, $wId, $gId, $pId)
{
    $data = $this->getRulePrices($date, $wId, $gId, array($pId));
    if (isset($data[$pId])) {
        return $data[$pId];
    }

    return false;
}

public function getRulePrices($date, $websiteId, $customerGroupId, $productIds)
{
    $adapter = $this->_getReadAdapter();
    $select  = $adapter->select()
        ->from($this->getTable('catalogrule/rule_product_price'), array('product_id', 'rule_price'))
        ->where('rule_date = ?', $this->formatDate($date, false))
        ->where('website_id = ?', $websiteId)
        ->where('customer_group_id = ?', $customerGroupId)
        ->where('product_id IN(?)', $productIds);
    return $adapter->fetchPairs($select);
}
```    
В итоге если полученная цена, меньше, чем та, что была у продукта первоначально, то применяется она

Таблица с ценами `catalogrule_product_price` заполняется при индексе правил.

Применения всех правил каталога происходит в `Mage_CatalogRule_Model_Rule->applyAll()`
```php
<?
public function applyAll()
{
    $this->getResourceCollection()->walk(array($this->_getResource(), 'updateRuleProductData'));
    $this->_getResource()->applyAllRules();
    $this->_invalidateCache();
    $indexProcess = Mage::getSingleton('index/indexer')->getProcessByCode('catalog_product_price');
    if ($indexProcess) {
        $indexProcess->reindexAll();
    }
}
```

## How are the catalog price rules stored in the database?

  * `catalogrule` — таблица с правилами
  * `catalogrule_customer_group` — таблица с отношениями правил и пользовательских групп
  * `catalogrule_group_website` — таблица с отношениями правил, сайтов и групп
  * `catalogrule_product` — связь продуктов и правил, также хранит условия применяемые к продукту и скидку
  * `catalogrule_product_price` - хранит цены продуктов с применёными правилами
