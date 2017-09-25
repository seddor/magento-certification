# Describe the relationship between templates and blocks

Блоки которые унаследованы от класса Mage_Core_Block_Template, могут иметь шаблоны. Шаблоны — это phtml файлы с версткой. ШШаблоны блоки присваиваются в лаяутах, или вручную через метод setTemplate. Шаблон у блока можно изменять динамически.

## Can any block in Magento use a template file?

Только блоки наследуемые от Mage_Core_Block_Template

## How does the $this variable work inside the template file?

$this ссылается на класс блока, в темлейте можно использовать паблик методы из блока через $this.

## Is it possible to render a template without a block in Magento?

Нет.

## Is it possible to have a block without a template in Magento?

Да. Вывод можно изменять в метода \_toHtml().
