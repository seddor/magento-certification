#Describe how to implement advanced Adminhtml Grids and Forms

## Which block class do Magento grid classes typically extend?

`Mage_Adminhtml_Block_Widget_Grid`

## What is the default template for Magento grid instances?

*widget/grid.phtml*

## How can grid filters be customized?

Можно использовать метод колонки грида setFilterConditionCallback которая принимает функцию, которая будет использована как фильтр

```php
<?
protected function _addColumnFilterToCollection($column)
    {
        if ($this->getCollection()) {
            $field = ( $column->getFilterIndex() ) ? $column->getFilterIndex() : $column->getIndex();
            if ($column->getFilterConditionCallback()) {
                call_user_func($column->getFilterConditionCallback(), $this->getCollection(), $column);
            } else {
                $cond = $column->getFilter()->getCondition();
                if ($field && isset($cond)) {
                    $this->getCollection()->addFieldToFilter($field , $cond);
                }
            }
        }
        return $this;
    }
```    

## How does Magento actually perform sorting/paging/filtering operations?


```php
<?
/**
 * Sets sorting order by some column
 *   
 * @param Mage_Adminhtml_Block_Widget_Grid_Column $column
 * @return Mage_Adminhtml_Block_Widget_Grid
 */
protected function _setCollectionOrder($column)
{
    $collection = $this->getCollection();
    if ($collection) {
        $columnIndex = $column->getFilterIndex() ?
            $column->getFilterIndex() : $column->getIndex();
        $collection->setOrder($columnIndex, strtoupper($column->getDir()));
    }
    return $this;
}

protected function _preparePage()
{
    $this->getCollection()->setPageSize((int) $this->getParam($this->getVarNameLimit(), $this->_defaultLimit));
    $this->getCollection()->setCurPage((int) $this->getParam($this->getVarNamePage(), $this->_defaultPage));
}

protected function _addColumnFilterToCollection($column)
    {
        if ($this->getCollection()) {
            $field = ( $column->getFilterIndex() ) ? $column->getFilterIndex() : $column->getIndex();
            if ($column->getFilterConditionCallback()) {
                call_user_func($column->getFilterConditionCallback(), $this->getCollection(), $column);
            } else {
                $cond = $column->getFilter()->getCondition();
                if ($field && isset($cond)) {
                    $this->getCollection()->addFieldToFilter($field , $cond);
                }
            }
        }
        return $this;
    }
```

## What protected methods are specific to adminhtml grids, and how are they used?

TODO:Описание
  * protected function \_prepareLayout()
  * protected function \_prepareCollection()
  * protected function \_preparePage()
  * protected function \_prepareColumns()
  * protected function \_prepareMassactionBlock()
  * protected function \_prepareMassaction()
  * protected function \_prepareMassactionColumn()
  * protected function \_prepareGrid()
  * protected function \_beforeToHtml()
  * protected function \_afterLoadCollection()
  * protected function \_getExportHeaders()
  * protected function \_getExportTotals()
  * protected function \_setFilterValues($data)
  * protected function \_addColumnFilterToCollection($column)
  * protected function \_setCollectionOrder($column)
  * protected function \_getRssUrl($url)
  * protected function \_getFileContainerContent(array $fileData)
  * protected function \_exportCsvItem(Varien_Object $item, Varien_Io_File $adapter)
  * protected function \_addLinkModelFilterCallback($collection, $colum * n)
  * protected function \_exportExcelItem(Varien_Object $item, Varien_Io_File $adapter, $parser = null)
  * protected function \_decodeFilter(&$value)

## What is the standard column class in a grid, and what is its role?

`Mage_Adminhtml_Block_Widget_Grid_Column`

## What are column renderers used for in Magento?

Из класса `Mage_Adminhtml_Block_Widget_Grid_Column`

