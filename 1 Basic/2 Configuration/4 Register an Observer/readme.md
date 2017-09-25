# Register an Observer

## 1. Генерация события для обзервера

События генерируются с помощью метода `Mage::dispatchEvent($name, array $data = [])`
Например,
```
Mage::dispatchEvent('catalog_product_get_final_price', array('product' => $product, 'qty' => $qty));
```

## 2. Установка обработчика на событие

делается через config.xml модуля.

```xml
<global>
  <events>
    <catalog_product_get_final_price>
      <observers>
        <duplicate_price>
          <type>singleton</type>
          <class>Certification_Catalog_Model_Observer_Price</class><!-- если определены элиасы модели то можно cert_catalog/observer_price -->
          <method>duplicateFinalPrice</method>
        </certification_duplicate_price>
      </observers>
    </catalog_product_get_final_price>
  </events>
</global>
```

* вместо `global` может быть указана конкретная area
* `type` - может быть
  * singleton - или лбое другое, кроме перечиаслного ниже, если тип не указан, то тоже используется singleton, в данном случаее для вызова модели будет использоватся `Mage::getSingleton`
  * disabled - отключить отлавливание ивента, так-же можно отключать отлавливание маджентовских ивентов
  * object - для вызова модели будет использовано `Mage::getModel`
  * model - тоже самое, что и model
* `certification_duplicate_price` - это код обзервера ивента, должен быть кникальных

Сама модель обработчика создаётся в указаном в конфиге классе и методе, класс не наследуется не от чего

```php
<?
class Certification_Catalog_Model_Observer_Price
{

	/**
	 * Duplicates the final Price
	 *
	 * @param Varien_Event_Observer $observer
	 */
	public function duplicateFinalPrice(Varien_Event_Observer $observer)
	{
		$price = $observer->getEvent()->getProduct()->getFinalPrice(); //$observer->getProduct()->getFinalPrice()
		$price = $price * 2;
		$observer->getEvent()->getProduct()->setFinalPrice($price);
	}

}
```

Вызов всех обработчиков происходит здесь в `Mage_Core_Model_App->dispatchEvent($eventName, $args)`
