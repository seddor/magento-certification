# Create a simple form and grid for a custom entity

## Создание грида

  1. Экшен в контроллере
  ```php
  <?
  public function indexAction()
  {
      $this->loadLayout()->renderLayout();
  }
  ```
  2. Хендл
  ```xml
  <adminhtml_foo_bar_index>
      <reference name="content">
          <block type="foo_bar/adminhtml_baz" name="baz"></block>
      </reference>
  </adminhtml_foo_bar_index>
  ```
  3. Контейнер грида
  ```php
  <?
  class Foo_Bar_Block_Adminhtml_Baz extends Mage_Adminhtml_Block_Widget_Grid_Container
  {
    public function _construct()
    {
        parent::_construct();
        $this->_blockGroup = 'foo_bar';
        $this->_controller = 'adminhtml_baz';//путь до класса грида
    }
  }
  ```
  4. Грид
  ```php
  <?
  class Foo_Bar_Block_Adminhtml_Baz_Grid extends Mage_Adminhtml_Block_Widget_Grid
  {
    protected function _prepareCollection()
    {
        $this->setCollection(Mage::getModel('foo_bar/baz')->getCollection());
        return parent::_prepareCollection();
    }

    protected function _prepareColumns()
    {
        $this->addColumn('id', array(
            'header' => $this->__('ID'),
            'index' => 'id',
            'type' => 'number',
        ));

        $this->addColumn('name', array(
            'header' => $this->__('Name'),
            'index' => 'name'
        ));

        return parent::_prepareColumns();
    }

    public function getRowUrl($model)
    {
        return $this->getUrl('*/*/edit', array(
            'id' => $model->getId(),
        ));
    }
  }
  ```

## Создание формы

  1. Экшен в контроллере

  ```php
  <?
  public function editAction()
  {
      try {
          $this->_initModel();
          $this->_initAction()
              ->renderLayout();
      } catch (Mage_Core_Exception $e) {
          $this->_getSession()->addError($e->getMessages());
          $this->_redirectReferer();
      } catch (Exception $e) {
          $this->_getSession()->addError('System error');
          $this->_redirectReferer();
      }
  }

  private function _initModel()
  {
    $id = $this->getRequest()->getParam('id');
    $model = Mage::getModel('foo_bar/baz')->load($id);
    Mage::register('current_model', $model);
    return $model;
  }
  ```

  2. Хендл
  ```xml
  <adminhtml_foo_bar_edit>
      <reference name="content">
          <block type="foo_bar/adminhtml_baz_edit" name="baz.form"></block>
      </reference>
  </adminhtml_foo_bar_edit>
  ```
  3. Контейнер формы
  ```php
  <?
  class Foo_Bar_Block_Adminhtml_Baz_Edit extends Mage_Adminhtml_Block_Widget_Form_Container
{
    protected function _construct()
    {
        $this->_blockGroup = 'foo_bar';
        $this->_controller = 'adminhtml_baz';
    }

    /**
     * set header text
     *
     * @return string
     */
    public function getHeaderText()
    {
        if ($model = $this->_model()) {
            return $this->__("Edit Baz %s",
                $this->escapeHtml($model->getName()));
        } else {
            return $this->__('Add new Baz);
        }
    }

    public function getSaveUrl()
    {
        return $this->getUrl('*/*/save', array('_current' => true, 'back' => null));
    }

    private function _model()
    {
        return Mage::registry('current_model');
    }
}
  ```
  4. Грид
  ```php
  <?
  class Foo_Bar_Block_Adminhtml_Baz_Edit_Form extends Mage_Adminhtml_Block_Widget_Form
  {
      protected function _prepareForm()
      {
          $form = new Varien_Data_Form(array(
              'id'     => 'edit_form',
              'method' => 'post',
              'action' => $this->getData('action')
          ));

          $this->_prepareBazInfo($form);

          $form->setUseContainer(true);
          $form->addFieldNameSuffix('baz');
          if ($baz = $this->_model()) {
              $form->setValues($baz->getData());
          }

          $this->setForm($form);

          return parent::_prepareForm();
      }

      protected function _prepareBazInfo(Varien_Data_Form $form)
      {
          $fieldset = $form->addFieldset('baz_data',
          ['legend' => $this->__('Baz Information')]);

          $fieldset->addField('id', 'label', array(
              'label' => $this->__('ID'),
              'name'  => 'id',
          ));

          $fieldset->addField('name', 'label', array(
              'label' => $this->__('Name'),
              'name'  => 'name',
          ));
      }

      private function _model()
      {
          return Mage::registry('current_model');
      }

  }
  ```
