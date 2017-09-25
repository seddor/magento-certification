#Describe URL structure/processing in Magento

## Magento URL structure

Из Mage_Core_Model_Url:
```
* URL structure:
*
* https://user:password@host:443/base_path/[base_script][storeview_path]route_name/controller_name/action_name/param1/value1?query_param=query_value#fragment
*       \__________A___________/\____________________________________B_____________________________________/
* \__________________C___________________/              \__________________D_________________/ \_____E_____/
* \_____________F______________/                        \___________________________G______________________/
* \___________________________________________________H____________________________________________________/
*
* - A: authority
* - B: path
* - C: absolute_base_url
* - D: action_path
* - E: route_params
* - F: host_url
* - G: route_path
* - H: route_url
```
Здесь наиболее важны:
  * route_name — имя роуты из конфига модуля
  * controller_name — имя контролера, относильно директории *controllers* (т.е. Foo/Bar/controllers/catalog/TestControlle = catalog_test), по умолчанию index
  * action_name — имя метода-экшена, по умолчанию index

При матчинге роутов, к реквесту приписываются имена найденого модуля, контролера, экшена, если их удалось извлечь из переданого пути

## Реврайты

Реврайты бывают двух видов(в порядке проверки во фронт контроллере):

* Реврайты из БД
* Реврайты из конфигов


### Реврайты из БД

#### Таблица `core_url_rewrite`

Таблица содержит следующие поля:
  * url_rewrite_id
  * store_id
  * id_path — Используется уникальный идентификатор, например *product/1* или *category/1* или *55306700_1436466387* (кастомный редирект)
  * request_path — Запрашиваемый путь, например *sample-product.html*
  * target_path — настоящий путь, например *catalog/product/view/id/1* или *my-sample-product.html* (кастомный редирект)
  * is_system — флаг, того, что URL создан автоматически(используется при индексации)
  * options — кастомные опции редиректа, тут может быть ничего, `RP` или `R`
  * description
  * category_id
  * product_id

Урл могут быть:
* системные — генерирует сама Маджента, при сохранении продкта или категории, например
```
    id_path = product/1
    request_path = my-sample-product.html
    target_path = catalog/product/view/id/1
    is_system = 1
    product_id = 1
```
* кастомные — вводятся вручную в админке в URL Rewrite Management, или при изменение урлов продуктов, категорий, например
```
    id_path = random_string
    request_path = old-url.html
    target_path = my-sample-product.html
    is_system = 0
    options = RP
```
В URL Rewrite Management можно указать опцию реврайта(PR(для постоянных редиректо) или R(для временых редиректов))

### Метод _rewriteDb()
код:
```php
<?
protected function _rewriteDb()
    {
        if (null === $this->_rewrite->getStoreId() || false === $this->_rewrite->getStoreId()) {
            $this->_rewrite->setStoreId($this->_app->getStore()->getId());
        }

        $requestCases = $this->_getRequestCases();
        $this->_rewrite->loadByRequestPath($requestCases);

        $fromStore = $this->_request->getQuery('___from_store');
        if (!$this->_rewrite->getId() && $fromStore) {
            $stores = $this->_app->getStores(false, true);
            if (!empty($stores[$fromStore])) {
                /** @var $store Mage_Core_Model_Store */
                $store = $stores[$fromStore];
                $fromStoreId = $store->getId();
            } else {
                return false;
            }

            $this->_rewrite->setStoreId($fromStoreId)->loadByRequestPath($requestCases);
            if (!$this->_rewrite->getId()) {
                return false;
            }

            // Load rewrite by id_path
            $currentStore = $this->_app->getStore();
            $this->_rewrite->setStoreId($currentStore->getId())->loadByIdPath($this->_rewrite->getIdPath());

            $this->_setStoreCodeCookie($currentStore->getCode());

            $targetUrl = $currentStore->getBaseUrl() . $this->_rewrite->getRequestPath();
            $this->_sendRedirectHeaders($targetUrl, true);
        }

        if (!$this->_rewrite->getId()) {
            return false;
        }

        $this->_request->setAlias(Mage_Core_Model_Url_Rewrite::REWRITE_REQUEST_PATH_ALIAS,
            $this->_rewrite->getRequestPath());
        $this->_processRedirectOptions();

        return true;
    }
```

### Реврайты из конфигов

#### Определение реврайта в конфиге
в config.xml
```xml
<global>
    <rewrite>
        <foo_bar_rewrite>
            <from><![CDATA[#^/foo/bar/baz/?$#]]></from> <!-- Здесь задается регулярка -->
            <to><![CDATA[/another/path/index/]]></to>
            <complete>1</complete>
        </foo_bar_rewrite>
    </rewrite>
</global>
```

#### Метод _rewriteConfig()
```php
<?
protected function _rewriteConfig()
{
    $config = $this->_config->getNode('global/rewrite');
    if (!$config) {
        return false;
    }
    foreach ($config->children() as $rewrite) {
        $from = (string)$rewrite->from;
        $to = (string)$rewrite->to;
        if (empty($from) || empty($to)) {
            continue;
        }
        $from = $this->_processRewriteUrl($from);
        $to   = $this->_processRewriteUrl($to);

        $pathInfo = preg_replace($from, $to, $this->_request->getPathInfo());
        if (isset($rewrite->complete)) {
            $this->_request->setPathInfo($pathInfo);
        } else {
            $this->_request->rewritePathInfo($pathInfo);
        }
    }
    return true;
}
```
