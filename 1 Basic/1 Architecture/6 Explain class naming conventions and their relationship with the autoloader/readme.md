# Explain class naming conventions and their relationship with the autoloader

При каждом запросе к мадежнте в её самом базовом файле `Mage.php`.
Происходит слеудующее:
```php
<?
...
Mage::register('original_include_path', get_include_path());

if (defined('COMPILER_INCLUDE_PATH')) {
    $appPath = COMPILER_INCLUDE_PATH;
    set_include_path($appPath . PS . Mage::registry('original_include_path'));
    include_once COMPILER_INCLUDE_PATH . DS . "Mage_Core_functions.php";
    include_once COMPILER_INCLUDE_PATH . DS . "Varien_Autoload.php";
} else {
    /**
     * Set include path
     */
    $paths = array();
    $paths[] = BP . DS . 'app' . DS . 'code' . DS . 'local';
    $paths[] = BP . DS . 'app' . DS . 'code' . DS . 'community';
    $paths[] = BP . DS . 'app' . DS . 'code' . DS . 'core';
    $paths[] = BP . DS . 'lib';

    $appPath = implode(PS, $paths);
    set_include_path($appPath . PS . Mage::registry('original_include_path'));
    include_once "Mage/Core/functions.php";
    include_once "Varien/Autoload.php";
}

Varien_Autoload::register();
```
Маджента инклудит пути для кодпулов и библиотек, а также иницилирует класс автолоадера `Varien_Autoload`
в котором определён метод `autoload`:
```php
<?
public function autoload($class)
{
    if ($this->_collectClasses) {
        $this->_arrLoadedClasses[self::$_scope][] = $class;
    }
    if ($this->_isIncludePathDefined) {
        $classFile =  COMPILER_INCLUDE_PATH . DIRECTORY_SEPARATOR . $class;
    } else {
        $classFile = str_replace(' ', DIRECTORY_SEPARATOR, ucwords(str_replace('_', ' ', $class)));
    }
    $classFile.= '.php';
    //echo $classFile;die();
    return include $classFile;
}
```
Он принимет один параметр `$class` и затем
1. Заеменяет `_` на ` `(пробел).
2. Преобразает в верхний регсит каждое слово в строке
3. Заменяет пробелы на DIRECTORY_SEPARATOR
4. Инклудит полученый путь добавляя `.php` в конец
