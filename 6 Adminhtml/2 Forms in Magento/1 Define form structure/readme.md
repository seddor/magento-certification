# Define form structure, form templates, grids in Magento, and grid containers and elements

## Which block does a standard Magento form extend?

  * Класс контейнера — `Mage_Adminhtml_Block_Widget_Form_Container`
  * Класс форм — `Mage_Adminhtml_Block_Widget_Form`

## What is the default template for a Magento form?

  * темплейт контейнера — *widget/form/container.phtml*
  * темплейт формы — *widget/form.phtml*

## Describe the role of a form container and its template.

Контейнер содержит кнопки, хеадер.

темплейт
```php
<?php echo $this->getFormInitScripts() ?>
<div class="content-header">
    <?php echo $this->getHeaderHtml() ?>
    <p class="form-buttons"><?php echo $this->getButtonsHtml('header') ?></p>
</div>
<?php echo $this->getFormHtml() ?>
<?php if ($this->hasFooterButtons()): ?>
    <div class="content-footer">
        <p class="form-buttons"><?php echo $this->getButtonsHtml('footer') ?></p>
    </div>
<?php endif; ?>
<script type="text/javascript">
    editForm = new varienForm('edit_form', '<?php echo $this->getValidationUrl() ?>');
</script>
<?php echo $this->getFormScripts() ?>
```

## Describe the concept of Form elements, and list system elements implemented in Magento.

Класс — `Mage_Adminhtml_Block_Widget_Form_Element`
Темплейт — *widget/form/element.phtml*

минимальный элемент формы

## Describe the concept of Fieldsets.

Класс — `Varien_Data_Form_Element_Fieldset`
Содержит набо элементов

## How can you render an element with a custom template?

 1. `setTemplate()` метод у элемента
 2. Создать кастомный блок со своим темлейтом и добавить на форму
 ```
 $gallery = $this->getLayout()->createBlock('foo_adminhtml/widget_form_galleryPreview')
    ->setImages($images);
$fieldset->addField('images', 'text', [
    'label' => $this->__('Images'),
    'name'  => 'images',
])->setRenderer($gallery);
 ```
