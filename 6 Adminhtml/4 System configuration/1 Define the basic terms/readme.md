# Define the basic terms, elements, and structure of system configuration XML

## How can elements in system configuration be rendered with a custom template?

Кастомный рендор можно сделать с помощью `<frontend_model>`

```XML
<flat_catalog_category translate="label">
  <label>Use Flat Catalog Category</label>
  <frontend_type>select</frontend_type>
  <frontend_model>adminhtml/system_config_form_field_select_flatcatalog</frontend_model>
  <backend_model>catalog/system_config_backend_catalog_category_flat</backend_model>
  <source_model>adminhtml/system_config_source_yesno</source_model>
  <sort_order>100</sort_order>
  <show_in_default>1</show_in_default>
  <show_in_website>0</show_in_website>
  <show_in_store>0</show_in_store>
</flat_catalog_category>
```

Фронтэнд модель можно унаследовать от `Mage_Adminhtml_Block_System_Config_Form_Field`, кастомный вывод можно задать в метода `getElementHtml`

Если нужен кастомный вид группы, то можно создать фронт жнд модель и унаследовать класс `Mage_Adminhtml_Block_System_Config_Form_Fieldset`

## How does the structure of system.xml relate to the rendered elements in the System Configuration view?

```xml
<config>
  <tabs> <!-- табы -->
    <example translate="label" module="foo_bar">
      <label>Example Tab</label>
      <sort_order>200</sort_order>
    </example>
  </tabs>
  <sections>
    <foo translate="label" module="foo_bar"> <!-- при создании кастомной секции ей нужно прописать acl -->
      <label>Foo Bar Config</label>
      <tab>example</tab>
      <frontend_type>text</frontend_type>
      <sort_order>1500</sort_order>
      <show_in_default>1</show_in_default> <!-- эти три ноды задают область применения и видимости конфига,  -->
      <show_in_website>0</show_in_website> <!-- в данном случае это глобальный когфиг -->
      <show_in_store>0</show_in_store>      <!-- так же может быть соотвено для сайта и стора -->
      <groups>
        <bar translate="label">
          <label>Bar</label>
          <frontend_type>text</frontend_type>
          <sort_order>1</sort_order>
          <show_in_default>1</show_in_default>
          <show_in_website>0</show_in_website>
          <show_in_store>0</show_in_store>
          <fields>
            <baz translate="label">
              <label>Baz</label>
              <frontend_type>select</frontend_type> <!-- -->
              <source_model>adminhtml/system_config_source_yesno</source_model>
              <sort_order>1</sort_order> <!-- задаёт положение относительно других элементов -->
              <show_in_default>1</show_in_default>
              <show_in_website>0</show_in_website>
              <show_in_store>0</show_in_store>
              <frontend_class>baz</frontend_class> <!-- добавляет css класс элементу -->
              <comment>this is comment</comment> <!-- комментарий под полем -->
              <tooltip>HINT TEXT</tooltip> <!-- высплывающая подсказка -->
            </baz>
          </fields>
        </bar>
      </groups>
    </foo>
  </sections>
</config>
```

## How can the CSS class of system configuration elements be changed?

С помощью ноды `<frontend_class>`

## What is the syntax for specifying the options in dropdowns and multiselects?

нужно указать элис к source model в ноде `<source_model>` класс должен имплементить метод toOptionArray()

## Which classes are used to parse and render system configuration XML?

  * `Mage_Adminhtml_Model_Config`
  * `Mage_Adminhtml_Block_System_Config_Tabs`
  * `Mage_Adminhtml_Block_System_Config_Form`
  * `Mage_Adminhtml_Block_System_Config_Form_Fieldset`
  * `Mage_Adminhtml_Block_System_Config_Form_Field`

## What is the syntax to specify a custom renderer for a field in system configuration?

указать кастомный `<frontend_class>` и `<frontend_type>`, создать кастомый блок для рендеринга указаный во `<frontend_model>`

## How does Magento store data for system configuration?

Данные хранятся в `core_congig_data`

## What is the difference between Mage::getStoreConfig(...) and Mage::getConfig()->getNode(...)?

  * getStoreConfig — возвращает значения, обычно в виде строки
  * getNode — возвращает экземляр класса Mage_Core_Model_Config_Element

```php
<?
public static function getStoreConfig($path, $store = null)
{
    return self::app()->getStore($store)->getConfig($path);
}

public function getNode($path=null, $scope='', $scopeCode=null)
{
    if ($scope !== '') {
        if (('store' === $scope) || ('website' === $scope)) {
            $scope .= 's';
        }
        if (('default' !== $scope) && is_int($scopeCode)) {
            if ('stores' == $scope) {
                $scopeCode = Mage::app()->getStore($scopeCode)->getCode();
            } elseif ('websites' == $scope) {
                $scopeCode = Mage::app()->getWebsite($scopeCode)->getCode();
            } else {
                Mage::throwException(Mage::helper('core')->__('Unknown scope "%s".', $scope));
            }
        }
        $path = $scope . ($scopeCode ? '/' . $scopeCode : '' ) . (empty($path) ? '' : '/' . $path);
    }

    /**
     * Check path cache loading
     */
    if ($this->_useCache && ($path !== null)) {
        $path   = explode('/', $path);
        $section= $path[0];
        if (isset($this->_cacheSections[$section])) {
            $res = $this->getSectionNode($path);
            if ($res !== false) {
                return $res;
            }
        }
    }
    return  parent::getNode($path);
}
```


http://freaksidea.com/php_and_somethings/show-60-magento-konfighuratsiia-ot-a-do-ia-system-xml

http://alanstorm.com/magento_system_configuration_in_depth_tutorial/
