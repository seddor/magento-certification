# Describe order creation in the admin

Заказы можно создавать через форму в админке. Она состоит из следующих шагов:

1. Выбор клиента
2. Выбор стора
3. Введение деталей заказа
  * Продукты
  * Адрес доставки и выставления счёта
  * Метод доставки
  * Метод оплаты
4. Размещения заказа   

## Основные классы

Контролеры формы находятся в *app/code/core/Mage/Adminhtml/controllers/Sales/Order*

  * Mage_Adminhtml_Sales_Order_CreateController
  * Mage_Adminhtml_Sales_Order_EditController (наследует Mage_Adminhtml_Sales_Order_CreateController)
  * Mage_Adminhtml_Sales_Order_CreditmemoController
  * Mage_Adminhtml_Sales_Order_InvoiceController
  * Mage_Adminhtml_Sales_Order_ShipmentController
  * Mage_Adminhtml_Sales_Order_StatusController

## Создание заказа

Происходит в `Mage_Adminhtml_Sales_Order_CreateController->indexAction()`, в нём вызывается метод `_initSession()`

```php
<?
/**
 * Initialize order creation session data
 *
 * @return Mage_Adminhtml_Sales_Order_CreateController
 */
protected function _initSession()
{
    /**
     * Identify customer
     */
    if ($customerId = $this->getRequest()->getParam('customer_id')) {
        $this->_getSession()->setCustomerId((int) $customerId);
    }

    /**
     * Identify store
     */
    if ($storeId = $this->getRequest()->getParam('store_id')) {
        $this->_getSession()->setStoreId((int) $storeId);
    }

    /**
     * Identify currency
     */
    if ($currencyId = $this->getRequest()->getParam('currency_id')) {
        $this->_getSession()->setCurrencyId((string) $currencyId);
        $this->_getOrderCreateModel()->setRecollect(true);
    }

    //Notify other modules about the session quote
    Mage::dispatchEvent('create_order_session_quote_initialized',
            array('session_quote' => $this->_getSession()));

    return $this;
}
```

Шаги переключаются аяксами, блоки грузятся через метод `Mage_Adminhtml_Sales_Order_CreateController->loadBlockAction()`

Обработки данных экшенов формы вынесена в метод `_processActionData`

