# Describe how to extend the Magento multishipping implementation

Мультишипинг чекаут позволяет отправлять продукты на разные адреса в одном заказе.

его включение настраивается в админке, так-же там указывается макимальное количество продуктов доступных к заказу в мультишипинге. Мкльтшипнг доступен только для зарегистрированных пользователей с адресом.

Блоки чекаута находятся в **app/code/core/Mage/Checkout/Block**, основные:

  * Select Addresses – Addresses.php
  * Shipping Information – Shipping.php
  * Billing Information – Billing.php
  * Place Order – Overview.php
  * Order Success – Success.php

Контролер для работы с адресами на мультшипинг чекауте — `Mage_Checkout_Multishipping_AddressController`
Модель для мультишипинга — `Mage_Checkout_Model_Type_Multishipping`


http://www.divisionlab.com/solvingmagento/magento-multishipping-checkout/

# Identify limitations of the multishipping implementation

## How does the storage of quotes for multishipping and onepage checkouts differ?

Информация с адресами доставки айтемов хранится `sales_flat_quote_address_item`

## Which quotes in a multishipping checkout flow will be virtual?

Для виртуальных айтемов используется билинг адрес.

## What is the difference in the multishipping processing for a quote with virtual products in it?

Для виртуальных атемов не вводится адрес доставки.

## How can different product types be split among multiple addresses when using multishipping in Magento?

Айтемы добавляются в `sales_flat_quote_address_item` и ссылаются на родительской адрес в `sales_flat_quote_address`

## How many times are total models executed on a multishipping checkout in Magento?

Для каждого адреса

## Which model is responsible for multishipping checkout in Magento?

Модель для мультишипинга — `Mage_Checkout_Model_Type_Multishipping`

http://magecert.com/checkout.html
