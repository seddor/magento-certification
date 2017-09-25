# Write install and upgrade scripts using set-up resources

Внутри setup и upgrade скрипты ничен не различаются.

## Создание таблиц

```php
<?
$installer = $this;
$installer->startSetup();

try {
    $table = $installer->getConnection()
        ->newTable($installer->getTable('oggetto_catalog/brand'))
        ->addColumn('brand_id', Varien_Db_Ddl_Table::TYPE_INTEGER, null, [
            'unsigned' => true,
            'identity' => true,
            'nullable' => false,
            'primary'  => true,
        ], 'Brand ID')
        ->addColumn('erp_id', Varien_Db_Ddl_Table::TYPE_INTEGER, 11, [], 'ERP ID')
        ->addColumn('name', Varien_Db_Ddl_Table::TYPE_TEXT, 255, [], 'Brand name')
        ->addColumn('show_in_catalog', Varien_Db_Ddl_Table::TYPE_BOOLEAN, 1, [], 'Is displayed in catalog')
        ->addColumn('image', Varien_Db_Ddl_Table::TYPE_TEXT, 255, [], 'Brand image')
        ->addColumn('text', Varien_Db_Ddl_Table::TYPE_TEXT, null, [], 'Brand text')
        ->addColumn('is_active', Varien_Db_Ddl_Table::TYPE_BOOLEAN, 1, ['default' => 1], 'Is active')
        ->addColumn('for_trade_in', Varien_Db_Ddl_Table::TYPE_BOOLEAN, 1, ['default' => 1], 'Is for trade-in')
        ->addColumn(
            'for_service_booking', Varien_Db_Ddl_Table::TYPE_BOOLEAN, 1, ['default' => 1], 'Is for service booking'
        )
        ->setComment('Catalog brands');
    $installer->getConnection()->createTable($table);

$installer->endSetup();

```

## Добаление колонок к существующей

```php
<?
$installer->getConnection()->addColumn(
    $installer->getTable('oggetto_catalog/model'),
    'short_name',
    [
        'type'    => Varien_Db_Ddl_Table::TYPE_TEXT,
        'length'  => 255,
        'default' => null,
        'comment' => 'Model short name'
    ]
);
```

## Удаление колонок

```php
<?
$installer->getConnection()->dropColumn(
    $installer->getTable('oggetto_catalog/product_credit_offer'),
    'erp_id'
);
```

## добалвение индексов

```php
<?
$installer->getConnection()->addIndex(
        $installer->getTable('oggetto_catalog/brand_dealer'),
        $installer->getIdxName('oggetto_catalog/brand_dealer', ['brand_id', 'dealer_id']),
        ['brand_id', 'dealer_id'],
        Varien_Db_Adapter_Interface::INDEX_TYPE_UNIQUE
    );
```

## таблица с foreign key

```php
<?
$table = $installer->getConnection()
    ->newTable($installer->getTable('oggetto_catalog/brand_dealer'))
    ->addColumn('id', Varien_Db_Ddl_Table::TYPE_INTEGER, null, [
        'unsigned' => true,
        'identity' => true,
        'nullable' => false,
        'primary'  => true,
    ], 'Link ID')
    ->addColumn('brand_id', Varien_Db_Ddl_Table::TYPE_INTEGER, null, array(
        'unsigned' => true,
        'nullable' => false,
    ), 'Brand ID')
    ->addColumn('dealer_id', Varien_Db_Ddl_Table::TYPE_INTEGER, null, array(
        'unsigned' => true,
        'nullable' => false,
    ), 'Dealer center ID')
    ->addForeignKey(
        $installer->getFkName('oggetto_catalog/brand_dealer', 'brand_id', 'oggetto_catalog/brand', 'brand_id'),
        'brand_id',
        $installer->getTable('oggetto_catalog/brand'),
        'brand_id', Varien_Db_Ddl_Table::ACTION_CASCADE, Varien_Db_Ddl_Table::ACTION_CASCADE
    )
    ->addForeignKey(
        $installer->getFkName('oggetto_catalog/brand_dealer', 'dealer_id', 'dealer/center', 'center_id'),
        'dealer_id',
        $installer->getTable('dealer/center'),
        'center_id', Varien_Db_Ddl_Table::ACTION_CASCADE, Varien_Db_Ddl_Table::ACTION_CASCADE
    )
    ->setComment('Brand-dealer links');
$installer->getConnection()->createTable($table);
```
