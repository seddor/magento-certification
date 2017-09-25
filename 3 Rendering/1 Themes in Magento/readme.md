#Define and describe the use of themes in Magento

Темы используюся для изменения лаяутов, скинов(JS, CSS, image etc.) и блоков.

Текущая тема настраивается в админке, в system - configuration - design - themes

Здесь указывается из какой темы браь темлейты, скины, лаяуты, перевод, а также указывается родительская тема.

# Define and describe the use of design packages

В пакетах хранится несколько тем. В пределов пакета тема может наследоватся от другой темы в пакете.

# Describe the process of defining template file paths

Метод для получения пути к файлу Mage_Core_Model_Design_Package->\_fallback();

```php
<?
protected function _fallback($file, array &$params, array $fallbackScheme = array(array()))
{
    if ($this->_shouldFallback) {
        foreach ($fallbackScheme as $try) {
            $params = array_merge($params, $try);
            $filename = $this->validateFile($file, $params);
            if ($filename) {
                return $filename;
            }
        }
        $params['_package'] = self::BASE_PACKAGE;
        $params['_theme']   = self::DEFAULT_THEME;
    }
    return $this->_renderFilename($file, $params);
}
```

Файл сначала ищестся по иерахии темы, и если не будет найден, то берётся из базовой темы.
