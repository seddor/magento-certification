# Describe the Payment Bridge

Предназначен для совместимости с PCI — стандартом безопасности данных платёжных карт.Модуль отвечающий за это — `Enterprise_Pbridge`

По сути модуль предоставляет надстройку над методами оплаты, которая реализует методы оплаты с помощью карт посредством некого АПИ.

Методы оплаты проходящие через Payment Bridge наследуюются от абстрактного класса `Enterprise_Pbridge_Model_Payment_Method_Abstract`, который наследует `Mage_Payment_Model_Method_Cc`. Многние действия при оплате сводятся к вызову соответвующих методов у класс `Enterprise_Pbridge_Model_Payment_Method_Pbridge` наследуемого от `Mage_Payment_Model_Method_Abstract` в котором действия

  - authorize
  - capture
  - refund
  - void
  - acceptPayment
  - denyPayment
  - fetchTransactionInfo

сводятся к обращению к АПИ
