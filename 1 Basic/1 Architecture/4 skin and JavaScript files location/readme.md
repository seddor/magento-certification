# Describe Magento skin and JavaScript files location

## Skin

`skin` эта директория в корне проекта, в ней хранятся различные вещи предназначеные для фронт-энд разработки, такие как стили, js, картинки и прочее. Структура подиректорий разбита на `design area`, `package` и `theme` в таком же виде как в директории `design`. Т.е. каждой теме соответвует своя поддиректория в skin по такому пути ***skin/{{area}}/{{package}}/{{theme}}***

##JavaScript

JS подразделяется на два вида

* skin JS хранится в skin директории темы в ***skin/{{area}}/{{package}}/{{theme}}/js*** используется соотвено для нужд темы
* JavaScript Libraries/Frameworks в ***js/{{libary}}***
