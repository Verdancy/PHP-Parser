PHP Parser
==========


This is a PHP 5.2 to PHP 8.1 parser written in PHP. Its purpose is to simplify static code analysis and
manipulation.



Features
--------

The main features provided by this library are:

 * Parsing PHP 5, PHP 7, and PHP 8 code into an abstract syntax tree (AST).
   * Invalid code can be parsed into a partial AST.
   * The AST contains accurate location information.
 * Dumping the AST in human-readable form.
 * Converting an AST back to PHP code.
   * Experimental: Formatting can be preserved for partially changed ASTs.
 * Infrastructure to traverse and modify ASTs.
 * Resolution of namespaced names.
 * Evaluation of constant expressions.
 * Builders to simplify AST construction for code generation.
 * Converting an AST into JSON and back.

Quick Start
-----------

Install the library using [composer](https://getcomposer.org):

    php composer.phar require nikic/php-parser

Parse some PHP code into an AST and dump the result in human-readable form:

```php
<?php
use PhpParser\Error;
use PhpParser\NodeDumper;
use PhpParser\ParserFactory;

$code = <<<'CODE'
<?php

function test($foo)
{
    var_dump($foo);
}
CODE;

$parser = (new ParserFactory)->create(ParserFactory::PREFER_PHP7);
try {
    $ast = $parser->parse($code);
} catch (Error $error) {
    echo "Parse error: {$error->getMessage()}\n";
    return;
}

$dumper = new NodeDumper;
echo $dumper->dump($ast) . "\n";
```

This dumps an AST looking something like this:

```
array(
    0: Stmt_Function(
        byRef: false
        name: Identifier(
            name: test
        )
        params: array(
            0: Param(
                type: null
                byRef: false
                variadic: false
                var: Expr_Variable(
                    name: foo
                )
                default: null
            )
        )
        returnType: null
        stmts: array(
            0: Stmt_Expression(
                expr: Expr_FuncCall(
                    name: Name(
                        parts: array(
                            0: var_dump
                        )
                    )
                    args: array(
                        0: Arg(
                            value: Expr_Variable(
                                name: foo
                            )
                            byRef: false
                            unpack: false
                        )
                    )
                )
            )
        )
    )
)
```

Let's traverse the AST and perform some kind of modification. For example, drop all function bodies:

```php
use PhpParser\Node;
use PhpParser\Node\Stmt\Function_;
use PhpParser\NodeTraverser;
use PhpParser\NodeVisitorAbstract;

$traverser = new NodeTraverser();
$traverser->addVisitor(new class extends NodeVisitorAbstract {
    public function enterNode(Node $node) {
        if ($node instanceof Function_) {
            // Clean out the function body
            $node->stmts = [];
        }
    }
});

$ast = $traverser->traverse($ast);
echo $dumper->dump($ast) . "\n";
```

This gives us an AST where the `Function_::$stmts` are empty:

```
array(
    0: Stmt_Function(
        byRef: false
        name: Identifier(
            name: test
        )
        params: array(
            0: Param(
                type: null
                byRef: false
                variadic: false
                var: Expr_Variable(
                    name: foo
                )
                default: null
            )
        )
        returnType: null
        stmts: array(
        )
    )
)
```

Finally, we can convert the new AST back to PHP code:

```php
use PhpParser\PrettyPrinter;

$prettyPrinter = new PrettyPrinter\Standard;
echo $prettyPrinter->prettyPrintFile($ast);
```

This gives us our original code, minus the `var_dump()` call inside the function:

```php
<?php

function test($foo)
{
}
```





