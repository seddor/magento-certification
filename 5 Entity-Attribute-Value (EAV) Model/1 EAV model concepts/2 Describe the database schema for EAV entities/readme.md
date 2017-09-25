# Describe the database schema for EAV entities

Структура БД для EAV на примере customer

## Entity

Таблица `customer_entity` здесть имеются поля

  * entity_id
  * entity_type_id
  * attribute_set_id
  * created_at
  * updated_ar
  * а также некоторые специфичные для даной сущности поля, которые часто используются, поэтому лучше их хранить сразу с энтити, а не в отдельной таблице со значениями(например, email).

## Attribute

Все аттрибуты хранятся в таблице `eav_attribute`, она содержит такие поля

  * attribute_id
  * entity_type_id
  * attribute_code
  * attribute_model
  * backend_model
  * backend_type
  * backend_table
  * frontend_model
  * frontend_input
  * frontend_label
  * frontend_class
  * source_model
  * is_required
  * is_user_defined
  * default_value
  * is_unique
  * note

Ссылки на аттрибуты связаные с энити кастомера хранятся в таблице `customer_eav_attribute`

## Value

Значения хранятся в таблицах с именем вида `customer_entity_{{value_type}}`. Для кастомреа это

  * customer_entity_datetime
  * customer_entity_decimal
  * customer_entity_int
  * customer_entity_text
  * customer_entity_varchar

Стуктура этих таблиц схожа и имеет следующий вид:

  * value_id
  * entity_type_id
  * attribute_id
  * entity_id
  * value
