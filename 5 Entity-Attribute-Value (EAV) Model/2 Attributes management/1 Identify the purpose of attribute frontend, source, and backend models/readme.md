# Identify the purpose of attribute frontend, source, and backend models

Все аттрибуты хранятся в таблице `eav_attribute`:

  * attribute_id — ИД
  * entity_type_id — ИД из eav_entity_type
  * attribute_code — код аттрибтута, связка entity_type_id и attribute_code уникальный
  * attribute_model — модель для работы с аттрибутом по умолчанию используется `eav/entity_attribute`
  * backend_model — модель для выполнения дествий перед сохранением, удалением, валидации, по умолчанию `Mage_Eav_Model_Entity_Attribute_Backend_Default`
  * backend_type — тип аттрибута, влияет на аблицу, в которой будут хранить значения. Доступные типа
      1. varchar
      2. int
      3. text
      4. decimal
      5. datetime
      6. static — аттрибут будет хранится в главной таблице сущности    
  * backend_table — можно использовать для задачи кастомной таблицы для значений
  * frontend_model — модуль отвечающая за получения значений на фронтенде, абстрактный класс для таких моделей `Mage_Eav_Model_Entity_Attribute_Frontend_Abstract`
  * frontend_input — тип инпута для форм
  * frontend_label — лейбал для фронтенда
  * frontend_class — класс который можно получить $attribute->getClass();
  * source_model — модель для получение значений для селектов, мультиселектов, класс должен реализовывать toOptionArray();
  * is_required — флаг обязательности
  * is_user_defined — флаг был ли аттрибут создан маджентой
  * default_value — дефолтное значение
  * is_unique — уникальности значений аттрибута
  * note
