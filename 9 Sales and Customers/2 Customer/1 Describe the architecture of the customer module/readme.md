# Describe the architecture of the customer module

Основные части связаные с кастомерами в мадженте:

  - Customer — EAV
  - Customer address — EAV
  - Customer Group

## Customer

EAV модель

  - Класс модели — Mage_Customer_Model_Customer
  - Класс ресурсной модели — Mage_Customer_Model_Resource_Customer
  - Класс коллекции — Mage_Customer_Model_Resource_Customer_Collection

## Customer Address

EAV модель

  - Класс модели — Mage_Customer_Model_Address
  - Класс ресурсной модели — Mage_Customer_Model_Resource_Address
  - Класс коллекции — Mage_Customer_Model_Resource_Address_Collection

## Customer Group

Используются для разделения кастомеров, также в группах указывается налоговый класс для участников группы.
