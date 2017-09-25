# Describe the similarities and differences between adminhtml and frontend interface and routing

```xml
<routers>
    <admin>
        <area>admin</area>
        <class>Mage_Core_Controller_Varien_Router_Admin</class>
    </admin>
    <standard>
        <area>frontend</area>
        <class>Mage_Core_Controller_Varien_Router_Standard</class>
    </standard>
</routers>
```

## Which areas in configuration are only loaded for the admin area?

все настройки в ноде `<admin>`

## What is the difference between admin and frontend controllers?

* Фронтедновские контролеры наследуются от `Mage_Core_Controller_Front_Action`
* админские контролеры наследуют от `Mage_Adminhtml_Controller_Action` админские контролеры так же находяется под ACL

## When does Magento figure out which area to use on the current page?

Это определяется в  Mage_Core_Controller_Varien_Front->dispatch(), в методе preDispatch()

In Frontend controller:
```
    public function preDispatch()
    {
        $this->getLayout()->setArea($this->_currentArea);

        parent::preDispatch();
        return $this;
    }
```
In Backend controller:
```
    public function preDispatch()
    {
        // override admin store design settings via stores section
        Mage::getDesign()
            ->setArea($this->_currentArea)
            ->setPackageName((string)Mage::getConfig()->getNode('stores/admin/design/package/name'))
            ->setTheme((string)Mage::getConfig()->getNode('stores/admin/design/theme/default'))
        ;
        ....
    }
```
## How you can make your controller work under the /admin route?
```xml
<admin>
    <routers>
        <adminhtml>
            <args>
                <modules>
                    <dealer before="Mage_Adminhtml">Foo_Bar_Adminhtml</dealer>
                </modules>
            </args>
        </adminhtml>
    </routers>
</admin>
```
