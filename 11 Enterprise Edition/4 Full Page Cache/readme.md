# Identify the elements and functioning of Enterprise Edition Full Page Cache

Fullpage cache предназначен для полностраничного кеширования, модуль отвечающий за это `Enterprise_PageCache`.

## Процесс запроса

1. в index.php вызывается `Mage::run()`
2. Инстаницируется объект `Mage_Core_Model_App` у него вызается `run()`
3. В этом метода у интстанца `Mage_Core_Model_Cache` взывается метод `processRequest()`

```php
if ($this->_cache->processRequest()) {
  $this->getResponse()->sendResponse();
}
```

4. В `processRequest()` вызывается кеш процессор из поля `_requestProcessors`, значение поля берётся из опций передаваемыйх при инцилизации объекта `Mage_Core_Model_Cache`. Значение берётся из файла `enterprise.xml` (загрружается в Mage_Core_Model_Config->loadBase())

```xml
<config>
    <global>
        <cache>
            <request_processors>
                <ee>Enterprise_PageCache_Model_Processor</ee>
            </request_processors>
            <frontend_options>
                <slab_size>1040000</slab_size>
            </frontend_options>
        </cache>
        <full_page_cache>
            <backend>Mage_Cache_Backend_File</backend>
            <backend_options>
                <cache_dir>full_page_cache</cache_dir>
            </backend_options>
        </full_page_cache>
        <skip_process_modules_updates>0</skip_process_modules_updates>
    </global>
</config>
```

5. У `Enterprise_PageCache_Model_Processor` вызывается метод `extractContent`

```php
<?
public function extractContent($content)
{
    $cacheInstance = Enterprise_PageCache_Model_Cache::getCacheInstance();
    /*
     * Apply design change
     */
    $designChange = $cacheInstance->load($this->getRequestCacheId() . self::DESIGN_CHANGE_CACHE_SUFFIX);
    if ($designChange) {
        $designChange = unserialize($designChange);
        if (is_array($designChange) && isset($designChange['package']) && isset($designChange['theme'])) {
            $designPackage = Mage::getSingleton('core/design_package');
            $designPackage->setPackageName($designChange['package'])
                ->setTheme($designChange['theme']);
        }
    }

    if (!$this->_designExceptionExistsInCache) {
        //no design exception value - error
        //must be at least empty value
        return false;
    }
    if (!$content && $this->isAllowed()) {
        $subprocessorClass = $this->getMetadata('cache_subprocessor');
        if (!$subprocessorClass) {
            return $content;
        }

        /*
         * @var Enterprise_PageCache_Model_Processor_Default
         */
        $subprocessor = new $subprocessorClass;
        $this->setSubprocessor($subprocessor);
        $cacheId = $this->prepareCacheId($subprocessor->getPageIdWithoutApp($this));

        $content = $cacheInstance->load($cacheId);

        if ($content) {
            if (function_exists('gzuncompress')) {
                $content = gzuncompress($content);
            }
            $content = $this->_processContent($content);

            // restore response headers
            $responseHeaders = $this->getMetadata('response_headers');
            $response = Mage::app()->getResponse();
            if (is_array($responseHeaders)) {
                $response->clearHeaders();
                foreach ($responseHeaders as $header) {
                    $response->setHeader($header['name'], $header['value'], $header['replace']);
                }
            }

            // renew recently viewed products
            $productId = $cacheInstance->load($this->getRequestCacheId() . '_current_product_id');
            $countLimit = $cacheInstance->load($this->getRecentlyViewedCountCacheId());
            if ($productId && $countLimit) {
                Enterprise_PageCache_Model_Cookie::registerViewedProducts($productId, $countLimit);
            }
        }

    }
    return $content;
}
```

6. Если контент загружен из кеша, то это контент возвращается в боди, и затем отсылается респонс
```
Mage::app()->getResponse()->appendBody($content);
```

7. Сохранение в кеш происходит по `controller_front_send_response_before`

```xml
<controller_front_send_response_before>
  <observers>
    <enterprise_pagecache>
      <class>enterprise_pagecache/observer</class>
      <method>cacheResponse</method>
    </enterprise_pagecache>
  </observers>
</controller_front_send_response_before>
```

```php
<?
public function cacheResponse(Varien_Event_Observer $observer)
{
    if (!$this->isCacheEnabled()) {
        return $this;
    }
    $frontController = $observer->getEvent()->getFront();
    $request = $frontController->getRequest();
    $response = $frontController->getResponse();
    $this->_saveDesignException();
    $this->_saveCookieConfig();
    $this->_checkAndSaveSslOffloaderHeaderToCache();
    $this->_processor->processRequestResponse($request, $response);
    return $this;
}
```

8. Сохранение происходит в `Enterprise_PageCache_Model_Processor->processRequestResponse()`

## Placeholder — динамические блоки

Вещи которые нельзя хранить в полностраничном кеше частично хранятся в куки.

- Список плейсхолдеров хранится в `cache.xml`

