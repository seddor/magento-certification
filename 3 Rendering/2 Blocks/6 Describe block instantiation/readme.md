# Describe block instantiation

## How can a template’s block instance be accessed inside the template file, and how can other block instances be accessed?

* getChild — для получения дочернего блока
* getChildHtml — для получения отрендереного контента дочернего блока
* getChildChildHtml — для получения контента дочеренего блока дочернего блока
* getSortedChildBlocks — полусение всех дочерних блоков
* getChildGroup — получение группы детей
* getChildData — получение данных дочернего блока

## How can block instances be accessed from the controller?

$this->getLayout() вернет Mage_Core_Model_Layout, через который можно работать с блоками

## How can block instances be accessed inside install scripts or other model class instances?

Mage::app()->getLayout()
