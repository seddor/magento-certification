# Describe the different Web Service APIs available within the Magento Core

## SOAP API

1. — v1, 2. — v2

**URL**

  1. {{base_url}}/api/?wsdl
  2. {{base_url}}/api/v2.soap?wsdl

**Методы**

  1. $client->call($session, "sales_order.list")
  2. $client->salesOrderList();

**handler**  

  1. Mage_Api_Model_Server_Handler
  2. Mage_Api_Model_Server_V2_Handler

**controller**

  1. Mage_Api_SoapController
  2. Mage_Api_V2_SoapController
