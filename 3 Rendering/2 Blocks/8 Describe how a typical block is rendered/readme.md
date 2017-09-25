# Describe how a typical block is rendered

## Which class performs rendering of the template?

Класс Mage_Core_Block_Template, метод renderView()
```php
<?
public function renderView()
{
    $this->setScriptPath(Mage::getBaseDir('design'));
    $html = $this->fetchView($this->getTemplateFile());
    return $html;
}
```

## Which classes are responsible for figuring out the absolute path for including the template file?

Mage_Core_Block_Template->getTemplateFile()
```php
<?
public function getTemplateFile()
{
    $params = array('_relative'=>true);
    $area = $this->getArea();
    if ($area) {
        $params['_area'] = $area;
    }
    $templateName = Mage::getDesign()->getTemplateFilename($this->getTemplate(), $params);
    return $templateName;
}
```

## In which method are templates rendered?

Mage_Core_Block_Abstract->toHtml() в нём вызывается \_toHtml() дочернего блока(Mage_Core_Block_Template)

## How can output buffering be enabled/disabled when templates are rendered?

Mage_Core_Model_Layout::setDirectOuput(true)
