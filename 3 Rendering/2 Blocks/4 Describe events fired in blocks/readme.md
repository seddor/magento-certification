# Describe events fired in blocks

## How can block output be caught using an observer?

Перед самым выводом блоке генирирует событие `core_block_abstract_to_html_after`.
```
Mage::dispatchEvent(
  'core_block_abstract_to_html_after',
  [
    'block'     => $this,
    'transport' => self::$_transportObject
  ]
);
$html = self::$_transportObject->getHtml();
```

## What events do Mage_Core_Block_Abstract and Mage_Core_Block_Template fire?

### Before/After Layout Object is set

```
  Mage::dispatchEvent('core_block_abstract_prepare_layout_before', array('block' => $this));
  Mage::dispatchEvent('core_block_abstract_prepare_layout_after', array('block' => $this));
```

### Before Block is Rendered

```
  Mage::dispatchEvent('core_block_abstract_to_html_before', array('block' => $this));
```

### After Block is Rendered before returning HTML

```
  Mage::dispatchEvent('core_block_abstract_to_html_after', array('block' => $this, 'transport' => self::$\_transportObject));
```