```php
<?
protected function _getRendererByType()
    {
        $type = strtolower($this->getType());
        $renderers = $this->getGrid()->getColumnRenderers();

        if (is_array($renderers) && isset($renderers[$type])) {
            return $renderers[$type];
        }

        switch ($type) {
            case 'date':
                $rendererClass = 'adminhtml/widget_grid_column_renderer_date';
                break;
            case 'datetime':
                $rendererClass = 'adminhtml/widget_grid_column_renderer_datetime';
                break;
            case 'number':
                $rendererClass = 'adminhtml/widget_grid_column_renderer_number';
                break;
            case 'currency':
                $rendererClass = 'adminhtml/widget_grid_column_renderer_currency';
                break;
            case 'price':
                $rendererClass = 'adminhtml/widget_grid_column_renderer_price';
                break;
            case 'country':
                $rendererClass = 'adminhtml/widget_grid_column_renderer_country';
                break;
            case 'concat':
                $rendererClass = 'adminhtml/widget_grid_column_renderer_concat';
                break;
            case 'action':
                $rendererClass = 'adminhtml/widget_grid_column_renderer_action';
                break;
            case 'options':
                $rendererClass = 'adminhtml/widget_grid_column_renderer_options';
                break;
            case 'checkbox':
                $rendererClass = 'adminhtml/widget_grid_column_renderer_checkbox';
                break;
            case 'massaction':
                $rendererClass = 'adminhtml/widget_grid_column_renderer_massaction';
                break;
            case 'radio':
                $rendererClass = 'adminhtml/widget_grid_column_renderer_radio';
                break;
            case 'input':
                $rendererClass = 'adminhtml/widget_grid_column_renderer_input';
                break;
            case 'select':
                $rendererClass = 'adminhtml/widget_grid_column_renderer_select';
                break;
            case 'text':
                $rendererClass = 'adminhtml/widget_grid_column_renderer_longtext';
                break;
            case 'store':
                $rendererClass = 'adminhtml/widget_grid_column_renderer_store';
                break;
            case 'wrapline':
                $rendererClass = 'adminhtml/widget_grid_column_renderer_wrapline';
                break;
            case 'theme':
                $rendererClass = 'adminhtml/widget_grid_column_renderer_theme';
                break;
            default:
                $rendererClass = 'adminhtml/widget_grid_column_renderer_text';
                break;
        }
        return $rendererClass;
    }
```

## How can JavaScript used for a Magento grid be customized?

Можно ввоспользоватся магическим мметодом `getAdditionalJavaScript` у грида

Его содержимое буде выведено в темлейте грида здесь:
```php
<script type="text/javascript">
//<![CDATA[
    <?php echo $this->getJsObjectName() ?> = new varienGrid('<?php echo $this->getId() ?>', '<?php echo $this->getGridUrl() ?>', '<?php echo $this->getVarNamePage() ?>', '<?php echo $this->getVarNameSort() ?>', '<?php echo $this->getVarNameDir() ?>', '<?php echo $this->getVarNameFilter() ?>');
    <?php echo $this->getJsObjectName() ?>.useAjax = '<?php echo $this->getUseAjax() ?>';
    <?php if($this->getRowClickCallback()): ?>
        <?php echo $this->getJsObjectName() ?>.rowClickCallback = <?php echo $this->getRowClickCallback() ?>;
    <?php endif; ?>
    <?php if($this->getCheckboxCheckCallback()): ?>
        <?php echo $this->getJsObjectName() ?>.checkboxCheckCallback = <?php echo $this->getCheckboxCheckCallback() ?>;
    <?php endif; ?>
    <?php if($this->getRowInitCallback()): ?>
        <?php echo $this->getJsObjectName() ?>.initRowCallback = <?php echo $this->getRowInitCallback() ?>;
        <?php echo $this->getJsObjectName() ?>.initGridRows();
    <?php endif; ?>
    <?php if($this->getMassactionBlock()->isAvailable()): ?>
    <?php echo $this->getMassactionBlock()->getJavaScript() ?>
    <?php endif ?>
    <?php echo $this->getAdditionalJavaScript(); ?>
//]]>
</script>
```

## What is the role of the grid container class and its template?

Контейнер гридов(`Mage_Adminhtml_Block_Widget_Grid_Container`) — это класс, который включает в себя грид. Обычно он содержит такие вещи, как кнопки, тайлтл и т.д.

Грид автоматически добавляется к контейнеру

```php
<?
protected function _prepareLayout()
{
    $this->setChild( 'grid',
        $this->getLayout()->createBlock( $this->_blockGroup.'/' . $this->_controller . '_grid',
        $this->_controller . '.grid')->setSaveParametersInSession(true) );
    return parent::_prepareLayout();
}
```

## What is the programmatic structure of mass actions?

Для массовых дествий в блоке грида используется метод `_prepareMassaction`

метод для получения блока `Mage_Adminhtml_Block_Widget_Grid_Massaction_Abstract`

```
public function getMassactionBlock()
{
    return $this->getChild('massaction');
}
```

также для массовых действий требуется экшен в контролерере
