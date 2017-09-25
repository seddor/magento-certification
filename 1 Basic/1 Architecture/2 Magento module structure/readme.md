# Describe typical Magento module structure

Минимальная страктура модуля состоит из
1. xml файла в ***app/etc/modules*** с определением модул.
Напрмер модуль `Mage_Widget`. xml файл с его определненим лежить в ***app/etc/modules/Mage_Widget.xml*** и содержит следующее
```xml
<config>
    <modules>
        <Mage_Widget>
            <active>true</active>
            <codePool>core</codePool>
            <depends>  
              <!--
               зависимость модуля от другого модуля,
               используется для определения порядка
               загрузки модулей, и разрешения возможных
               конфликтов среди модулей
              -->
                <Mage_Cms />
            </depends>
        </Mage_Widget>
    </modules>
</config>
```
2. Основного кода модуля, содержащегося в ***app/code/<codePool>/<Company>/<Module>***
Например модуль `Foo_Bar` определёный в кодпуле `local` будет в ***app/code/local/Foo/Bar***. Внтури содержатся структурные директории модуля, их набор может различатся в зависимости от возможностей модуля. Основные структурные директории:
 * Block
 * controllers
 * Model
 * Helper
 * etc -- xml-файлы с конфигами
 * sql -- интсталяционые скрипты для таблиц, схем в БД
 * data -- инсталяционые скрипты для данных в БД
3. Также модуль может содержать дополнительнын части, такие как
  * Переводы в `app/etc/locale`
  * Лаяуты и темплейты в ***app/design/<area>/<package>/<theme>/(template|layouts)***
  area == frontend или adminhtml
