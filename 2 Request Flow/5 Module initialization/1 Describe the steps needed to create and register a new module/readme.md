# Describe the steps needed to create and register a new module

1. Создать файл определения модуля в app/etc/modules, например *Foo_Bar.xml*
```
<?xml version="1.0"?>
<config>
    <modules>
        <Foo_Bar>
            <active>true</active>
            <codePool>local</codePool>
            <depends>
              <Mage_Core />
            </depends>
        </Foo_Bar>
    </modules>
</config>
```
2. основные файлы модуля будут хранится в app/code/local/Foo/Bar
