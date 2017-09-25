# Describe the role of the front controller

Класс front контроллера — `Mage_Core_Controller_Varien_Front`.

Роль этого контролера — сопоставление имеющихся роутов с реквестом, и отправка ответа на реквест.

Фронт контроллер вызывается в конце метода `Mage_Core_Model_App->run()`:
```
$this->getFrontController()->dispatch();
```

Порядок работы фронт контроллера

## 1 Mage_Core_Controller_Varien_Front->init()

код:
```php
<?
public function init()
{
  Mage::dispatchEvent('controller_front_init_before', array('front'=>$this));

  $routersInfo = Mage::app()->getStore()->getConfig(self::XML_STORE_ROUTERS_PATH);

  Varien_Profiler::start('mage::app::init_front_controller::collect_routers');
  foreach ($routersInfo as $routerCode => $routerInfo) {
    if (isset($routerInfo['disabled']) && $routerInfo['disabled']) {
      continue;
    }
    if (isset($routerInfo['class'])) {
      $router = new $routerInfo['class'];
      if (isset($routerInfo['area'])) {
        $router->collectRoutes($routerInfo['area'], $routerCode);
      }
        $this->addRouter($routerCode, $router);
      }
    }
    Varien_Profiler::stop('mage::app::init_front_controller::collect_routers');

    Mage::dispatchEvent('controller_front_init_routers', array('front'=>$this));

    // Add default router at the last
    $default = new Mage_Core_Controller_Varien_Router_Default();
    $this->addRouter('default', $default);

    return $this;
}
```
В этом методе собираются все доступные роуты, по умолчанию их 5:
  * Mage_Core_Controller_Varien_Router_Admin
  * Mage_Core_Controller_Varien_Router_Standard
  * Mage_Install_Controller_Router_Install
  * Mage_Cms_Controller_Router
  * Mage_Core_Controller_Varien_Router_Default

### Добавление кастомных роутов

Можно добавлять кастомные роуты через config.xml
```xml
<default>
    <web>
        <routers>
            <foobar>
                <area>frontend</area>
                <class>Foo_Bar_Controller_Route</class>
            </foobar>
        </routers>
    </web>
</default>
```
Кастомный контроллер должен наследоваться от `Mage_Core_Controller_Varien_Router_Abstract`

Второй способ довление кастомных роутов: через события фронт контроллера
```xml
<global>
  <events>
    <controller_front_init_routers>
      <observers>
        <foo_bar_route>
          <class>Foo_Bar_Controller_Route</class>
          <method>initControllerRouters</method>
        </foo_bar_route>
      </observers>
    </controller_front_init_routers>
  </events>
</global>
```

### Использование существующих роутов

config.xml:
```xml
<frontend>
  <routers>
    <catalog>
      <use>standard</use>
      <args>
        <module>Mage_Catalog</module>
        <frontName>catalog</frontName>
      </args>
    </catalog>
  </routers>
</frontend>
```
Контролеры для фронт-энда должны наследоваться от `Mage_Core_Controller_Front_Action`, для админки от — `Mage_Adminhtml_Controller_Action`.

## 2 Mage_Core_Controller_Varien_Front->dispatch()
код:
```php
<?
public function dispatch()
{
    $request = $this->getRequest();

    // If pre-configured, check equality of base URL and requested URL
    $this->_checkBaseUrl($request);

    $request->setPathInfo()->setDispatched(false);

    $this->_getRequestRewriteController()->rewrite();

    Varien_Profiler::start('mage::dispatch::routers_match');
    $i = 0;
    while (!$request->isDispatched() && $i++ < 100) {
        foreach ($this->_routers as $router) {
            /** @var $router Mage_Core_Controller_Varien_Router_Abstract */
            if ($router->match($request)) {
                break;
            }
        }
    }
    Varien_Profiler::stop('mage::dispatch::routers_match');
    if ($i>100) {
        Mage::throwException('Front controller reached 100 router match iterations');
    }
    // This event gives possibility to launch something before sending output (allow cookie setting)
    Mage::dispatchEvent('controller_front_send_response_before', array('front'=>$this));
    Varien_Profiler::start('mage::app::dispatch::send_response');
    $this->getResponse()->sendResponse();
    Varien_Profiler::stop('mage::app::dispatch::send_response');
    Mage::dispatchEvent('controller_front_send_response_after', array('front'=>$this));
    return $this;
}
```
Здесь сначала происходит процес поиска урлов по реврайтам(подробнее будет рассмотрен позже), затем, если соотвеие не было найдено, то в каждой роуте вызывается метод match, пока не будет найдено соответствие, иначе вызвается дефотлный роут(обычно он ведёт на 404).


В конце вызвается $this->getResponse()->sendResponse();
