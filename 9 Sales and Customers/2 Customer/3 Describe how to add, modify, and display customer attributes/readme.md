# Describe how to add, modify, and display customer attributes

Атрибуты можно добавить\изменить так:

  - Через инсталеры
  - Через добавление, редактирование аттрибутов в админке (EE)

Таблица `customer_form_attribute`, содержит соотвие различных кодов форм и ИД аттрибутов кастомеров, которые доступны на этой форме, так-же эта информация используется для копирования информации при создании квоты.


## What is the structure of tables in which customer information is stored?

Кастомеры и их адреса это EAV, модели, информация поделена между таблицами

### Customer

  - customer_entity
  - customer_entity_datetime
  - customer_entity_decimal
  - customer_entity_int
  - customer_entity_text
  - customer_entity_varchar

Поля в таблице `customer_entity`

  - entity_id
  - customer_id
  - attribute_set_id
  - website_id
  - email
  - group
  - increment_id
  - store_id
  - created_at
  - updated_at
  - is_active
  - disable_auto_group_change

### Customer address

  - customer_address_entity
  - customer_address_entity_datetime
  - customer_address_entity_decimal
  - customer_address_entity_int
  - customer_address_entity_text
  - customer_address_entity_varchar

Поля в таблице `customer_address_entity`

  - entity_id
  - entity_type_id
  - attribute_set_id
  - increment_id
  - parent_id (customer_entity id)
  - created_at
  - updated_at
  - is_active


## What is the customer resource model?

Mage_Customer_Model_Resource_Customer

## How is customer information validated?

Класс `Mage_Customer_Model_Customer` содержит метод `validate()` для валидации

```php
<?
public function validate()
    {
        $errors = array();
        if (!Zend_Validate::is( trim($this->getFirstname()) , 'NotEmpty')) {
            $errors[] = Mage::helper('customer')->__('The first name cannot be empty.');
        }

        if (!Zend_Validate::is( trim($this->getLastname()) , 'NotEmpty')) {
            $errors[] = Mage::helper('customer')->__('The last name cannot be empty.');
        }

        if (!Zend_Validate::is($this->getEmail(), 'EmailAddress')) {
            $errors[] = Mage::helper('customer')->__('Invalid email address "%s".', $this->getEmail());
        }

        $password = $this->getPassword();
        if (!$this->getId() && !Zend_Validate::is($password , 'NotEmpty')) {
            $errors[] = Mage::helper('customer')->__('The password cannot be empty.');
        }
        if (strlen($password) && !Zend_Validate::is($password, 'StringLength', array(self::MINIMUM_PASSWORD_LENGTH))) {
            $errors[] = Mage::helper('customer')
                ->__('The minimum password length is %s', self::MINIMUM_PASSWORD_LENGTH);
        }
        $confirmation = $this->getPasswordConfirmation();
        if ($password != $confirmation) {
            $errors[] = Mage::helper('customer')->__('Please make sure your passwords match.');
        }

        $entityType = Mage::getSingleton('eav/config')->getEntityType('customer');
        $attribute = Mage::getModel('customer/attribute')->loadByCode($entityType, 'dob');
        if ($attribute->getIsRequired() && '' == trim($this->getDob())) {
            $errors[] = Mage::helper('customer')->__('The Date of Birth is required.');
        }
        $attribute = Mage::getModel('customer/attribute')->loadByCode($entityType, 'taxvat');
        if ($attribute->getIsRequired() && '' == trim($this->getTaxvat())) {
            $errors[] = Mage::helper('customer')->__('The TAX/VAT number is required.');
        }
        $attribute = Mage::getModel('customer/attribute')->loadByCode($entityType, 'gender');
        if ($attribute->getIsRequired() && '' == trim($this->getGender())) {
            $errors[] = Mage::helper('customer')->__('Gender is required.');
        }

        if (empty($errors)) {
            return true;
        }
        return $errors;
    }
```

И также метод для валидации пароля при его восстановлении после сброса

```php
<?
public function validateResetPassword()
{
    $errors   = array();
    $password = $this->getPassword();
    if (!Zend_Validate::is($password, 'NotEmpty')) {
        $errors[] = Mage::helper('customer')->__('The password cannot be empty.');
    }
    if (!Zend_Validate::is($password, 'StringLength', array(self::MINIMUM_PASSWORD_LENGTH))) {
        $errors[] = Mage::helper('customer')
            ->__('The minimum password length is %s', self::MINIMUM_PASSWORD_LENGTH);
    }
    $confirmation = $this->getPasswordConfirmation();
    if ($password != $confirmation) {
        $errors[] = Mage::helper('customer')->__('Please make sure your passwords match.');
    }

    if (empty($errors)) {
        return true;
    }
    return $errors;
}
```

## How can customer-related email templates be manipulated?

Можно создавать новые шаблоны и назначать их в системной конфигурации

## What is the difference between shipping and billing addresses for a customer?

Нет разницы, для адресов используется одна и та же сущность(customer address). Разница есть только в использовании этих адресов

## How does the number of shipping and billing address entities affect the frontend interface for customers?

Никак

## How does customer information affect prices and taxes?

Налоги подсчитываются исходя из customer group и адресса доставки/высталвления счёта
Catalog rule и Cart rule могут мыть назначены для конкретных Customer Group

## How can attributes be added to a customer address? How are custom address attributes you converted to an order address?

Атрибуты можно добавить через инсталлеры, для того, чтобы атрибуты копировались при заказе нужно их указать в config.xml в `global/fieldsets/sales_convert_order_address`

## Can a customer be added to two customer groups at the same time?

Нет
