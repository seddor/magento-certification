# Create and add code to pages

## How can code be modified or added to Magento pages using the following methods?

  * Template customizations
```xml
  <action method="setTemplate"><template>another/template.phtml</template></action>
```
или
```xml
<block type="core/template" name="block" template="template.phtml">
```
  * Layout customizations

  Можно измениять через local.xml или лаяут модуля

  * Overriding block classes

```
<block type="my_block/name" name="override"
```

  * Registering observers on general block events

app/code/local/Colin/Request/etc/config.xml

```xml
<global>
  <core_block_abstract_prepare_layout_before>
      <observers>
          <colin_request_prepare_layout>
              <class>colin_request/observer</class>
              <method>generateBlocks</method>
              <type>singleton</type>
          </colin_request_prepare_layout>
      </observers>
  </core_block_abstract_prepare_layout_before>
  </events>
</global>
```

app/code/local/Colin/Request/Model/Observer.php

```php
<?
public function generateBlocks($observer)
{
    $block = $observer->getEvent()->getBlock();
    if ($block->getType() == 'page/html_header') {
        // Do Something
    }

    if ($block->getNameInLayout() === 'content') {
        $text = Mage::app()->getLayout()->createBlock('core/text', 'my_string', array('before', '-'));
        $text->setText('Hello World');
        $block->append($text);
    }

}
```

## In which circumstances are each of the above methods more or less appropriate than others?
