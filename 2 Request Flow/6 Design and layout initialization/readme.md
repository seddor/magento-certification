# Identify the steps in the request flow in which:
  * Design data is populated
  * Layout configuration files are parsed
  * Layout is compiled
  * Output is rendered


Генерация вывода
1. Контроллкер получается из роута
2. Контроллер наследует Mage_Core_Controller_Varien_Action
3. В экшене контролера вызывается $this->loadLayout(), который генерирует хендлы, лаяуты и блоки
4. Метод $this->renderLayout() выводит в браузер результат

## 1 Mage_Core_Controller_Varien_Action->loadLayout()
```php
<?
public function loadLayout($handles = null, $generateBlocks = true, $generateXml = true)
{
    // if handles were specified in arguments load them first
    if (false!==$handles && ''!==$handles) {
        $this->getLayout()->getUpdate()->addHandle($handles ? $handles : 'default');
    }

    // add default layout handles for this action
    $this->addActionLayoutHandles();

    $this->loadLayoutUpdates();

    if (!$generateXml) {
        return $this;
    }
    $this->generateLayoutXml();

    if (!$generateBlocks) {
        return $this;
    }
    $this->generateLayoutBlocks();
    $this->_isLayoutLoaded = true;

    return $this;
}
```

1) хендлы которые добавляются
  * default — общий хендл, добавляется для всех
  * хендл экшена, например: *catalog_category_view*
  * STOTE_{{store_code}}
  * THEME_{{area}}_{{package}}_{{theme}}
  * customer_logged_in/customer_logged_out — используются редко
  Также в экшене контролера можно руками добавить кастомные хендлы, например для всех категорий и продуктов добавляются хендлы вида:
  * CATEGORY_{{category_id}} или PRODUCT_{{product_id}} соответственно

## 2 Mage_Core_Model_Layout->generateXml()

Лаяуты для модулей добавляются через config.xml
```xml
<frontend>
  <layout>
      <updates>
          <catalog>
              <file>catalog.xml</file>
          </catalog>
      </updates>
  </layout>
</frontend>
```
Все файлы лаяутов + local.xml мерджатся в один xml в метода Mage_Core_Model_Layout_Update->getFileLayoutUpdatesXml();

## 3 Mage_Core_Model_Layout->generateBlocks()

Метод парсит дерево лаяута и создаёт соотвено блоки или производит с ними иные операции
```
switch ($node->getName()) {
  case 'block':
    $this->_generateBlock($node, $parent);
    $this->generateBlocks($node);
    break;
  case 'reference':
    $this->generateBlocks($node);
    break;
  case 'action':
    $this->_generateAction($node, $parent);
    break;
}
```

## 4 Mage_Core_Controller_Varien_Action->renderLayout();

Здесь для вывода результатов в браузер вызывается метод `Mage_Core_Model_Layout->getOutput()`
```php
<?
public function getOutput()
{
    $out = '';
    if (!empty($this->_output)) {
        foreach ($this->_output as $callback) {
            $out .= $this->getBlock($callback[0])->{$callback[1]}();
        }
    }

    return $out;
}
```
