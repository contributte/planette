---
date: "2013-01-30"
draft: false
title: "Macro loader"
tags: ["latte", "template"]
type: "blog"
slug: "macro-loader"
author: "Martin Zlámal"
---

This short tutorial will show you, how to create and register custom macros.

Built on **Nette 2.0.8**.


## Create macro

First, we create macroSet and add all our macros. How to register macros to template will be shown later.

Note: in Nette 2.1 and older, you have to use Nette\Latte namespace instead of Latte.

```php
$set = Latte\Macros\MacroSet::install($latte->compiler);
$set->addMacro('id', NULL, NULL, 'if ($_l->tmp = array_filter(%node.array)) echo \' id="\' . %escape(implode(" ", array_unique($_l->tmp))) . \'"\'');
```

Use case in template:

```html
<div n:id="TRUE ? success : error">Id success</div>
<div n:id="FALSE ? success : error">Id error</div>
```


### Syntax

Macro has few [basic parameters](https://api.nette.org/2.0/Nette.Latte.Macros.MacroSet.html#_addMacro)

```php
addMacro($name, $begin, $end = NULL, $attr = NULL);
```

Parameters `$begin` and `$end` are used to create classic macro, parameter `$attr` is used for n:macro.

For deeper knowledge about those parameters you can check [classic vs n:macro](#toc-classic-vs-n-macroů) section.

This should seem more clear now:
```php
$set->addMacro('id', NULL, NULL,
  'if ($_l->tmp = array_filter(%node.array))
     echo \' id="\' . %escape(implode(" ", array_unique($_l->tmp))) . \'"\'
');
```

We can use special aliases like `%node.word`, `%node.array`, `%node.args`, `%escape()`, `%modify()`, `%var` and `%raw` while writing content code of macro. To more see [documentation](doc:en/templating#toc-user-defined-macros). For example, when we write in macro `%escape($_template->var)`, it will have same result as `{$var}` in template.

For better understanding, you can check some examples in Nette:
[CacheMacro](https://api.nette.org/2.0/source-Latte.Macros.CacheMacro.php.html#19), [CoreMacros](https://api.nette.org/2.0/source-Latte.Macros.CoreMacros.php.html#22) a [FormMacros](https://api.nette.org/2.0/source-Latte.Macros.FormMacros.php.html#24)


### `$attr` as callback

In case of long method you can register callback to standalone method.

```php

$set->addMacro('id', NULL, NULL, array($this, 'macroId'));

// ...


/**
 * n:id="..."
 */
public function macroId(MacroNode $node, PhpWriter $writer)
{
    return $writer->write('if ($_l->tmp = array_filter(%node.array)) echo \' id="\' . %escape(implode(" ", array_unique($_l->tmp))) . \'"\'');
}

```



## Registration

As mentioned in the first part, after creating macroSet, we have have to register it. Only after registration macros can be used in template.

First we create separate class and place it o `libs` folder.

**CustomMacros.php**

```php
use Latte\MacroNode;
use Latte\PhpWriter;

class CustomMacros extends Latte\Macros\MacroSet
{

    public static function install(Latte\Compiler $compiler)
    {
        $set = new static($compiler);
        $set->addMacro('id', NULL, NULL, array($set, 'macroId'));
    }


    /**
     * n:id="..."
     */
    public function macroId(MacroNode $node, PhpWriter $writer)
    {
        return $writer->write('if ($_l->tmp = array_filter(%node.array)) echo \' id="\' . %escape(implode(" ", array_unique($_l->tmp))) . \'"\'');
    }

}
```

Then we add our class to latte compilation in `config.neon`

```yaml
factories:
    nette.latte:
        class: Latte\Engine
        setup:
            - CustomMacros::install(::$service->getCompiler())
```yaml

Since **Nette 2.1-dev** there exists special section for macros:

```yaml
nette:
    latte:
        macros:
            - CustomMacros::install
```


## Classic macro vs. n:macro

```php
addMacro($name, $begin, $end = NULL, $attr = NULL);
```

Using only *$begin* and *$end* parameters we create classic macro.

Using only *$attr* parameter we create n:macro.

To combine both in universal macro, we have to register anonymous function or define all parameters.

### Use case

**Presenter**

```php
# obě makra (všechny parametry)
$set->addMacro("test1", "start", "end", "attributes");

# obě makra (anonymní funkce, callbacks)
$set->addMacro("test1", function($node, $writer) { ... });
$set->addMacro("test1", callback($this, 'myMacro');
$set->addMacro("test1", array('Macros', 'myMacro'));

# pouze n:makro
$set->addMacro("test2", NULL, NULL, "attributes");

# pouze klasické makro
$set->addMacro("test3", "start", "end");

```

**Template**

```html
{test1 "parameter1"}Lorem lipsum..{/test1}

<div n:test1="parameter1">Lorem lipsum..</div>

<span n:test2="parameter1">Lorem lipsum..</span>

{test3 "parameter1"}Lorem lipsum..{/test3}
```

**Html**

```html
<?php start ?>Lorem lipsum..<?php end ?>

<div <?php atributy ?>>Lorem lipsum..</div>

<span <?php atributy?>>Lorem lipsum..</span>

<?php start ?>Lorem lipsum..<?php end ?>
```
