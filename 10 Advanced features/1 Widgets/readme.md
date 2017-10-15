# Create frontend widgets and describe widget architecture

Виджеты в мадженте это конфигурируемые блоки, которые можно добавлять на страницы через админку.
Виджеты могут добавлятся на CMS страницу через редактор или руками:

```
{{widget type="foo_widget/featured_video" title="My Featured Video" youtube="https://www.youtube.com/embed/dQw4w9WgXcQ" template="foo/widget/large.phtml"}}

```

Или виджеты можно доставить через специальное меню Widget


## Создание кастомного виджета

### Widget.xml

Конфиги виджета хранятся в собственных конфигурационных файлах `widget.xml`

 ```xml
<widgets>
  <foo_youtube type="foo_widget/youtube">
    <name>YouTube Example Widget</name>
    <description>
      This wiget displays a YouTube video.
    </description>
    <parameters>
      <video_id>
        <required>1</required>
        <visible>1</visible>
        <value>Enter ID Here</value>
        <label>YouTube Video ID</label>
        <type>text</type>
      </video_id>
    </parameters>
  </foo_youtube>
</widget>
```

 - в type пишется элиас к блоку виджета
 - В ноде parameters пишутся параметры виджета, они могут быть видмимыми и доступны для настройки из админки при создании экземпляра виджета

### Класс блока видтжета

```php
<?
class Foo_Widget_Block_Youtube extends Mage_Core_Block_Template
    implements Mage_Widget_Block_Interface
{

}
```

 Блок должен имплеменить интерфейс `Mage_Widget_Block_Interface`

 ```php
 <?
 interface Mage_Widget_Block_Interface
{
    /**
     * Produce and return widget's html output
     *
     * @return string
     */
    public function toHtml();

    /**
     * Add data to the widget.
     * Retains previous data in the widget.
     *
     * @param array $arr
     * @return Mage_Widget_Block_Interface
     */
    public function addData(array $arr);

    /**
     * Overwrite data in the widget.
     *
     * $key can be string or array.
     * If $key is string, the attribute value will be overwritten by $value.
     * If $key is an array, it will overwrite all the data in the widget.
     *
     * @param string|array $key
     * @param mixed $value
     * @return Varien_Object
     */
    public function setData($key, $value = null);
}
```

Далее нужно или реализовать метод `toHtml` или использовать `toHeml` из готовых маджентовских блоков унаследовавшись от них.
Сам виджет можно будет добавить в админке

### Параметры виджета

Параметры виджета имеют следующие поля

  - visible
  - required
  - label
  - type
  - value — здесь можно задать дефолтные значения для параметров
  - values — значения, которые могут быть выбраны в параметре

```xml
<values>
    <large_video translate="label">
        <label>Large Video</label>
        <value>colin_widget/large.phtml</value>
    </large_video>
    <small_video translate="label">
        <label>Small Video</label>
        <value>colin_widget/small.phtml</value>
    </small_video>
</values>

```
  - sort_order
  - helper_block — блок для рендринга параметра, например:

  ```xml
  <helper_block>
      <type>adminhtml/cms_page_widget_chooser</type>
      <data>
          <button translate="open">
              <open>Select Page...</open>
          </button>
      </data>
  </helper_block>
  ```

### Ограничения для виджетов

При добалении блока через CMS-страницу можно ограничить его отображение в определёных частях страницы

Например, ограничить чтобы виджет отображался только в разделе content(с выбраном темлейтом large_video) и в right(для small_video)

```xml
<supported_blocks>
    <colin_widget_content>
        <block_name>content</block_name>
        <template>
            <default>large_video</default>
        </template>
    </colin_widget_content>
    <colin_widget_sidebar>
        <block_name>right</block_name>
        <template>
            <default>small_video</default>
        </template>
    </colin_widget_sidebar>
</supported_blocks>
```


## What classes are typically involved in Magento widget architecture?

Интерфейс `Mage_Widget_Block_Interface`. Большинстао виджетов наследуется от классов `Mage_Core_Block_Abstract`

## How are admin-configurable parameters and their options specified?

Через ноду `parameters`
