# Describe the stages in the lifecycle of a block

## Which class is responsible for creating an instance of the block?

Mage_Core_Model_Layout

## Which class is responsible for figuring out which blocks should be created for certain pages?

Mage_Core_Model_Layout_Update

## How is the tree of blocks typically rendered?

Дерево рендерится от корня к дочерним блокам, через childHtml

## Is it possible to create an instance of the block and render it on the page without using the Magento layout?

Можно использовать Mage_Core_Model_Layout->createBlock();

## Is it possible to create an instance of the block and add it to the current layout manually?

Да, через Mage_Core_Model_Layout.

## How are a block’s children rendered? Once you added a child to the block, can you expect it will be rendered automatically?

Вывод блока проихсодит вручную, через $this->getChildHtml('name');

Блок Mage_Core_Block_Text_List рендерит всех своих детей в методе \_toHtml()

## What is a difference in rendering process for different types of blocks?

Вывод темплейта осущевтвялется через \_toHtml(), в случае с Mage_Core_Block_Template в нём рендерится темплйт блока
