# Describe system configuration scopes

## How do different scopes (global, website, store) work in Magento system configuration?

скопы задаются с помощью этих нод:

```
<show_in_default>1</show_in_default>
<show_in_website>1</show_in_website>
<show_in_store>1</show_in_store>
```

  * default — глобально
  * website — для вебсайта
  * store — для стора


## How does Magento store information about option values and their scopes?

Дефолтные значения для конфигов можно задавть в `config.xml` в ноде `default`

```xml
<default>
  <module_name>
    <group_name>
      <field_name1>field_value1</field_name1>
      <field_name2>field_value2</field_name2>
    </group_name>
  </module_name>
</default>
```

Сами значения хранятся в таблице `core_config_data`, она имеет следующие поля:

  * config_id
  * scope — default/stores/websites
  * scope_id — ид сайта или стора
  * path — путь к конфигу (sections/groups/fields)
  * value — значение конфига
