#Describe Magento templates and layout files location

##Design area

В Мадженте присутвют три(фактически используются две) design area, все они находятся в `app/design`

* adminhtml -- админка
* frontend -- фронт
* install -- темлейты и лаяуты для инсталятора мадженты

## Templates

Шаблоны хранятся по пути ***app/design/{{area}}/{{package}}/{{theme}}/template***.

## Layouts

Лаяуты хранятся по пути ***app/design/{{area}}/{{package}}/{{theme}}/lauout***.

Для дочерних тем также присутвует файл ***app/design/{{area}}/{{package}}/{{theme}}/etc/theme.xml***

В котором указывается тема-родитель, например тема наследуемая от темы rwd содержит следующий `theme.xml`:
```xml
<?xml version="1.0"?>
<theme>
    <parent>rwd/default</parent>
</theme>
```