```php
<?
/**
    * Process request data with additional logic for saving quote and creating order
    *
    * @param string $action
    * @return Mage_Adminhtml_Sales_Order_CreateController
    */
   protected function _processActionData($action = null)
   {
       $eventData = array(
           'order_create_model' => $this->_getOrderCreateModel(),
           'request_model'      => $this->getRequest(),
           'session'            => $this->_getSession(),
       );

       Mage::dispatchEvent('adminhtml_sales_order_create_process_data_before', $eventData);

       /**
        * Saving order data
        */
       if ($data = $this->getRequest()->getPost('order')) {
           $this->_getOrderCreateModel()->importPostData($data);
       }

       /**
        * Initialize catalog rule data
        */
       $this->_getOrderCreateModel()->initRuleData();

       /**
        * init first billing address, need for virtual products
        */
       $this->_getOrderCreateModel()->getBillingAddress();

       /**
        * Flag for using billing address for shipping
        */
       if (!$this->_getOrderCreateModel()->getQuote()->isVirtual()) {
           $syncFlag = $this->getRequest()->getPost('shipping_as_billing');
           $shippingMethod = $this->_getOrderCreateModel()->getShippingAddress()->getShippingMethod();
           if (is_null($syncFlag)
               && $this->_getOrderCreateModel()->getShippingAddress()->getSameAsBilling()
               && empty($shippingMethod)
           ) {
               $this->_getOrderCreateModel()->setShippingAsBilling(1);
           } else {
               $this->_getOrderCreateModel()->setShippingAsBilling((int)$syncFlag);
           }
       }

       /**
        * Change shipping address flag
        */
       if (!$this->_getOrderCreateModel()->getQuote()->isVirtual() && $this->getRequest()->getPost('reset_shipping')) {
           $this->_getOrderCreateModel()->resetShippingMethod(true);
       }

       /**
        * Collecting shipping rates
        */
       if (!$this->_getOrderCreateModel()->getQuote()->isVirtual() &&
           $this->getRequest()->getPost('collect_shipping_rates')
       ) {
           $this->_getOrderCreateModel()->collectShippingRates();
       }


       /**
        * Apply mass changes from sidebar
        */
       if ($data = $this->getRequest()->getPost('sidebar')) {
           $this->_getOrderCreateModel()->applySidebarData($data);
       }

       /**
        * Adding product to quote from shopping cart, wishlist etc.
        */
       if ($productId = (int) $this->getRequest()->getPost('add_product')) {
           $this->_getOrderCreateModel()->addProduct($productId, $this->getRequest()->getPost());
       }

       /**
        * Adding products to quote from special grid
        */
       if ($this->getRequest()->has('item') && !$this->getRequest()->getPost('update_items') && !($action == 'save')) {
           $items = $this->getRequest()->getPost('item');
           $items = $this->_processFiles($items);
           $this->_getOrderCreateModel()->addProducts($items);
       }

       /**
        * Update quote items
        */
       if ($this->getRequest()->getPost('update_items')) {
           $items = $this->getRequest()->getPost('item', array());
           $items = $this->_processFiles($items);
           $this->_getOrderCreateModel()->updateQuoteItems($items);
       }

       /**
        * Remove quote item
        */
       $removeItemId = (int) $this->getRequest()->getPost('remove_item');
       $removeFrom = (string) $this->getRequest()->getPost('from');
       if ($removeItemId && $removeFrom) {
           $this->_getOrderCreateModel()->removeItem($removeItemId, $removeFrom);
       }

       /**
        * Move quote item
        */
       $moveItemId = (int) $this->getRequest()->getPost('move_item');
       $moveTo = (string) $this->getRequest()->getPost('to');
       if ($moveItemId && $moveTo) {
           $this->_getOrderCreateModel()->moveQuoteItem($moveItemId, $moveTo);
       }

       if ($paymentData = $this->getRequest()->getPost('payment')) {
           $this->_getOrderCreateModel()->getQuote()->getPayment()->addData($paymentData);
       }

       $eventData = array(
           'order_create_model' => $this->_getOrderCreateModel(),
           'request'            => $this->getRequest()->getPost(),
       );

       Mage::dispatchEvent('adminhtml_sales_order_create_process_data', $eventData);

       $this->_getOrderCreateModel()
           ->saveQuote();

       if ($paymentData = $this->getRequest()->getPost('payment')) {
           $this->_getOrderCreateModel()->getQuote()->getPayment()->addData($paymentData);
       }

       /**
        * Saving of giftmessages
        */
       $giftmessages = $this->getRequest()->getPost('giftmessage');
       if ($giftmessages) {
           $this->_getGiftmessageSaveModel()->setGiftmessages($giftmessages)
               ->saveAllInQuote();
       }

       /**
        * Importing gift message allow items from specific product grid
        */
       if ($data = $this->getRequest()->getPost('add_products')) {
           $this->_getGiftmessageSaveModel()
               ->importAllowQuoteItemsFromProducts(Mage::helper('core')->jsonDecode($data));
       }

       /**
        * Importing gift message allow items on update quote items
        */
       if ($this->getRequest()->getPost('update_items')) {
           $items = $this->getRequest()->getPost('item', array());
           $this->_getGiftmessageSaveModel()->importAllowQuoteItemsFromItems($items);
       }

       $data = $this->getRequest()->getPost('order');
       $couponCode = '';
       if (isset($data) && isset($data['coupon']['code'])) {
           $couponCode = trim($data['coupon']['code']);
       }
       if (!empty($couponCode)) {
           if ($this->_getQuote()->getCouponCode() !== $couponCode) {
               $this->_getSession()->addError(
                   $this->__('"%s" coupon code is not valid.', $this->_getHelper()->escapeHtml($couponCode)));
           } else {
               $this->_getSession()->addSuccess($this->__('The coupon code has been accepted.'));
           }
       }

       return $this;
   }
```

### Выбор клиента

Форма выбора клиента реализована в блоке `Mage_Adminhtml_Block_Sales_Order_Create_Customer`, она содержит в себе грид — `Mage_Adminhtml_Block_Sales_Order_Create_Customer_Grid`.

### Выбор стора

Форма выбора стора — `Mage_Adminhtml_Block_Sales_Order_Create_Store` содержит в себе `Mage_Adminhtml_Block_Sales_Order_Create_Store_Select` который наследуется от `Mage_Adminhtml_Block_Store_Switcher`

### Продукты

Для поиска и добавления продуктов используется блок `Mage_Adminhtml_Block_Sales_Order_Create_Search` в котором содежится грид продуктов — `Mage_Adminhtml_Block_Sales_Order_Create_Search_Grid`.

### Адреса клиента

Основной класс адресов — `Mage_Adminhtml_Block_Sales_Order_Create_Form_Address`, от него наследуются классы для адреса доставки(`Mage_Adminhtml_Block_Sales_Order_Create_Shipping_Address`) и адреса оплаты(`Mage_Adminhtml_Block_Sales_Order_Create_Billing_Address`)

Для заказов из админки нельзя использовать несколько адресов доставки(multishipping)

### Метод доставки

Класс контейнера — `Mage_Adminhtml_Block_Sales_Order_Create_Shipping_Method`, формы — `Mage_Adminhtml_Block_Sales_Order_Create_Shipping_Method_Form`

### Метод оплаты

Класс контейнера — `Mage_Adminhtml_Block_Sales_Order_Create_Billing_Method`, формы — `Mage_Adminhtml_Block_Sales_Order_Create_Billing_Method_Form`
