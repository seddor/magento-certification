# Explain different mechanisms for disabling block output

## In which ways can block output be disabled in Magento?

Можно удалять блоки через лаяут

```
<remove name="name" />
```

```
<reference name="parent">
  <action method="unsetChild">
      <name>name</name>
  </action>
</reference>
```

помиомо этого у блока есть метод unsetCallChild,
```
/**
 * Call a child and unset it, if callback matched result
 *
 * $params will pass to child callback
 * $params may be array, if called from layout with elements with same name, for example:
 * ...<foo>value_1</foo><foo>value_2</foo><foo>value_3</foo>
 *
 * Or, if called like this:
 * ...<foo>value_1</foo><bar>value_2</bar><baz>value_3</baz>
 * - then it will be $params1, $params2, $params3
 *
 * It is no difference anyway, because they will be transformed in appropriate way.
 *
 * @param string $alias
 * @param string $callback
 * @param mixed $result
 * @param array $params
 * @return Mage_Core_Block_Abstract
 */
public function unsetCallChild($alias, $callback, $result, $params)
{
    $child = $this->getChild($alias);
    if ($child) {
        $args = func_get_args();
        $alias = array_shift($args);
        $callback = array_shift($args);
        $result = (string)array_shift($args);
        if (!is_array($params)) {
            $params = $args;
        }

        if ($result == call_user_func_array(array(&$child, $callback), $params)) {
            $this->unsetChild($alias);
        }
    }
    return $this;
}
```

## Which method can be overridden to control block output?

Можно переопределить методы `\_toHtml()` или `\_afterToHtml()`
