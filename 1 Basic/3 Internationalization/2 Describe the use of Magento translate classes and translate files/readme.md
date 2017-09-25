# Describe the use of Magento translate classes and translate files

[Статья про локализацию](https://belvg.com/blog/magento-certified-developer-exam-internationalization.html)

Существует 3 типа переводов(в порядке загрузки):

1. Переводы из модуля
2. Переводы из темы
3. Переводы из БД

Приоритет переводов обратный порядку загрузки(т.е. перевод из БД затрёт остальные переводы)

## 1 Переводы из модуля

* **Конфигурация.** В config.xml модуля, в нужной area добавить
 ```xml
<translate>
  <modules>
      <Module_Name>
          <files>
              <default>Module_Name.csv</default>
          </files>
      </Module_Name>
  </modules>
</translate>
```
* **Файл с переводами.** Хранится в *app/locale/{{local_code}}/Module_Name.csv*.
Содержит что-то вроде:
```
"product","продукт"
```

## 2 Переводы из темы

* **Файл с переводами.** Хранится в *app/design/{{package}}/{{theme}}/locale/{{local_code}}/translate.csv*.
Содержит что-то вроде:
```
"product","продукт"
```

## 3 Переводы из БД

Переводы которые быле добавлены с помощью построчного превода(System -> Configuration -> Developer -> Translate Inline.)


Переводы хранятся в таблице — `core_translate`.


## Как работает перевод

Для того, чтобы перевести что-то в мадженте нужно использовать функцию `__()`

```php
<?
public function __()
{
    $args = func_get_args();
    $expr = new Mage_Core_Model_Translate_Expr(array_shift($args), $this->getModuleName());
    array_unshift($args, $expr);
    return $this->_getApp()->getTranslator()->translate($args);
}
```

В неё передаётся:

* Текст перевода
* Дополнтельно можно передать параметры для подстановки, например
```
$this->__('%s product of %s products', 1, 20);
```
Соответствено получится перевод фразы `1 product of 20 products`

Перевод берётся в следующем порядке

* проверка кеша(если не активирован $forceReload)
* загрузка из модуля(при загрзке перевода из модулей все файлы переводов мерджатся в один)
* загрузка из темы
* загрузка из ДБ
* сохранение в кеш(если не активирован $forceReload)
