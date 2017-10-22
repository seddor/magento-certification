# Describe how to customize, extend, and troubleshoot the Enterprise Edition reward point system

Magento Reward Point System позволяет создавать бонусные программы для пользователей.
Реализовано в модуле `Enterprise_Reward`.  

## How do the features offered by the reward point system hook into other Magento modules?

Добавление поинтов происходит по ивентам,

  - newsletter_subscriber_save_commit_after
  - enterprise_invitation_save_commit_after
  - adminhtml_customer_save_after
  - customer_save_after
  - review_save_commit_after
  - tag_save_commit_after
  - sales_order_save_after
  - sales_model_service_quote_submit_before
  - sales_order_creditmemo_save_after
  - sales_order_invoice_save_commit_after
  - order_cancel_after
  - checkout_multishipping_refund_all
  - checkout_type_multishipping_create_orders_single
  - sales_model_service_quote_submit_failure

Также добавляет новые поля в фиелдсет при конвертации квоты в заказ

```xml
<fieldsets>
    <sales_convert_quote_address>
        <reward_points_balance>
            <to_order>*</to_order>
        </reward_points_balance>
        <reward_currency_amount>
            <to_order>*</to_order>
        </reward_currency_amount>
        <base_reward_currency_amount>
            <to_order>*</to_order>
        </base_reward_currency_amount>
    </sales_convert_quote_address>
</fieldsets>
```

И добавляет новый тотал для квот, инвойсов и кредитмемо

```xml
<quote>
    <totals>
        <reward>
            <class>enterprise_reward/total_quote_reward</class>
            <after>weee,discount,tax,tax_subtotal,grand_total</after>
            <before>giftcardaccount,customerbalance</before>
            <renderer>enterprise_reward/checkout_total</renderer>
        </reward>
    </totals>
</quote>
<order_invoice>
    <totals>
        <reward>
            <class>enterprise_reward/total_invoice_reward</class>
            <after>grand_total</after>
            <before>giftcardaccount,customerbalance</before>
        </reward>
    </totals>
</order_invoice>
<order_creditmemo>
    <totals>
        <reward>
            <class>enterprise_reward/total_creditmemo_reward</class>
            <after>weee,discount,tax,grand_total,customerbalance,giftcardaccount</after>
        </reward>
    </totals>
</order_creditmemo>
```


## Логика работы  

Для каджого действия существует свой класс, который реализует логику для проверки можно ли добавлять поинты, и сколько. Все классы действия наследуются от `Enterprise_Reward_Model_Action_Abstract`.

Поинты могут быть использованы в качестве оплаты. Рейты для поинтов настраиваются через админку и могут быть разными для групп.

Типичный сценарий изменения баланса поинтов:

  1. Отлавливается определённый ивент
  2. В обзёрвере примерно такой код:

  ```php
  <?
  Mage::getModel('enterprise_reward/reward')
      ->setCustomerId($order->getCustomer()->getId())
      ->setWebsiteId(Mage::app()->getStore($order->getStoreId())->getWebsiteId())
      ->setPointsDelta($order->getRewardPointsBalance())
      ->setAction(Enterprise_Reward_Model_Reward::REWARD_ACTION_REVERT)
      ->setActionEntity($order)
      ->updateRewardPoints();
  ```

  3. Метод **updateRewardPoints()**

  ```php
<?
public function updateRewardPoints()
{
    $this->_rewardPointsUpdated = false;
    if ($this->canUpdateRewardPoints()) {
        try {
            $this->save();
            $this->_rewardPointsUpdated = true;
        } catch (Exception $e) {
            $this->_rewardPointsUpdated = false;
            throw $e;
        }
    }
    return $this;
}

protected function _beforeSave()
{
    $this->loadByCustomer()
        ->_preparePointsDelta()
        ->_preparePointsBalance();
    return parent::_beforeSave();
}

protected function _preparePointsDelta()
{
    $delta = 0;
    $action = $this->getActionInstance($this->getAction());
    if ($action !== null) {
        $delta = $action->getPoints($this->getWebsiteId());
    }
    if ($delta) {
        if ($this->hasPointsDelta()) {
            $delta = $delta + $this->getPointsDelta();
        }
        $this->setPointsDelta((int)$delta);
    }
    return $this;
}
  ```


## Under which conditions may reward points be assigned?

Поинты назначаются или забираются за определенные действия:

  - Регистрацию
  - подписку
  - создание заказа
  - отмена заказа
  - возврат заказа
  - написание отзыва
  - изменены вручную через админку

## Which steps are required to add new custom options to assign reward points?

1. Нужно содзать свой класс, который будет наследоваться от `Enterprise_Reward_Model_Reward`, в нём нужно в статичный массив добавить новый тип действия для назначения/удаления поинтов
2. Класс-действия должен наследоваться от `Enterprise_Reward_Model_Action_Abstract`
3. Создать метод в котором будет обновляться количество поинтов(например по какому-то ивенту)
