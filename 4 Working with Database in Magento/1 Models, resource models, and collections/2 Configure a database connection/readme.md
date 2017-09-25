# Configure a database connection

Настройки конекта с БД конфигурируются в *app/etc/local.xml*

```xml
<resources>
  <db>
    <table_prefix><![CDATA[]]></table_prefix>
  </db>
  <default_setup>
    <connection>
      <host><![CDATA[localhost]]></host>
      <username><![CDATA[root]]></username>
      <password><![CDATA[123123]]></password>
      <dbname><![CDATA[magento]]></dbname>
      <initStatements><![CDATA[SET NAMES utf8]]></initStatements>
      <model><![CDATA[mysql4]]></model>
      <type><![CDATA[pdo_mysql]]></type>
      <pdoType><![CDATA[]]></pdoType>
      <active>1</active>
      </connection>
  </default_setup>
</resources>
```

Основные конекты конфигурируются в *app/etc/config.xml*

```xml
<resources>
  <default_setup>
    <connection>
      <host>localhost</host>
      <username/>
      <password/>
      <dbname>magento</dbname>
      <model>mysql4</model>
      <initStatements>SET NAMES utf8</initStatements>
      <type>pdo_mysql</type>
      <active>0</active>
      <persistent>0</persistent>
    </connection>
  </default_setup>
  <default_write>
    <connection>
      <use>default_setup</use>
    </connection>
  </default_write>
  <default_read>
    <connection>
      <use>default_setup</use>
    </connection>
  </default_read>
  <core_setup>
    <setup>
      <module>Mage_Core</module>
    </setup>
    <connection>
      <use>default_setup</use>
    </connection>
  </core_setup>
  <core_write>
    <connection>
      <use>default_write</use>
    </connection>
  </core_write>
  <core_read>
    <connection>
      <use>default_read</use>
    </connection>
  </core_read>
</resources>
```


## Добавление кастомных конекшенов

*app/code/local/Colin/Database/etc/config.xml*

```xml
<wordpress_db>
    <connection>
        <host><![CDATA[127.0.0.1]]></host>
        <username><![CDATA[username]]></username>
        <password><![CDATA[password]]></password>
        <dbname><![CDATA[wp_exam]]></dbname>
        <initStatements><![CDATA[SET NAMES utf8]]></initStatements>
        <model><![CDATA[mysql4]]></model>
        <type><![CDATA[pdo_mysql]]></type>
        <active>1</active>
        <persistent>0</persistent>
    </connection>
</wordpress_db>
<wordpress_read>
    <connection>
        <use>wordpress_db</use>
    </connection>
</wordpress_read>
<wordpress_write>
    <connection>
        <use>wordpress_db</use>
    </connection>
</wordpress_write>
```

*app/code/local/Colin/Database/controllers/IndexController.php*

```php
<?
 public function wordpressAction()
 {
     // $resource Mage_Core_Model_Resource
     $resource = Mage::getSingleton('core/resource');
     $connection = $resource->getConnection('wordpress_db');

     // $connection Varien_Db_Adapter_Interface
     $connection = $resource->getConnection('wordpress_read');

     try {
         $posts = $connection->query("
           SELECT post_title
           FROM wp_posts
           WHERE post_status='publish'
           AND post_type = 'post'
           ORDER BY post_date DESC
         ");
     } catch (Exception $e) {
         echo $e->getMessage();
         return;
     }

     // $posts Varien_Db_Statement_Pdo_Mysql
     foreach ($posts as $post) {
         echo "<strong>Title:</strong> " . $post['post_title'] . "<br />";
     }
 }
```
