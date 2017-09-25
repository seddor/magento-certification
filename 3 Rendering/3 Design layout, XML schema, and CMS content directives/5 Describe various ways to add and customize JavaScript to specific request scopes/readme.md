# Describe various ways to add and customize JavaScript to specific request scopes

## Which block is responsible for rendering JavaScript in Magento?

Mage_Page_Block_Html_Head

## Which modes of including JavaScript does Magento support?

  * addItem
  * addJs
  * addJsIe

  `addJs` и `addJsIe` внутри себе вызывают `addItem`

```php
<?
/**
 * Add HEAD Item
 *
 * Allowed types:
 *  - js
 *  - js_css
 *  - skin_js
 *  - skin_css
 *  - rss
 *
 * @param string $type
 * @param string $name
 * @param string $params
 * @param string $if
 * @param string $cond
 * @return Mage_Page_Block_Html_Head
 */
public function addItem($type, $name, $params=null, $if=null, $cond=null)
{
    if ($type==='skin_css' && empty($params)) {
        $params = 'media="all"';
    }
    $this->_data['items'][$type.'/'.$name] = array(
        'type'   => $type,
        'name'   => $name,
        'params' => $params,
        'if'     => $if,
        'cond'   => $cond,
   );
    return $this;
}
```

## Which classes and files should be checked if a link to a custom JavaScript file isn’t being rendered on a page?

  * XML лаяуты
  * директорию skin/js
  * Mage_Page_Block_Html_Head
