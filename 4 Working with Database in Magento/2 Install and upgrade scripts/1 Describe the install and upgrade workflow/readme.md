# Describe the install/upgrade workflow

## Конфигурация

Для исползьование инсталяционных скриптов для добавление таблиц моделей нужно:

  * У модуля должна быть указана версия
  * У модуля должны быть сконфигурированы модели, ресурсы
  * У модуля должна быть сконфигурирован setup resource конфиг


### Версия модуля

```xml
<modules>
    <Foo_Bar>
        <version>0.0.1</version>
    </Foo_Bar>
</modules>
```

### Конфигурация моделей

```xml
<global>
  <models>
    <foo_bar>
      <class>Foo_Bar_Model</class>
      <resourceModel>foo_bar_resource</resourceModel>
    </foo_bar>
    <foo_bar_resource>
      <class>Foo_Bar_Model_Resource</class>
      <entities>
        <baz>
          <table>baz</table>
        </baz>
      </entities>
    </foo_bar_resource>
  </models>
</global>
```


### setup resource

```xml
<global>
    <resources>
        <foo_bar_setup>
            <setup>
                <module>Foo_Bar</module>
            </setup>
        </foo_bar_setup>
    </resources>
</global>
```

Установка запускается при любом последующем обращении к мадженте. При загрузке модулей будет вызван метод
```
Mage_Core_Model_Resource_Setup::applyAllUpdates();
```


Все установки записываются в таблицу `core_resource`.

| **code** | **version** | **data_version** |
| :- | :- |:- |
| foo_bar_setup | 0.0.1 | 0.0.1 |


Инсталяционные скрипты кладутся в директорию sql/foo_bar_setup:
```
install-0.0.1.php
upgrade-0.0.1-0.0.2.php
```
