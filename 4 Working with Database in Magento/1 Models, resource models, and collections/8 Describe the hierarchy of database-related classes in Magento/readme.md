# Describe the hierarchy of database-related classes in Magento

  * Magento_Db_Adapter_Pdo_Mysql — адаптер Mysql для работы с БД
  * Mage_Core_Model_Resource — класс для получения конектов и адаптеров
  * Varien_Db_Select — наследует `Zend_Db_Select` используется для построения запросов
  * Mage_Core_Model_Resource_Setup — класс для data и SQL(install/update) скриптов.
  * Mage_Core_Model_Resource_Db_Abstract — абстрактный класс ресурсной модели, реализует свзяь модели с БД, посредством адаптеров.
  * Varien_Data_Collection_Db  — Базовый класс коллеций
