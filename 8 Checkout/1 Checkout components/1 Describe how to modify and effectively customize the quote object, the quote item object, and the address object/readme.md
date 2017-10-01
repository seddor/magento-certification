# Describe how to modify and effectively customize the quote object, the quote item object, and the address object

## What is the quote model used for in Magento?

Квота используется для хранения продуктов добавленных в корзину, в квоте выполняется подсчёт итогов, хранится информации об адресе доставки и оплаты.
`Mage_Sales_Model_Quote` — модель квоты.


## What is the shopping cart model used for in Magento?

Корзина используется для взаимодействия с квотой.

`Mage_Checkout_Model_Cart` — модель квоты, имплиментит `Mage_Checkout_Model_Cart_Interface`.
```php
<?
interface Mage_Checkout_Model_Cart_Interface
{
    /**
     * Add product to shopping cart (quote)
     *
     * @param   int|Mage_Catalog_Model_Product $productInfo
     * @param   mixed                          $requestInfo
     * @return  Mage_Checkout_Model_Cart_Interface
     */
    public function addProduct($productInfo, $requestInfo = null);

    /**
     * Save cart
     *
     * @abstract
     * @return Mage_Checkout_Model_Cart_Interface
     */
    public function saveQuote();

    /**
     * Associate quote with the cart
     *
     * @abstract
     * @param $quote Mage_Sales_Model_Quote
     * @return Mage_Checkout_Model_Cart_Interface
     */
    public function setQuote(Mage_Sales_Model_Quote $quote);

    /**
     * Get quote object associated with cart
     * @abstract
     * @return Mage_Sales_Model_Quote
     */
    public function getQuote();
}
```

Класс корзины не является стандартоной моделью мадженты, и наследуется от `Varien_Object`.

## How does Magento store information about the shopping cart?

Инфомация хранится в Quote, она распределена между этими таблицами:

  * sales_flat_quote
  * sales_flat_quote_address
  * sales_flat_quote_address_item
  * sales_flat_quote_item
  * sales_flat_quote_payment
  * sales_flat_quote_item_option
  * sales_flat_quote_shipping_rate

## How are different product types processed when added to the cart?

При добавлении продукта в квоту у модели типа вызывается метод `prepareForCartAdvanced`
```php
<?
public function prepareForCartAdvanced(Varien_Object $buyRequest, $product = null, $processMode = null)
{
    if (!$processMode) {
        $processMode = self::PROCESS_MODE_FULL;
    }
    $_products = $this->_prepareProduct($buyRequest, $product, $processMode);
    $this->processFileQueue();
    return $_products;
}
```
логика добавления конкретного типа продукта реализована в `_prepareProduct`.

## What is the difference between shipping and billing address objects in Magento? How is each used in the quote object?

Для работы с обоими типами адресов используется класс — `Mage_Sales_Model_Quote_Address`. Адреса используются при подсчёте итогов, в виртуальных заказах(содержащие виртуальные продукты) основным будет является билинг адрес, в остальных случаях шипинг.

## What is the difference in processing quote items for onepage and multishipping checkout in Magento?

  * onepage — заказ отправляется на один адрес, класс — `Mage_Checkout_Model_Type_Onepage`
  * multishipping — заказ может иметь несколько адресов получения, класс — `Mage_Checkout_Model_Type_Multishipping`

## How does Magento process additional information about products being added to the shopping cart (custom options, components of configurable products, etc.)?

Для дочерних продуктов конфигуреблов у айтемов есть поле `parent_item_id`.
Вся дополнительная информация хранится `sales_flat_quote_item_option`

В этой таблице для каждого айтема квоты есть несколько записей с дополнительной информацией.

  * info_buyRequest — есть у каждого айтема, хранится что-то вроде
  ```
a:6:{s:4:"uenc";s:96:"aHR0cDovLzUwLjU2LjIxMy4xNTcvaW5kZXgucGhwL2ZyZW5jaC1jdWZmLWNvdHRvbi10d2lsbC1veGZvcmQtNDIwLmh0bWw,";s:7:"product";s:3:"402";s:15:"related_product";s:0:"";s:15:"super_attribute";a:2:{i:92;s:2:"22";i:180;s:2:"78";}s:3:"qty";s:1:"1";s:10:"return_url";s:0:"";}
  ```
  * custom options — хранятся в строках с кодами типа option_2, option_1, и содержат значения опции
  * строки с кодом attributes хранят информацию у выбраных в конфигуреблах значений аттрибутов:
  ```
a:2:{i:92;s:2:"22";i:180;s:2:"78";}
  ```

## How do products in the shopping cart affect the checkout process?

Продукты без адреса доставки(виртуальные) не требуют этот адрес в чекауте, в качестве основного у них испрользуется билинг адрес.

## How can the billing and shipping addresses affect the checkout process?

Адреса используются при подсчёте итогов заказа, т.е. влияют на его величину.

## When exactly does inventory decrementing occur?

1. Перед размещением заказа в квоте вызвается метод `submitOrder`
2. в нём диспатчится ивент `sales_model_service_quote_submit_before` с параметрами 'order' => $order, 'quote' => $quote.
3. Ивент отлавливается в
```xml
<sales_model_service_quote_submit_before>
    <observers>
        <inventory>
            <class>cataloginventory/observer</class>
            <method>subtractQuoteInventory</method>
        </inventory>
    </observers>
</sales_model_service_quote_submit_before>
```
```php
<>
public function subtractQuoteInventory(Varien_Event_Observer $observer)
{
    $quote = $observer->getEvent()->getQuote();

    // Maybe we've already processed this quote in some event during order placement
    // e.g. call in event 'sales_model_service_quote_submit_before' and later in 'checkout_submit_all_after'
    if ($quote->getInventoryProcessed()) {
        return;
    }
    $items = $this->_getProductsQty($quote->getAllItems());

    /**
     * Remember items
     */
    $this->_itemsForReindex = Mage::getSingleton('cataloginventory/stock')->registerProductsSale($items);

    $quote->setInventoryProcessed(true);
    return $this;
}
```

## When exactly does card authorization and capturing occur?

при выполнении оплаты заказа вызывается метод `Mage_Sales_Model_Order_Payment->place()` в котором вызываются соответствующие методы в методу оплаты
