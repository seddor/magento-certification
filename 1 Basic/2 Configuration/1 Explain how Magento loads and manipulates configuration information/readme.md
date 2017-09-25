# Explain how Magento loads and manipulates configuration information

Конфигурация мадженты может хранится в

1. xml файлах
2. в БД в таблице `core_config_data`

## 1 xml файлы

Конфигурация в xml файлах хранится в ***app/etc***(общая, базовая конфигурация) и в папках ***etc*** модулей.

В ***app/etc***:

* config.xml - содержит дефолтные настройки, обычно не изменяется
* local.xml - файл с базовыми настройками, обычно настройки в этом файле уникальны для конкретного инстанца мадженты, в нём содержатся локальные настройки(например настройки конекта к БД, настройки кеширования и т.п.)

В директории ***etc*** содежатся настройки добавляемые в модулях.

## Загрузка конфигурации

1. Маджента из Mage::run вызывает метод `run()` у `Mage_Core_Model_App`
```php
<?
public function run($params)
    {
        $options = isset($params['options']) ? $params['options'] : array();
        //инцилизация базовых конфигов
        $this->baseInit($options);
        Mage::register('application_params', $params);

        if ($this->_cache->processRequest()) {
            $this->getResponse()->sendResponse();
        } else {
            //если запрос не закеширован происходит иницилизация конфигов модулей
            $this->_initModules();

            ...
        }
        return $this;
    }
```
2. Иницилизация базовых конфигов
класс Mage_Core_Model_App
```php
<?
public function baseInit($options)
{
    $this->_initEnvironment();

    //Mage::getConfig() возвращает инстанс Mage_Core_Model_Config
    $this->_config = Mage::getConfig();
    $this->_config->setOptions($options);

    //здесь иницилизируется базовый конфиг см. 2.1
    $this->_initBaseConfig();
    $cacheInitOptions = is_array($options) && array_key_exists('cache', $options) ? $options['cache'] : array();
    $this->_initCache($cacheInitOptions);

    return $this;
}
```

  2.1 в `_initBaseConfig` происходит вызов метода `loadBase` класса `Mage_Core_Model_Config`
```php
<?
public function loadBase()
    {
        $etcDir = $this->getOptions()->getEtcDir();
        $files = glob($etcDir.DS.'*.xml');
        $this->loadFile(current($files));
        while ($file = next($files)) {
            $merge = clone $this->_prototype;
            $merge->loadFile($file);
            $this->extend($merge);
        }
        if (in_array($etcDir.DS.'local.xml', $files)) {
            $this->_isLocalConfigLoaded = true;
        }
        return $this;
    }
```

3. Иницилизация конфигов модулей, метод `Mage_Core_Model_App->_initModules()`, в нём происходит следующее:
 3.1 Если кеш пустой то проводится иницилизация модулей
 3.2 вызов `Mage_Core_Model_Config->loadModules()`
 ```php
 <?
 public function loadModules()
     {
         Varien_Profiler::start('config/load-modules');
         //загрузка информации о всех модулях из app/etc/modules
         $this->_loadDeclaredModules();

         $resourceConfig = sprintf('config.%s.xml', $this->_getResourceConnectionModel('core'));
         //загрузка и мердж всех модульных файлов конфигурации
         $this->loadModulesConfiguration(array('config.xml',$resourceConfig), $this);

         /**
          * Prevent local.xml directives overwriting
          * исключение директив local.xml  
          */
         $mergeConfig = clone $this->_prototype;
         $this->_isLocalConfigLoaded = $mergeConfig->loadFile($this->getOptions()->getEtcDir().DS.'local.xml');
         if ($this->_isLocalConfigLoaded) {
             $this->extend($mergeConfig);
         }

         $this->applyExtends();
         Varien_Profiler::stop('config/load-modules');
         return $this;
     }
 ```
 3.3 вызов `Mage_Core_Model_Config->loadDb()` в котором происходит загрузка конфигов модулей из БД
 3.4 вызов `Mage_Core_Model_Config->saveCache()` для сохранения кеша