```xml
<placeholders>
  <last_viewed_products>
    <block>reports/product_viewed</block>
    <placeholder>VIEWED_PRODUCTS</placeholder>
    <container>Enterprise_PageCache_Model_Container_Viewedproducts</container>
    <cache_lifetime>86400</cache_lifetime>
  </last_viewed_products>
</placeholders>
```
- регистрация блока происходит в ивенте `core_block_abstract_to_html_before`

```xml
<core_block_abstract_to_html_before>
    <observers>
        <enterprise_pagecache_register_parent_blocks>
            <class>enterprise_pagecache/observer</class>
            <method>registerBlockContext</method>
        </enterprise_pagecache_register_parent_blocks>
    </observers>
</core_block_abstract_to_html_before>
```

```php
<?
public function registerBlockContext(Varien_Event_Observer $observer)
{
    if (!$this->isCacheEnabled()) {
        return $this;
    }
    $block = $observer->getEvent()->getBlock();
    $this->registerContext($block);
    return $this;
}
```

- Классы плейсхолдеров наслдеуются от абстрактного класса `Enterprise_PageCache_Model_Container_Abstract`

### Использование плейсхолдеров

1. Добавление блока из кеша происходит в `Enterprise_PageCache_Model_Processor->_processContainers()`

```php
<?
protected function _processContainers(&$content)
{
    $placeholders = array();
    preg_match_all(
        Enterprise_PageCache_Model_Container_Placeholder::HTML_NAME_PATTERN,
        $content, $placeholders, PREG_PATTERN_ORDER
    );
    $placeholders = array_unique($placeholders[1]);
    $containers = array();
    foreach ($placeholders as $definition) {
        $placeholder = new Enterprise_PageCache_Model_Container_Placeholder($definition);
        $container = $placeholder->getContainerClass();
        if (!$container) {
            continue;
        }

        $container = new $container($placeholder);
        $container->setProcessor($this);
        if (!$container->applyWithoutApp($content)) {
            $containers[] = $container;
        } else {
            preg_match($placeholder->getPattern(), $content, $matches);
            if (array_key_exists(1,$matches)) {
                $containers = array_merge($this->_processContainers($matches[1]), $containers);
                $content = preg_replace($placeholder->getPattern(), str_replace('$', '\\$', $matches[1]), $content);
            }
        }
    }
    return $containers;
}
```
2. Если из плейсхолдера не удалось извлечь контент без полной инициализации приложения, то контенеры добавляются в `Mage::register('cached_page_containers', $containers);`
3. Маджента отправляет запрос к `Enterprise_PageCache_RequestController`
```php
<?
Mage::app()->getRequest()
    ->setModuleName('pagecache')
    ->setControllerName('request')
    ->setActionName('process')
    ->isStraight(true);
```    
4. В `Enterprise_PageCache_RequestController->processAction()` берутся контейнеры из регистри и у каждого вызывается `applyInApp`.
5. Контент добавляется в тело ответа, в сессию сохранячется различная системная информация

```php
<?
public function processAction()
{
    $processor  = Mage::getSingleton('enterprise_pagecache/processor');
    $content    = Mage::registry('cached_page_content');
    $containers = Mage::registry('cached_page_containers');
    $cacheInstance = Enterprise_PageCache_Model_Cache::getCacheInstance();
    foreach ($containers as $container) {
        $container->applyInApp($content);
    }
    $this->getResponse()->appendBody($content);
    // save session cookie lifetime info
    $cacheId = $processor->getSessionInfoCacheId();
    $sessionInfo = $cacheInstance->load($cacheId);
    if ($sessionInfo) {
        $sessionInfo = unserialize($sessionInfo);
    } else {
        $sessionInfo = array();
    }
    $session = Mage::getSingleton('core/session');
    $cookieName = $session->getSessionName();
    $cookieInfo = array(
        'lifetime' => $session->getCookie()->getLifetime(),
        'path'     => $session->getCookie()->getPath(),
        'domain'   => $session->getCookie()->getDomain(),
        'secure'   => $session->getCookie()->isSecure(),
        'httponly' => $session->getCookie()->getHttponly(),
    );
    if (!isset($sessionInfo[$cookieName]) || $sessionInfo[$cookieName] != $cookieInfo) {
        $sessionInfo[$cookieName] = $cookieInfo;
        // customer cookies have to be refreshed as well as the session cookie
        $sessionInfo[Enterprise_PageCache_Model_Cookie::COOKIE_CUSTOMER] = $cookieInfo;
        $sessionInfo[Enterprise_PageCache_Model_Cookie::COOKIE_CUSTOMER_GROUP] = $cookieInfo;
        $sessionInfo[Enterprise_PageCache_Model_Cookie::COOKIE_CUSTOMER_LOGGED_IN] = $cookieInfo;
        $sessionInfo[Enterprise_PageCache_Model_Cookie::CUSTOMER_SEGMENT_IDS] = $cookieInfo;
        $sessionInfo[Enterprise_PageCache_Model_Cookie::COOKIE_MESSAGE] = $cookieInfo;
        $sessionInfo = serialize($sessionInfo);
        $cacheInstance->save($sessionInfo, $cacheId, array(Enterprise_PageCache_Model_Processor::CACHE_TAG));
    }
}
```
