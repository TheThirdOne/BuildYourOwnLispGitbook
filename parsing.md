Parsing
=======

![polish man](img/polish_man.png "A Polish Nobleman &bull; A typical Polish Notation user")

Polish Notation
---------------

To try out `mpc` we're going to implement a simple grammar that resembles a mathematical subset of our Lisp. It's called [Polish Notation](http://en.wikipedia.org/wiki/Polish_notation) and is a notation for arithmetic where the operator comes before the operands.

For example...
<table class='table' style='display: inline'>
  <tr><td>`1 + 2 + 6`</td><td>*is*</td><td>`+ 1 2 6`</td></tr>
  <tr><td>`6 + (2 * 9)`</td><td>*is*</td><td>`+ 6 (* 2 9)`</td></tr>
  <tr><td>`(10 * 2) / (4 + 2)`</td><td>*is*</td><td>`/ (* 10 2) (+ 4 2)`</td></tr>
</table>

We need to work out a grammar which describes this notation. We can begin by describing it *textually* and then later formalise our thoughts.

To start, we observe that in polish notation the operator always comes first in an expression, followed by either numbers or other expressions in parenthesis. This means we can say "a *program* is an *operator* followed by one or more *expressions*," where "an *expression* is either a *number*, or, in parenthesis, an *operator* followed by one or more *expressions*".

More formally...

<table class='table'>
  <tr><td>`Program`</td><td>*the start of input*, an `Operator`, one or more `Expression`, and *the end of input*.</td></tr>
  <tr><td>`Expression`</td><td>either a `Number` *or* `'('`, an `Operator`, one or more `Expression`, and an `')'`.</td></tr>
  <tr><td>`Operator`</td><td>`'+'`, `'-'`, `'*'`, or `'/'`.</td></tr>
  <tr><td>`Number`</td><td>an optional `-`, and one or more characters between `0` and `9`</td></tr>
</table>


Regular Expressions
-------------------

We should be able to encode most of the above rules using things we know already, but *Number* and *Program* might pose some trouble. They contain a couple of constructs we've not learnt how to express yet. We don't know how to express the start or the end of input, optional characters, or range of characters.

These can be expressed, but they require something called a *Regular Expression*. Regular expressions are a way of writing grammars for small sections of text such as words or numbers. Grammars written using regular expressions can't consist of multiple rules, but they do give precise, and concise, control over what is matched and what isn't. Here are some basic rules for writing regular expressions.

<table class='table'>
  <tr><td>`a`</td><td>The character `a` is required.</td></tr>
  <tr><td>`[abcdef]`</td><td>Any character in the set `abcdef` is required.</td></tr>
  <tr><td>`[a-f]`</td><td>Any character in the range `a` to `f` is required.</td></tr>
  <tr><td>`a?`</td><td>The character `a` is optional.</td></tr>
  <tr><td>`a*`</td><td>Zero or more of the character `a` are required.</td></tr>
  <tr><td>`a+`</td><td>One or more of the character `a` are required.</td></tr>
  <tr><td>`^`</td><td>The start of input is required.</td></tr>
  <tr><td>`$`</td><td>The end of input is required.</td></tr>
</table>

These are all the regular expression rules we need for now. [Whole books](http://regex.learncodethehardway.org/) have been written on learning regular expressions. For those curious much more information can be found online or from these sources. We will be using them in later chapters, so some basic knowledge will be required, but you won't need to master them for now.

In an `mpc` grammar we write regular expressions by putting them between forward slashes `/`. Using the above guide our *Number* rule can be expressed as a regular expression using the string `/-?[0-9]+/`.


Installing mpc
--------------

Before we work on writing this grammar we first need to *include* the `mpc` headers, and then *link* to the `mpc` library, just as we did for `editline` on Linux and Mac. Starting with your code from chapter 4, you can rename the file to `parsing.c` and download `mpc.h` and `mpc.c` from the [mpc repo](http://github.com/orangeduck/mpc). Put these in the same directory as your source file.

To *include* `mpc` put `#include "mpc.h"` at the top of the file. To *link* to `mpc` put `mpc.c` directly into the compile command. On linux you will also have to link to the maths library by adding the flag `-lm`.

On **Linux** and **Mac**

```
cc -std=c99 -Wall parsing.c mpc.c -ledit -lm -o parsing
```

On **Windows**

```
cc -std=c99 -Wall parsing.c mpc.c -o parsing
```

<div class="alert alert-warning">
  **Hold on, don't you mean `#include <mpc.h>`?**

  There are actually two ways to include files in C. One is using angular brackets `>` as we've seen so far, and the other is with quotation marks `""`.

  The only difference between the two is that using angular brackets searches the system locations for headers first, while quotation marks searches the current directory first. Because of this system headers such as `<stdio.h>` are typically put in angular brackets, while local headers such as `"mpc.h"` are typically put in quotation marks.
</div>


Polish Notation Grammar
-----------------------

Formalising the above rules further, and using some regular expressions, we can write a final grammar for the language of polish notation as follows. Read the below code and verify that it matches what we had written textually, and our ideas of polish notation.

```c
/* Create Some Parsers */
mpc_parser_t* Number   = mpc_new("number");
mpc_parser_t* Operator = mpc_new("operator");
mpc_parser_t* Expr     = mpc_new("expr");
mpc_parser_t* Lispy    = mpc_new("lispy");

/* Define them with the following Language */
mpca_lang(MPC_LANG_DEFAULT,
  "                                                     \
    number   : /-?[0-9]+/ ;                             \
    operator : '+' | '-' | '*' | '/' ;                  \
    expr     : <number> | '(' <operator> <expr>+ ')' ;  \
    lispy    : /^/ operator> expr>+ /$/ ;             \
  ",
  Number, Operator, Expr, Lispy);
```

We need to add this to the interactive prompt we started on in chapter 4. Put this code right at the beginning of the `main` function before we print the *Version* and *Exit* information. At the end of our program we also need to delete the parsers when we are done with them. Right before `main` returns we should place the following clean-up code.

```c
/* Undefine and Delete our Parsers */
mpc_cleanup(4, Number, Operator, Expr, Lispy);
```

<div class="alert alert-warning">
  **I'm getting an error `undefined reference to `mpc_lang'`**

  That should be `mpca_lang`, with an `a` at the end!
</div>

Parsing User Input
------------------

Our new code creates a `mpc` parser for our *Polish Notation* language, but we still need to actually *use* it on the user input supplied each time from the prompt. We need to edit our `while` loop so that rather than just echoing user input back, it actually attempts to parse the input using our parser. We can do this by replacing the call to `printf` with the following `mpc` code, that makes use of our program parser `Lispy`.

```c
/* Attempt to Parse the user Input */
mpc_result_t r;
if (mpc_parse("stdin>";, input, Lispy, &;r)) {
  /* On Success Print the AST */
  mpc_ast_print(r.output);
  mpc_ast_delete(r.output);
} else {
  /* Otherwise Print the Error */
  mpc_err_print(r.error);
  mpc_err_delete(r.error);
}
```

This code calls the `mpc_parse` function with our parser `Lispy`, and the input string `input`. It copies the result of the parse into `r` and returns `1` on success and `0` on failure. We use the address of operator `&` on `r` when we pass it to the function. This operator will be explained in more detail in later chapters.

On success a internal structure is copied into `r`, in the field `output`. We can print out this structure using `mpc_ast_print` and delete it using `mpc_ast_delete`.

Otherwise there has been an error, which is copied into `r` in the field `error`. We can print it out using `mpc_err_print` and delete it using `mpc_err_delete`.

Compile these updates, and take this program for a spin. Try out different inputs and see how the system reacts. Correct behaviour should look like the following.

```lispy
Lispy Version 0.0.0.0.2
Press Ctrl+c to Exit

lispy> + 5 (* 2 2)
>:
  regex:
  operator|char: '+'
  expr|number|regex: '5'
  expr|>:
    char: '('
    operator|char: '*'
    expr|number|regex: '2'
    expr|number|regex: '2'
    char: ')'
  regex:
lispy> hello
stdin>:0:0: error: expected '+', '-', '*' or '/' at 'h'
lispy> / 1dog & cat
stdin>:0:3: error: expected end of input at 'd'
lispy>
```

<div class="alert alert-warning">
  **I'm getting an error `stdin>:0:0: error: Parser Undefined!`.**

  This error is due to the syntax for your grammar supplied to `mpca_lang` being incorrect. See if you can work out what part of the grammar is incorrect. You can use the reference code for this chapter to help you find this, and verify how the grammar should look.
</div>


Reference
---------

<div class="panel-group alert alert-warning" id="accordion">
  <a href="parsing.c">parsing.c</a>
</div>


Bonus Marks
-----------

<div class="alert alert-warning">
  <ul class="list-group">
    <li class="list-group-item">&rsaquo; Write a regular expression matching strings of all `a` or `b` such as `aababa` or `bbaa`.</li>
    <li class="list-group-item">&rsaquo; Write a regular expression matching strings of consecutive `a` and `b` such as `ababab` or `aba`.</li>
    <li class="list-group-item">&rsaquo; Write a regular expression matching `pit`, `pot` and `respite` but *not* `peat`, `spit`, or `part`.</li>
    <li class="list-group-item">&rsaquo; Change the grammar to add a new operator such as `%`.</li>
    <li class="list-group-item">&rsaquo; Change the grammar to recognize operators written in textual format `add`, `sub`, `mul`, `div`.</li>
    <li class="list-group-item">&rsaquo; Change the grammar to recognize decimal numbers such as `0.01`, `5.21`, or `10.2`.</li>
    <li class="list-group-item">&rsaquo; Change the grammar to make the operators written conventionally, between two expressions.</li>
    <li class="list-group-item">&rsaquo; Use the grammar from the previous chapter to parse `Doge`. You must add *start* and *end* of input!</li>
  </ul>
</div>
