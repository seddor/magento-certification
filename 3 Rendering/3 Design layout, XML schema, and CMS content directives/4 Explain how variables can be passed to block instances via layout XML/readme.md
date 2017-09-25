#Explain how variables can be passed to block instances via layout XML

## How can variables be passed to the block using the following methods?
  * From layout xml file
 с помощью action
 ```xml
   <action method="setFoo"><data>Foo</data></action>
 ```

  * From controller

  получить блок с помощью $this->getLayout()->getBlock('name')->setFoo('Foo')

  * From one block to another

  $this->getChild('name')->setFoo('Foo')

  * From an arbitrary location (for example, install/upgrade scripts, models)

  Mage::app()->getLayout()->getBlock('name')->setFoo('Foo')

## In which circumstances are each of the above methods more or less appropriate than others?
