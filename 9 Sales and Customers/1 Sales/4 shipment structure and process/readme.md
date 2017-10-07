# Describe the order shipment structure and process

Базовый класс для доставки — `Mage_Sales_Model_Order_Shipment`

## How shipment templates be customized?

Через xml лаяуты, email темлейты соотвено через email темплейты

## How can different items from a single order be shipped to multiple addresses? Is it possible at all?

Не виртальные продукты могут быть отправлены на разные адреса в одном заказе если разрешён multishipping checkout

## How does Magento store shipping and tracking information?

Информация хранится в этих таблицах:

  * sales_flat_shipment
  * sales_flat_shipment_comment
  * sales_flat_shipment_grid
  * sales_flat_shipment_item
  * sales_flat_shipment_track
