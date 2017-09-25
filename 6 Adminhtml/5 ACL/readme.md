# Access Control Lists (ACL) and permissions in Magento

ACL — используется в мадженто для разграничения ролей пользователей админки и API. ACL конфигурируется через `adminhtml.xml`

ACL состоит из 3х частей:

  * Ресурс — часть системы к которой нужен доступ
  * Роль — Роль разграничивает уровень доступа к ресурсам
  * Пользователь — Пользователи системы, им присваива.тся разилчные роли

## Ограничения доступа в админке, меню и контролеры

### создание меню

adminhtml.xml

```xml
<menu>
  <foo translate="title" module="foo_bar">
    <title>Foo</title>
    <sort_order>50</sort_order>
    <children>
      <bar translate="title">
        <title>Bar</title>
        <sort_order>1</sort_order>
        <action>adminhtml/foo/bar</action> <!-- вложеные элементы так же задаются через ноду <children> -->
      </bar>
    </children>
  </foo>
</menu>
```

### создание ACL для меню

adminhtml.xml

```xml
<acl>
  <resources>
    <admin>
      <children>
        <foo translate="title" module="foo_bar">
          <title>Foo</title>
          <sort_order>50</sort_order>
          <children>
            <bar>
              <title>Bar</title>
              <sort_order>1</sort_order>
            </bar>
          </children>
        </foo>
      </children>
    </admin>
  </resources>
</acl>
```

### применгение ACL в контроллере

В контроллере для применения ACL нужно переопределить метод `_isAllowed`

```
protected function _isAllowed()
{
  //указывается путь из adminhtml.xml
    return Mage::getSingleton('admin/session')->isAllowed('foo/bar');
}
```

### создание ACL для секций системной конфигурации

adminhtml.xml

```xml
<acl>
    <resources>
        <admin>
                <system>
                    <children>
                        <config>
                            <children>
                                <foo_bar>
                                    <title>Foo Bar Config</title>
                                </foo_bar>
                            </children>
                        </config>
                    </children>
                </system>
            </children>
        </admin>
    </resources>
</acl>
```

## Работа ACL

роли хранятся в таблице `admin_role`
тип роли G — обозначает, что роль является групповой

| role_id | parent_id | tree_level | sort_order | role_type |	user_id |   role_name    |
| :------ | :-------- | :--------- | :--------- | :-------- | :------ | :------------- |
|  1      |    0      |      1     |     1      |    G      |     0   | Administrators |
|  31     |    0      |      1     |     0      |    G      |     0   | test role      |


Правила по которым ограничиваются ресурсы указываются в таблице `admin_rule`

| rule_id | role_id   | resource_id | privileges | assert_id | role_type | permission |
| :------ | :-------- | :---------  | :--------- | :-------- | :-------- | :--------- |
|  1      |    1      |     all     |     NULL   |    0      |     G     |    allow   |
|  2      |    31     |     all     |     NULL   |    0      |     G     |    deny    |
|  126    |    31     | admin/sales |     NULL   |    0      |     G     |    allow   |

Как видно роли  test role доступнен только ресур `admin/sales`


Проверка доступности ресурса в админке осущевлсяется в `Mage_Adminhtml_Controller_Action` в методе `preDispatch`

```php
<?
if ($this->getRequest()->isDispatched()
    && $this->getRequest()->getActionName() !== 'denied'
    && !$this->_isAllowed()) {
    $this->_forward('denied');
    $this->setFlag('', self::FLAG_NO_DISPATCH, true);
    return $this;
}
```

метод `_isAllowed()` переопределяется в каждом контролере использующем ACL и выглядит примерно так:

```
protected function _isAllowed()
{
    return Mage::getSingleton('admin/session')->isAllowed('resource_id');
}
```

Метод `Mage_Admin_Model_Session`

```php
<?
public function isAllowed($resource, $privilege = null)
{
    $user = $this->getUser();
    $acl = $this->getAcl();

    if ($user && $acl) {
        if (!preg_match('/^admin/', $resource)) {
            $resource = 'admin/' . $resource;
        }

        try {
            return $acl->isAllowed($user->getAclRole(), $resource, $privilege);
        } catch (Exception $e) {
            try {
                if (!$acl->has($resource)) {
                    return $acl->isAllowed($user->getAclRole(), null, $privilege);
                }
            } catch (Exception $e) { }
        }
    }
    return false;
}
```

Доступность ресурса проверяется в `Mage_Admin_Model_Acl` наследуемом от `Zend_Acl` в котором вызывается мето `isAllowed`
