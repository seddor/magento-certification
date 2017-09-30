# Troubleshoot and customize Magento indexes

## Типа индексов

* Flat index — для продуктов и для категорий.

# Troubleshoot and customize Magento indexes

Check the following classes:

- Mage_Index_Model_Indexer_Abstract

Then depending on the catalog type:

- Mage_Catalog_Model_Product_indexer_Eav
- Mage_Catalog_Model_Product_Indexer_Flat

If its a price related issue

- Mage_Catalog_Model_Product_Indexer_Price

If its a stock issue

- Mage_CatalogInventory_Model_Indexer_Stock
