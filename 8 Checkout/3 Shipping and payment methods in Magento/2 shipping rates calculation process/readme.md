# Describe the shipping rates calculation process

## How does Magento calculate shipping rates?

Подсчёт цена за доставку происходит в `Mage_Shipping_Model_Shipping->collectRates()`

```php
<?
public function collectRates(Mage_Shipping_Model_Rate_Request $request)
{
    $storeId = $request->getStoreId();
    if (!$request->getOrig()) {
        $request
            ->setCountryId(Mage::getStoreConfig(self::XML_PATH_STORE_COUNTRY_ID, $request->getStore()))
            ->setRegionId(Mage::getStoreConfig(self::XML_PATH_STORE_REGION_ID, $request->getStore()))
            ->setCity(Mage::getStoreConfig(self::XML_PATH_STORE_CITY, $request->getStore()))
            ->setPostcode(Mage::getStoreConfig(self::XML_PATH_STORE_ZIP, $request->getStore()));
    }

    $limitCarrier = $request->getLimitCarrier();
    if (!$limitCarrier) {
        $carriers = Mage::getStoreConfig('carriers', $storeId);

        foreach ($carriers as $carrierCode => $carrierConfig) {
            $this->collectCarrierRates($carrierCode, $request);
        }
    } else {
        if (!is_array($limitCarrier)) {
            $limitCarrier = array($limitCarrier);
        }
        foreach ($limitCarrier as $carrierCode) {
            $carrierConfig = Mage::getStoreConfig('carriers/' . $carrierCode, $storeId);
            if (!$carrierConfig) {
                continue;
            }
            $this->collectCarrierRates($carrierCode, $request);
        }
    }

    return $this;
}
```

Цена за доставку конкретным курьером высчитывается в методе `collectRates(Mage_Shipping_Model_Rate_Request $request)` метода доставки. Он возвращает экземпляр класса `Mage_Shipping_Model_Rate_Result`.

## What is the hierarchy of shipping information in Magento?

Информация о итоговых ценах на доставку для конкретного адреса хранится в `sales_flat_quote_shipping_rate`.

## How does the TableRate shipping method work?

TableRate экспортируются из CSV хранится в таблице `shipping_tablerate`.

поля таблицы

  * website_id
  * dest_country_id
  * dest_region_id
  * dest_zip
  * condition_name
  * condition_value
  * price
  * cost

Выражения:

  * Price vs Destination - code: price
  * Weight vs Destination - code: weight
  * Items vs Destination - code: qty

Сам tableRate представлен отдельным курьером с моделью `Mage_Shipping_Model_Carrier_Tablerate`

## How do US shipping methods (FedEX, UPS, USPS) work?

Методы реализованы в модуле `Mage_Usa`, имеются в иерархии общий абстрактный класс `Mage_Usa_Model_Shipping_Carrier_Abstract`.
