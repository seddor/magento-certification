# Card operations (capturing and authorization)

## Which classes and methods are responsible for credit card operations (for example authorization or capturing)?

Основной класс для платёжных методов — `Mage_Payment_Model_Method_Abstract`, также есть класс для рабрты с кредитными картами — `Mage_Payment_Model_Method_Cc`

## What is the difference between “pay” and “capture” operations?

capture — собирает плтаёжную информацию
pay — производит обновление итогов для инвойса

## What are the roles of the order, order_payment, invoice, and payment methods in the process of charging a card?

1. Order: сохраняет информацию о заказе
2. Order payment: предоставляет интерфейс к фактическому способу оплаты
3. Invoice: Сохранение информации об оплате клиентом
4. Payment methods: оплата

## What are the roles of the total models in the context of the invoice object?

Пересчёт цен после завершения транзакции

## How does Magento store information about invoices?

Хранится в таблице `sales_flat_invoice`
