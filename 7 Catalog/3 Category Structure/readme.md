# Describe the Category Hierarchy Tree Structure implementation

Describe the Category Hierarchy Tree Structure implementation (the internal
structure inside the database), including:
  * The meaning of parent_id 0 — базовая категория для всех корневых категория
  * The construction of paths — например для категории 43 дочерней категории 32 и корневой категорий 2 путь будет, а базовой 1 = 1/2/32/43
  * The attributes required to display a new category in the store — на форме создания категории обязательны поля:
    * name
    * is active
    * Include in Navigation Menu
    * Available Product Listing Sort By
    * Default Product Listing Sort By


В таблице `catalog_category_entity` есть поле `path` — содержащее полный путь категории с ид её родительских категорий и ИД текущей категории(от корня до текущей категории). С помощью поля `path` определяется место категории в иерахии категорий. Для работы с деревом категорий в мадженте предусмотрен класс `Mage_Catalog_Model_Resource_Category_Tree` который используется, в частности, для построение деревовидной иерахии категории


## How is the category hierarchy reflected in the database? Does it differ when multiple root categories are present?

С помощью поля `path` в `catalog_category_entity`. Структура всегда начинается с базовой категории(с parent_id = 0).

## How is a catalog tree read from the database tables, with and without flat catalog tables enabled?

При использовании flat таблиц запросы будут производится к таблице `catalog_category_flat_store_{{store_id}}`, в которой помимо базовых данных категории и её статичных аттрибутов будут содержатся так же некоторые заиндексированые данные аттрибутов(например, имя категории, её урл-кей и т.д.)

## How does working with categories differ if the flat catalog is enabled on a model level?

При использовании flat в качестве ресурсной модели используется `catalog/category_flat` (Mage_Catalog_Model_Resource_Category_Flat) в качестве таблицы используется `catalog_category_flat_store_{{store_id}}`

## How is the category parent id path set on new categories?

При сохранении категории в админке у модели меняется `path`.
```php
<?    /**
     * Process category data before saving
     * prepare path and increment children count for parent categories
     *
     * @param Varien_Object $object
     * @return Mage_Catalog_Model_Resource_Category
     */
    protected function _beforeSave(Varien_Object $object)
    {
        parent::_beforeSave($object);

        if (!$object->getChildrenCount()) {
            $object->setChildrenCount(0);
        }
        if ($object->getLevel() === null) {
            $object->setLevel(1);
        }

        if (!$object->getId()) {
            $object->setPosition($this->_getMaxPosition($object->getPath()) + 1);
            $path  = explode('/', $object->getPath());
            $level = count($path);
            $object->setLevel($level);
            if ($level) {
                $object->setParentId($path[$level - 1]);
            }
            $object->setPath($object->getPath() . '/');

            $toUpdateChild = explode('/',$object->getPath());

            $this->_getWriteAdapter()->update(
                $this->getEntityTable(),
                array('children_count'  => new Zend_Db_Expr('children_count+1')),
                array('entity_id IN(?)' => $toUpdateChild)
            );

        }
        return $this;
    }

    /**
     * Process category data after save category object
     * save related products ids and update path value
     *
     * @param Varien_Object $object
     * @return Mage_Catalog_Model_Resource_Category
     */
    protected function _afterSave(Varien_Object $object)
    {
        /**
         * Add identifier for new category
         */
        if (substr($object->getPath(), -1) == '/') {
            $object->setPath($object->getPath() . $object->getId());
            $this->_savePath($object);
        }

        $this->_saveCategoryProducts($object);
        return parent::_afterSave($object);
    }
```    

## Which methods exist to read category children and how do they differ?

`Mage_Catalog_Model_Category` имеет следующие методы:

  * `getChildren()` — возвращает идишники детей

```php
<?
/**
 * Retrieve children ids comma separated
 *
 * @return string
 */
public function getChildren()
{
    return implode(',', $this->getResource()->getChildren($this, false));
}

//ресурс
/**
 * Return children ids of category
 *
 * @param Mage_Catalog_Model_Category $category
 * @param boolean $recursive
 * @return array
 */
public function getChildren($category, $recursive = true)
{
    $attributeId  = (int)$this->_getIsActiveAttributeId();
    $backendTable = $this->getTable(array($this->getEntityTablePrefix(), 'int'));
    $adapter      = $this->_getReadAdapter();
    $checkSql     = $adapter->getCheckSql('c.value_id > 0', 'c.value', 'd.value');
    $bind = array(
        'attribute_id' => $attributeId,
        'store_id'     => $category->getStoreId(),
        'scope'        => 1,
    );
    $select = $this->_getChildrenIdSelect($category, $recursive);
    $select
        ->joinLeft(
            array('d' => $backendTable),
            'd.attribute_id = :attribute_id AND d.store_id = 0 AND d.entity_id = m.entity_id',
            array()
        )
        ->joinLeft(
            array('c' => $backendTable),
            'c.attribute_id = :attribute_id AND c.store_id = :store_id AND c.entity_id = m.entity_id',
            array()
        )
        ->where($checkSql . ' = :scope');

    return $adapter->fetchCol($select, $bind);
}
```

  * `getAllChildren($asArray = false)` — возвращает идишники всех детей

```php
<?

    /**
     * Get all children categories IDs
     *
     * @param boolean $asArray return result as array instead of comma-separated list of IDs
     * @return array|string
     */
    public function getAllChildren($asArray = false)
    {
        $children = $this->getResource()->getAllChildren($this);
        if ($asArray) {
            return $children;
        }
        else {
            return implode(',', $children);
        }
    }

//ресурс
/**
 * Return all children ids of category (with category id)
 *
 * @param Mage_Catalog_Model_Category $category
 * @return array
 */
public function getAllChildren($category)
{
    $children = $this->getChildren($category);
    $myId = array($category->getId());
    $children = array_merge($myId, $children);

    return $children;
}
```

  * `getChildrenCategories()` — возвращает загруженную коллекцию включеных детей.

```php
<?
public function getChildrenCategories()
{
    return $this->getResource()->getChildrenCategories($this);
}

//ресурс

public function getChildrenCategories($category)
{
    $collection = $this->_getChildrenCategoriesBase($category);
    $collection->addAttributeToFilter('is_active', 1)
        ->addIdFilter($category->getChildren())
        ->load();

    return $collection;
}
```

  * `getChildrenCategoriesWithInactive()` — возвращает не загруженную коллекцию детей категории(включая выключеные)

```php
<?
public function getChildrenCategoriesWithInactive()
{
    return $this->getResource()->getChildrenCategoriesWithInactive($this);
}

//ресурс

public function getChildrenCategoriesWithInactive($category)
{
    $collection = $this->_getChildrenCategoriesBase($category);
    $collection->addFieldToFilter('parent_id', $category->getId());

    return $collection;
}
```
