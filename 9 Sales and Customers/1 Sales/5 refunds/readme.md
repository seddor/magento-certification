# Describe the architecture and processing of refunds

Процес возврата построен на работе с credit memos. Только для заказов с выставленым счётом можно сделать возврат.

## Which classes are involved, and which tables are used to store refund information?

Основной класс — `Mage_Sales_Model_Order_Creditmemo`.
Таблицы для информации о возвратах:

 * sales_flat_creditmemo
 * sales_flat_creditmemo_comment
 * sales_flat_creditmemo_grid
 * sales_flat_creditmemo_item
 * sales_refunded_aggregated
 * sales_refunded_aggregated_order

## How does Magento process taxes when refunding an order?

Налоги настраиваются через админку.

## How does Magento process shipping fees when refunding an order?

Управляется в моделе Credit Memo при подсчёте различных итогов.
Изначально сумма равная сумме заказа с ценами за доставку, но при возврате в админке это можно изменить

## What is the difference between online and offline refunding?

При онлайновом возврате, действия происходят в рамках системы.

При офлайноывом возврате, возврат происходит за рамками системы

## What is the role of the credit memo total models in Magento?

Подсчёт обещей суммы заказа после возврата
