# Describe the collection interface (filtering/sorting/grouping)

## Фильтрация

  * addFieldToFilter — не EAV
  * addAttributeToFilter — EAV

Оба метода принимают два параметра — фильтруемое поле и условия фильтрации, условия могут быть переданы стройкой(проверка на эквивалентность) или массивом

### Простая фильтрация


    $collection->addFieldToFilter('home', array('eq' => '4'));
    $collection->addFieldToFilter('home', array('neq' => '4'));
    $collection->addFieldToFilter('home', array('gt' => '4'));
    $collection->addFieldToFilter('home', array('gt' => '4'));
    $collection->addFieldToFilter('home', array('like' => '4'));

### IN и not IN

    $collection->addFieldToFilter('home', array('in' => array(1, 2)));
    $collection->addFieldToFilter('home', array('nin' => array(1, 2)));
    //SELECT `main_table`.* FROM `football_results` AS `main_table` WHERE (home IN(1, 2))

### From/To

    $collection->addFieldToFilter('home', array('from' => 1, 'to' => 2));
    //SELECT `main_table`.* FROM `football_results` AS `main_table` WHERE (home >= 1 AND home <= 2)


### Attribute Query

    $collection =  Mage::getModel('catalog/product')->getCollection();
    $collection->addAttributeToFilter("status", 1);

### OR Query


    $collection->addFieldToFilter(
        array('home', 'away'),
        array(4, 0)
    );
    var_dump((string)$collection->getSelect());
    // SELECT `main_table`.* FROM `football_results` AS `main_table` WHERE ((home = '4') OR (away = '0')


### AND Query

      $collection->addFieldToFilter('home', 4);
      $collection->addFieldToFilter('away', 0);
      var_dump((string)$collection->getSelect());
      // SELECT `main_table`.* FROM `football_results` AS `main_table` WHERE ((home = '4') AND (away = '0')


## Sorting

There are 2 methods which both call *_setOrder* method

1. addOrder
2. setOrder


    $collection =  Mage::getModel('colin_database/results')->getCollection();
    $collection->addOrder('home', Zend_Db_Select::SQL_ASC);
    $collection->setOrder('home', Zend_Db_Select::SQL_ASC);


We can also limit in 2 ways

    $collection->setCurPage(1)->setPageSize(5);
    $collection->getSelect()->limit(5);


## Подробнее про коллекции

https://magento2.atlassian.net/wiki/spaces/m1wiki/pages/14024829/Using+Magento+1.x+collections#app-switcher
