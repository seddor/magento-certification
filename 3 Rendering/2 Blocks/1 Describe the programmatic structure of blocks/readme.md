# Describe the programmatic structure of blocks

Блок это класс, который наследуется от абстрактного Mage_Core_Block_Abstract и меет метод. Класс Mage_Core_Block_Abstract имеет финальный метод toHtml(), который отвечает за вывод контента блока.

```php
<?
final public function toHtml()
    {
        Mage::dispatchEvent('core_block_abstract_to_html_before', array('block' => $this));
        if (Mage::getStoreConfig('advanced/modules_disable_output/' . $this->getModuleName())) {
            return '';
        }
        $html = $this->_loadCache();
        if ($html === false) {
            $translate = Mage::getSingleton('core/translate');
            /** @var $translate Mage_Core_Model_Translate */
            if ($this->hasData('translate_inline')) {
                $translate->setTranslateInline($this->getData('translate_inline'));
            }

            $this->_beforeToHtml();
            $html = $this->_toHtml();
            $this->_saveCache($html);

            if ($this->hasData('translate_inline')) {
                $translate->setTranslateInline(true);
            }
        }
        $html = $this->_afterToHtml($html);

        /**
         * Check framing options
         */
        if ($this->_frameOpenTag) {
            $html = '<' . $this->_frameOpenTag . '>' . $html . '<' . $this->_frameCloseTag . '>';
        }

        /**
         * Use single transport object instance for all blocks
         */
        if (self::$_transportObject === null) {
            self::$_transportObject = new Varien_Object;
        }
        self::$_transportObject->setHtml($html);
        Mage::dispatchEvent('core_block_abstract_to_html_after',
            array('block' => $this, 'transport' => self::$_transportObject));
        $html = self::$_transportObject->getHtml();

        return $html;
    }
```

## What are blocks used for in Magento?

Блоки используются для вывода.

## What is the parent block for all Magento blocks?

Mage_Core_Block_Abstract, наследуются от Varien_Object, так что блоки поддерживают маджентовские магические методы

## Which class does each block that uses a template extend?

Mage_Core_Block_Template

## In which way does a template block store information about its template file? Does it store an absolute or a relative path to the template?

Путь к шаблону хранится в протектед поле $\_template класса Mage_Core_Block_Template. Хранится путь относительно директории template темы

## What is the role of the Mage_Core_Block_Abstract class?

Является базовым классом, добавляет следующие возможно

* Добавляет метод для вызова хелперов
* getChildHtml() и getChild() для работы с дочерними блоками
* финальный метод toHtml()
* _beforeToHtml() вызываемый перед рендерингом блока
* _toHtml() определяет вывод блока
* кеширование
