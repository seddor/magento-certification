# Describe the programmatic structure of shipping methods, how to customize existing methods, and how to implement new methods

1. Конфиг метода
в config.xml
```xml
<config>
    <default>
        <carriers>
            <foo_shipping>
                <active>1</active>
                <model>foo_shipping/carrier</model>
                <title>Foo Shipping Carrier</title>
                <sort_order>10</sort_order>
                <sallowspecific>0</sallowspecific>
                <express_max_weight>1</express_max_weight>
            </foo_shipping>
        </carriers>
    </default>
</config>
```

  * sallowspecific — 0 для всех стран, Соответственно 1 — не для всех.
  * model — класс наследуемый от `Mage_Shipping_Model_Carrier_Abstract` и имплементирующий `Mage_Shipping_Model_Carrier_Interface`
2. Модель метода

```php
class Foo_Shipping_Model_Carrier
    extends Mage_Shipping_Model_Carrier_Abstract
    implements Mage_Shipping_Model_Carrier_Interface
{
    protected $_code = 'foo_shipping';

```

3. В моделе нужно реализовать метод `getAllowedMethods()`

```
public function getAllowedMethods()
{
    return array(
        'standard'    =>  'Standard delivery',
        'express'     =>  'Express delivery',
    );
}
```

4. Все ставки за доставку собираются в методе `collectRates(Mage_Shipping_Model_Rate_Request $request)`, он принмает информацию о доставке и возвращает итоговую ставку за доставку в виде экземпляра класса `Mage_Shipping_Model_Rate_Result`

```php
<?
class Foo_Shipping_Model_Carrier_Drone
    extends Mage_Shipping_Model_Carrier_Abstract
    implements Mage_Shipping_Model_Carrier_Interface
{

    protected $_code = 'foo_shipping';


    public function getAllowedMethods()
    {
        return array(
            'standard' => "Standard",
            'express'  => "Express"
        );
    }

    public function collectRates(Mage_Shipping_Model_Rate_Request $request)
    {
        if (!$this->getConfigFlag('active')) {
            return false;
        }

        //@var $result Mage_Shipping_Model_Rate_Result
        $result = Mage::getModel('shipping/rate_result');
        $result->append($this->_getStandardShippingRate());
        $result->append($this->_getExpressShippingRate());
        return $result;
    }


    protected function _getStandardShippingRate()
    {
        //@var $rate Mage_Shipping_Model_Rate_Result_Method
        $rate = Mage::getModel('shipping/rate_result_method');
        $rate->setCarrier($this->getCarrierCode());
        $rate->setMethodTitle("Standard");
        $rate->setMethod("standard");
        $rate->setPrice(50);
        return $rate;
    }

    protected function _getExpressShippingRate()
    {
        //@var $rate Mage_Shipping_Model_Rate_Result_Method
        $rate = Mage::getModel('shipping/rate_result_method');
        $rate->setCarrier($this->getCarrierCode());
        $rate->setMethodTitle("Express");
        $rate->setMethod("express");
        $rate->setPrice(100);
        return $rate;
    }
}
```
