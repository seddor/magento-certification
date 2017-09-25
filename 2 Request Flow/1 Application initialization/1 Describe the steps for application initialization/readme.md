#Describe the steps for application initialization

Шаги иницилиализации:
1. index.php
2. Mage::run($mageRunCode, $mageRunType);
3. Mage_Core_Model_App->run($params);
4. Mage_Core_Model_App->baseInit($options)
5. Mage_Core_Model_App->_initModules();
6. Mage_Core_Model_App->_initCurrentStore($scopeCode, $scopeType);
7. Dispatches the Front Controller

## 1 index.php

1. Проверка версии PHP.
2. Проверка существование Compiler и его инклуд
3. Проверка Mage.php
4. Проверка maintenance.flag(если существует то возвращает /errors/503.php)
5. require /app/bootstrap.php и Mage.php
6. Проверка параметра девелоп мода, и переключение в него при необходимости
7. Полусение параметров `MAGE_RUN_CODE` и `MAGE_RUN_TYPE`
8. Mage::run($mageRunCode, $mageRunType);

### MAGE_RUN_CODE

Код стора/вебсайта в окружении которого нужно запустить мадженту

### MAGE_RUN_TYPE

По умолчанию `store`

Есть такие типы:
  * store — текущий стор получается по переданому коду стора
  * group — текущий стор получается по коду группы
  * website — текущий стор получается по коду вебсайта

## 2 Mage::run($mageRunCode, $mageRunType);

1. Проверка опций
2. Инстанцирование объекта Mage_Core_Model_App
3. Проставление флага установлености
4. Установка опций config_model(по.умолчанию для конфигов используется Mage_Core_Model_Config)
5. Вызов Mage_Core_Model_App->run($params);

## 3 Mage_Core_Model_App->run($params);

1. Загрузка конфигурации (local.xml и config.xml) в baseInit();
2. Добавление в реест параметров запуска
3. Проверка кеша, и отправка ответа, если реквест уже закеширован
4. Иницилиализация модулей, запуск install/update script
5. Загрузка Areas(frontend, adminhtml, install)
6. Иницилиализация текущего стора
7. Запуск скриптов обволения данных
8. вызов $this->getFrontController()->dispatch();

## 4 Mage_Core_Model_App->baseInit($options)

1. Иницилизация окружения(setErrorHandler, установка дефолтной таймзоны)
2. Назачение модели конфигурации
3. Изменени опций конфигурации из $options
4. загразка базового конфига(local.xml и config.xml из app/etc)
5. иницилизация кеша

## 5 Mage_Core_Model_App->_initModules();

1. Проверка кеша
2. Загрузка модулей и их конфигов(из файлов)
3. Запуск инсталяционных скриптов
4. Загрузка конфигов модулей из БД
5. Сохранение кеша

## 6 Mage_Core_Model_App->_initCurrentStore($scopeCode, $scopeType);

Получение объекта стора по ранее переданым параметрам в Mage::run();
