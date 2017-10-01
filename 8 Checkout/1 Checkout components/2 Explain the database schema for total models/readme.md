# Explain the database schema for total models

## What are total models responsible for in Magento?

Отвечает за итоговой суммы за заказ, сюда входит: подсчёт налогов, учитывание скидок, оплаты доставки и т.д.

## How you can customize total models?

1. Регистрация кастомных total models в системе
в config.xml
```xml
<config>
  <global>
    <sales>
      <quote>
        <totals>
          <[total_identify]><!-- Identify of the total model -->
            <class>[your_extension]/[quote_total_model]</class>
            <after>wee,discount,tax,tax_subtotal,grand_total</after>
          </[total_identify]>
        </totals>
      </quote>
      <order_invoice>
        <totals>
          <[total_identify]>
            <class>[your_extension]/[invoice_total_model]</class>
          </[total_identify]>
        </totals>
      </order_invoice>
      <order_creditmemo>
        <totals>
          <[total_identify]>
            <class>[your_extension]/[creditmemo_total_model]</class>
          </[total_identify]>
        </totals>
      </order_creditmemo>
    </sales>
  </global>
</config>
```
2. Создать костомную total model
Модель должна наследоваться от `Mage_Sales_Model_Quote_Address_Total_Abstract` и для влияния на итоговую сумму нужно переопределить метод `collect(Mage_Sales_Model_Quote_Address $address)`

## How can the individual total models be identified for a given checkout process?

## How can the priority of total model execution be customized?

При определении модели их приоритет можно указать при помощт нод `before` и `after`.

## To which objects do total models have access in Magento, and which objects do they usually manipulate?

  * Quote
  * Order
  * Invoice
  * Creditmemo

## Which class runs total models processing?

`Mage_Sales_Model_Quote_Address_Total_Collector`

## What is the flow of total model execution?

## At which moment(s) are total models executed

При вызове метода `Mage_Sales_Model_Quote->collectTotals()` обычно, перед сохранении квоты
