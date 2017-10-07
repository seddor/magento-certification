# Describe the differences in order creation between the frontend and the admin

## Which classes are involved in order creation in the admin? What are their roles (especially the role of adminhtml classes)?

см. 9.1

  * Основной контроллер формы создания — `Mage_Adminhtml_Sales_Order_CreateController`.
  * Большинство блоков формы наследуются от `Mage_Adminhtml_Block_Sales_Order_Create_Form_Abstract`
  * Модель для работы с квотой — `Mage_Adminhtml_Model_Sales_Order_Create`
  * Для квоты используются такой же класс что и для кроты на фронте `Mage_Sales_Model_Quote`
  * Для заказа используются такой же класс что и для кроты на фронте `Mage_Sales_Model_Order`

## How does Magento calculate price when an order is created from the admin?

По добавлению айтема в `Mage_Adminhtml_Sales_Order_CreateController->processActionData()`

```
if ($this->getRequest()->has('item') && !$this->getRequest()->getPost('update_items') && !($action == 'save')) {
    $items = $this->getRequest()->getPost('item');
    $items = $this->_processFiles($items);
    $this->_getOrderCreateModel()->addProducts($items);
}
```
при вызове `Mage_Adminhtml_Model_Sales_Order_Create->addProducts()` при добавлении продуктов проставляется флаг о необходимости пересчёта итогов, позже в `Mage_Adminhtml_Sales_Order_CreateController` происходит вызов

```
$this->_getOrderCreateModel()
    ->saveQuote();
```

В котором при необходимости инициализируется перерасчёт итогов

```php
<?
public function saveQuote()
{
    if (!$this->getQuote()->getId()) {
        return $this;
    }

    if ($this->_needCollect) {
        $this->getQuote()->collectTotals();
    }

    $this->getQuote()->save();
    return $this;
}
```

## Which steps are necessary in order to create an order from the admin?

см. 9.1
1. Выбор клиента
2. Выбор стора
3. Введение деталей заказа
  * Продукты
  * Адрес доставки и выставления счёта
  * Метод доставки
  * Метод оплаты
4. Размещения заказа   

## What happens when existing orders are edited in the admin?

В мадженте нельзя редактировать существующие заказы, при редактировании заказ пересоздаётся

## What is the difference between order status and order state?

state — это одно из основных внутренних состояний маджетовских заказов, к нему может быть привязано несколько статусов.

Состояния определённые в дефолтной мадженте

  1. new
  2. pending_payment
  3. processing
  4. complete
  5. closed
  6. canceled
  7. holded

State нельзя добавить через админку, в отличии от статусов.

Статусы хранятся в `sales_order_status`, а отношение стейтов и статусов в `sales_order_status_state`.
Связи можно настраивать через админку, или с помощью xml конфига

```xml
<config>
    <global>
        <sales>
            <order>
                <statuses>
                    <my_processing_status translate="label">
                        <label>My proccesing status</label>
                    </my_processing_status>
                </statuses>
                <states>
                    <processing>
                        <statuses>
                            <my_processing_status />
                        </statuses>
                    </processing>
                </states>
            </order>
        </sales>  
    </global>
</config>
```
