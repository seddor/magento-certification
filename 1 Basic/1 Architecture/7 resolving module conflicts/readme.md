# Describe methods for resolving module conflicts.

Есть три типа конфликтов

* Конфликт конфигурации
* Конфликт реврайта
* Конфликт темы

## Конфликт конфигурации

Может возникнуть когда два модуля зависимы от одного и того же третьего, что иногда может привести к конфликтом во время загрукзи классов или файлов

### Пример

***app/etc/modules/Module_Two.xml***
```xml
<config>
  <modules>
    <Module_One>
      <active>true</active>
      <codePool>community</codePool>
      <depends><Mage_Core/></depends>
    </Module_One>
  </modules>
</config>
```
***app/etc/modules/Module_Two.xml***
```xml
<config>
  <modules>
    <Module_Two>
      <active>true</active>
      <codePool>community</codePool>
      <depends><Mage_Core/></depends>
    </Module_Two>
  </modules>
</config>
```
### Решение

Решением является помтроение иерахии зависимостей для одного из модулей, т.е. один модуль из конфликтующих моделей должен унаследоватя от другого:
***app/etc/modules/Module_Two.xml***
```xml
<config>
  <modules>
    <Module_Two>
      <active>true</active>
      <codePool>community</codePool>
      <depends><Mage_One/></depends>
    </Module_Two>
  </modules>
</config>
```

## Конфликт реврайтов

Конфликт возникает когда два разных класса и разных модулей реврайтят один класс

## Пример
***Module_One/etc/config.xml***

```xml
<global>
  <modules>
    <sales>
      <rewrite>
        <order>Module_One_Model_Rewrite_Sales_Order</order>
      </rewrite>
    </sales>
  </modules>
```
***Module_Two/etc/config.xml***
```xml
<global>
  <modules>
    <sales>
      <rewrite>
        <order>Module_Two_Model_Rewrite_Sales_Order</order>
      </rewrite>
    </sales>
  </modules>
```
## Решение

В данном случае один класс, который реврайтит должен унаследоватся от другого, а в конфигурации следует оставить только последнего в иерархии класса. Т.е.:
1. Это следует удалить:
***Module_One/etc/config.xml***
```xml
<global>
  <modules>
    <sales>
      <rewrite>
        <order>Module_One_Model_Rewrite_Sales_Order</order>
      </rewrite>
    </sales>
  </modules>
```
2. В ревратящем классе из второго модуля следует изменить:
***app/code/community/Module_Two/Model/Rewrite/Sales/Order.php***
```
class Module_Two_Model_Rewrite_Sales_Order extends Module_One_Model_Rewrite_Sales_Order {
  ...
}
```

## Дополительно

[Describing Methods for Resolving Module Conflicts](https://belvg.com/blog/get-ready-for-magento-certified-developer-exam-describing-methods-for-resolving-module-conflicts.html)
