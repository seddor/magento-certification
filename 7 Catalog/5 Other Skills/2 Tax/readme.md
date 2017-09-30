# Implement, troubleshoot, and modify Magento tax rules

## Классы налогов

Бывают 2х видов:

  * Product
  * Customer Group

Классы добавляются в продукты и группы а затем используются для высчитывания налоговой ставки.
Классы хранятся в таблице `tax_class`, таблица имеет 3 поля(ид, имя налогового класса и тип(продукт, кастомер)). Модель в мадженте для работы с классами — `Mage_Tax_Model_Class`, это стандартная маджентовская модель(`Mage_Core_Model_Abstract`), не содержит каких-либо специальных методов, в ней определены только две константы для типов налогово класса(продукт, кастомер).

Налоговый класс обязательное поля при создании кастомерской группы и большинства типов продуктов, кроме тех, которые не используют цены, например товары-группы.

В дефолтной установке мадженты у продуктов есть три налоговых класса, *Taxable Goods* и *Shipping*, помимо этого у продуктов можно выбрать налоговый класс *none*, он обозначает, что продукт не облагается налогом.

## Налоговые ставки

Налоговые ставки определяются относительно адреса. В зависимости от настроек магазина адресе доставки(или платежном) или на адресе магазина. Налоговые ставки определяются для страны, штата, и определёного почтового индекса

Модель налоговых ставок — `Mage_Tax_Model_Calculation_Rate`, в его `_beforeDelete` методе выбрасывается экзепшен, если налоговая ставка привязана к налоговому правилу.
Таблица — `tax_calculation_rate` + отдельная таблица `tax_calculation_rate_title` с названиями ставок для разных сторов.

## Налоговое правило

Налоговые правила используются для связи налоговых классов продуктов, классов кастомеров и налоговых ставок, причём в одной правиле могут быть связаны несколько классов и ставок. Если в правиле будет выбрано несколько налоговых ставок и при подсчёте налогов в результате будут подходить несколько налоговых ставок, маджента выберет наибольшую из них. Также в налоговом правиле можно указать его приоритет, используемый при подсчёте налога в заказе. При совпадении приоритетов правила применяются вместе(например правило 10% и 5% в итоге дадут 15% налог)

Модель правила — `Mage_Tax_Model_Calculation_Rule`. Метод `saveCalculationData()`, вызываемый при сохранении правила:
```php
<?
public function saveCalculationData()
{
    $ctc = $this->getData('tax_customer_class');
    $ptc = $this->getData('tax_product_class');
    $rates = $this->getData('tax_rate');

    Mage::getSingleton('tax/calculation')->deleteByRuleId($this->getId());
    foreach ($ctc as $c) {
        foreach ($ptc as $p) {
            foreach ($rates as $r) {
                $dataArray = array(
                    'tax_calculation_rule_id'   =>$this->getId(),
                    'tax_calculation_rate_id'   =>$r,
                    'customer_tax_class_id'     =>$c,
                    'product_tax_class_id'      =>$p,
                );
                Mage::getSingleton('tax/calculation')->setData($dataArray)->save();
            }
        }
    }
}
```

Он сохраняет подсчитанные данные налогов в таблицу `tax_calculation`.


## Tax request

Когда продукту нужна налоговая ставка, маджента генерирует tax request — Varien_Object, содержащий данные адреса и ид налогового класса клиента.
```php
<?
/**
 * Get request object with information necessary for getting tax rate
 * Request object contain:
 *  country_id (->getCountryId())
 *  region_id (->getRegionId())
 *  postcode (->getPostcode())
 *  customer_class_id (->getCustomerClassId())
 *  store (->getStore())
 *
 * @param   null|false|Varien_Object $shippingAddress
 * @param   null|false|Varien_Object $billingAddress
 * @param   null|int $customerTaxClass
 * @param   null|int $store
 * @return  Varien_Object
 */
public function getRateRequest(
    $shippingAddress = null,
    $billingAddress = null,
    $customerTaxClass = null,
    $store = null)
{
    if ($shippingAddress === false && $billingAddress === false && $customerTaxClass === false) {
        return $this->getRateOriginRequest($store);
    }
    $address = new Varien_Object();
    $customer = $this->getCustomer();
    $basedOn = Mage::getStoreConfig(Mage_Tax_Model_Config::CONFIG_XML_PATH_BASED_ON, $store);

    if (($shippingAddress === false && $basedOn == 'shipping')
        || ($billingAddress === false && $basedOn == 'billing')
    ) {
        $basedOn = 'default';
    } else {
        if ((($billingAddress === false || is_null($billingAddress) || !$billingAddress->getCountryId())
            && $basedOn == 'billing')
            || (($shippingAddress === false || is_null($shippingAddress) || !$shippingAddress->getCountryId())
                && $basedOn == 'shipping')
        ) {
            if ($customer) {
                $defBilling = $customer->getDefaultBillingAddress();
                $defShipping = $customer->getDefaultShippingAddress();

                if ($basedOn == 'billing' && $defBilling && $defBilling->getCountryId()) {
                    $billingAddress = $defBilling;
                } else if ($basedOn == 'shipping' && $defShipping && $defShipping->getCountryId()) {
                    $shippingAddress = $defShipping;
                } else {
                    $basedOn = 'default';
                }
            } else {
                $basedOn = 'default';
            }
        }
    }

    switch ($basedOn) {
        case 'billing':
            $address = $billingAddress;
            break;
        case 'shipping':
            $address = $shippingAddress;
            break;
        case 'origin':
            $address = $this->getRateOriginRequest($store);
            break;
        case 'default':
            $address
                ->setCountryId(Mage::getStoreConfig(
                Mage_Tax_Model_Config::CONFIG_XML_PATH_DEFAULT_COUNTRY,
                $store))
                ->setRegionId(Mage::getStoreConfig(Mage_Tax_Model_Config::CONFIG_XML_PATH_DEFAULT_REGION, $store))
                ->setPostcode(Mage::getStoreConfig(
                Mage_Tax_Model_Config::CONFIG_XML_PATH_DEFAULT_POSTCODE,
                $store));
            break;
    }

    if (is_null($customerTaxClass) && $customer) {
        $customerTaxClass = $customer->getTaxClassId();
    } elseif (($customerTaxClass === false) || !$customer) {
        $customerTaxClass = Mage::getModel('customer/group')
                ->getTaxClassId(Mage_Customer_Model_Group::NOT_LOGGED_IN_ID);
    }

    $request = new Varien_Object();
    $request
        ->setCountryId($address->getCountryId())
        ->setRegionId($address->getRegionId())
        ->setPostcode($address->getPostcode())
        ->setStore($store)
        ->setCustomerClassId($customerTaxClass);
    return $request;
}
```

## Отображение цен

Показ налогов для цен настраивается в конфиге. Товары могут отображаться с налогом, без него, с уже включеным налогом в цену.


http://www.divisionlab.com/solvingmagento/taxes-in-magento-module-mage_tax/
