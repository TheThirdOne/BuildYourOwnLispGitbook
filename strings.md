Strings
=======


Libraries
---------

![string](img/string.png "String &bull; How long is i")

Our Lisp is finally pretty functional. We should be able to write almost any functions we want. We can build some quite complex constructs using it, and even do some cool things that can't be done in lots of other heavyweight and popular languages!

Every time we update our program and run it again it is getting annoying having to type in again all of our functions. In this chapter we'll add the functionality to load code from a file and run it. This will allow us to start building a standard library up. Along the way we'll also add support for code comments, strings, and printing.


String Type
-----------

For the user to load a file we'll have to let them supply a string consisting of the file name. Our language supports symbols, but still doesn't support strings, which can include spaces and other characters. We need to add this possible `lval` type to specify the file names we need.

We start, as in other chapters, by adding an entry to our enum and adding an entry to our `lval` to represent the type's data.

```c
enum { LVAL_ERR, LVAL_NUM, LVAL_SYM, LVAL_STR, LVAL_FUN, LVAL_SEXPR, LVAL_QEXPR };
```

```c
/* Basic */
long num;
char* err;
char* sym;
char* str;

```

Next we can add a function for constructing string `lval`, very similar to how we construct constructing symbols.

```c
lval* lval_str(char* s) {
  lval* v = malloc(sizeof(lval));
  v->type = LVAL_STR;
  v->str = malloc(strlen(s) + 1);
  strcpy(v->str, s);
  return v;
}
```

We also need to add the relevant entries into our functions that deal with `lval`.

For **Deletion**...

```c
case LVAL_STR: free(v->str); break;
```

For **Copying**...

```c
case LVAL_STR: x->str = malloc(strlen(v->str) + 1); strcpy(x->str, v->str); break;
```

For **Equality**...

```c
case LVAL_STR: return (strcmp(x->str, y->str) == 0);
```

For **Type Name**...

```c
case LVAL_STR: return "String";
```

For **Printing** we need to do a little more. The string we store internally is different to the string we want to print. We want to print a string as a user might input it, using escape characters such as `\n` to represent a newline.

We therefore need to escape it before we print it. Luckily we can make use of a `mpc` function that will do this for us.

In the printing function we add the following...

```c
case LVAL_STR:   lval_print_str(v); break;
```

Where...

```c
void lval_print_str(lval* v) {
  /* Make a Copy of the string */
  char* escaped = malloc(strlen(v->str)+1);
  strcpy(escaped, v->str);
  /* Pass it through the escape function */
  escaped = mpcf_escape(escaped);
  /* Print it between " characters */
  printf("\"%s\"", escaped);
  /* free the copied string */
  free(escaped);
}
```


Reading Strings
---------------

Now we need to add support for parsing strings. As usual this requires first adding a new grammar rule called `string` and add it to our parser.

The rule we are going to use that represents a string is going to be the same as for C style strings. This means a string is essentially a series of escape characters, or normal characters, between two quotation marks `""`. We can specify this as a regular expression inside our grammar string as follows.

```
string  : /\"(\\\\.|[^\"])*\"/ ;
```

This looks pretty complicated but makes a lot more sense when explained in parts. It reads like this. A string is a `"` character, followed by zero or more of either a backslash `\\` followed by any other character `.`, or anything that *isn't** a `"` character `[^\\"]`. Finally it ends with another `"` character.

We also need to add a case to deal with this in the `lval_read` function.

```c
if (strstr(t->tag, "string")) { return lval_read_str(t); }
```

Because the input string is input in an escaped form we need to create a function `lval_read_str` which deals with this. This function is a little tricky because it has to do a few tasks. First it must strip the input string of the `"` characters on either side. Then it must unescape the string, converting series of characters such as `\n` to their actual encoded characters. Finally it has to create a new `lval` and clean up anything that has happened in-between.

```c
lval* lval_read_str(mpc_ast_t* t) {
  /* Cut off the final quote character */
  t->contents[strlen(t->contents)-1] = '\0';
  /* Copy the string missing out the first quote character */
  char* unescaped = malloc(strlen(t->contents+1));
  strcpy(unescaped, t->contents+1);
  /* Pass through the unescape function */
  unescaped = mpcf_unescape(unescaped);
  /* Construct a new lval using the string */
  lval* str = lval_str(unescaped);
  /* Free the string and return */
  free(unescaped);
  return str;
}
```

If this all works we should be able to play around with strings in the prompt. Next we'll add functions which can actually make use of them.

```lispy
lispy> "hello"
"hello"
lispy> "hello\n"
"hello\n"
lispy> "hello\""
"hello\""
lispy> head {"hello" "world"}
{"hello"}
lispy> eval (head {"hello" "world"})
"hello"
lispy>
```


Comments
--------

While we're building in new syntax to the language we may as well look at comments.

Just like in C, we can use comments in inform other people (or ourselves) about what the code is meant to do or why it has been written. In C comments go between `/*` and `*/`. Lisp comments, on the other hand, start with `;` and run to the end of the line.

I attempted to research why Lisps use `;` for comments, but it appears that the origins of this have been lost in the mists of time. In absence of real truth I imagine it as a small rebellion against the imperative languages such as C and Java which use semicolons so shamelessly and frequently to separate/terminate statements. Compared to Lisp all these languages are just comments!

So in lisp a comment is defined by a semicolon `;` followed by any number of characters that are not newline characters represented by either `\r` or `\n`. We can use another regex to define it.

```
comment : /;[^\\r\\n]*/ ;
```

As with strings we need to create a new parser and use this to update our language in `mpca_lang`. We also need to remember to add the parser to `mpc_cleanup`, and update the first integer argument to reflect the new number of parsers passed in.

Our final grammar now look like this.

```c
mpca_lang(MPC_LANG_DEFAULT,
  "                                              \
    number  : /-?[0-9]+/ ;                       \
    symbol  : /[a-zA-Z0-9_+\\-*\\/\\\\=<>!&]+/ ; \
    string  : /\"(\\\\.|[^\"])*\"/ ;             \
    comment : /;[^\\r\\n]*/ ;                    \
    sexpr   : '(' <expr>* ')' ;                  \
    qexpr   : '{' <expr>* '}' ;                  \
    expr    : <number>  | <symbol> | <string>    \
            | <comment> | <sexpr>  | <qexpr>;    \
    lispy   : /^/ <expr>* /$/ ;                  \
  ",
  Number, Symbol, String, Comment, Sexpr, Qexpr, Expr, Lispy);

```

And the cleanup function looks like this.

```c
mpc_cleanup(8, Number, Symbol, String, Comment, Sexpr, Qexpr, Expr, Lispy);
```

Because comments are only for programmings reading the code, our internal function for reading them in just consists of ignoring them. We can add a clause to deal with them in a similar way to brackets and parenthesis in `lval_read`.

```c
if (strstr(t->children[i]->tag, "comment")) { continue; }
```

Comments won't be of much use on the interactive prompt, but they will be very helpful for adding into files of code to annotate them.


Load Function
-------------

We want to built a function that can load and evaluate a file when passed a string of its name. To implement this function we'll need to make used of our grammar as we'll need it to to read in the file contents, parse, and evaluate them. Our load function is going to rely on our `mpc_parser*` called `Lispy`.

Therefore, just like with functions, we need to forward declare our parser pointers, and place them to the top of the file.

```c
mpc_parser_t* Number;
mpc_parser_t* Symbol;
mpc_parser_t* String;
mpc_parser_t* Comment;
mpc_parser_t* Sexpr;
mpc_parser_t* Qexpr;
mpc_parser_t* Expr;
mpc_parser_t* Lispy;

```

Our `load` function will be just like any other builtin. We need to start by checking that the input argument is a single string. Then we can use the `mpc_fparse_contents` function to read in the contents of a file using a grammar. Just like `mpc_parse` this parses the contents of a file into some `mpc_result` object, which is our case is an *abstract syntax tree** again or an *error**.

Slightly differently to our command prompt, on successfully parsing a file we shouldn't treat it like one expression. When typing into a file we let users list multiple expressions and evaluate all of them individually. To achieve this behaviour we need to loop over each expression in the contents of the file and evaluate it one by one. If there are any errors we should print them and continue.

If there is a parse error instead of chucking it away we're going to extract the message and put it into a error `lval` which we return. If there are no errors the return value for this builtin can just be the empty expression. The full code for this looks like this.

```c
lval* builtin_load(lenv* e, lval* a) {
  LASSERT_NUM("load", a, 1);
  LASSERT_TYPE("load", a, 0, LVAL_STR);

  /* Parse File given by string name */
  mpc_result_t r;
  if (mpc_fparse_contents(a->cell[0]->str, Lispy, &r)) {

    /* Read contents */
    lval* expr = lval_read(r.output);
    mpc_ast_delete(r.output);

    /* Evaluate each Expression */
    while (expr->count) {
      lval* x = lval_eval(e, lval_pop(expr, 0));
      /* If Evaluation leads to error print it */
      if (x->type == LVAL_ERR) { lval_println(x); }
      lval_del(x);
    }

    /* Delete expressions and arguments */
    lval_del(expr);
    lval_del(a);

    /* Return empty list */
    return lval_sexpr();

  } else {
    /* Get Parse Error as String */
    char* err_msg = mpc_err_string(r.error);
    mpc_err_delete(r.error);

    /* Create new error message using it */
    lval* err = lval_err("Could not load Library %s", err_msg);
    free(err_msg);
    lval_del(a);

    /* Cleanup and return error */
    return err;
  }
}
```


Command Line Arguments
----------------------

With the ability to load files, we can take the chance to add in some functionality typical of other programming languages. When file names are given as arguments to the command line we can try to run these files. For example to run a python file one might write `python filename.py`.

These command line arguments are accessible using the `argc` and `argv` variables that are given to `main`. The `argc` variable gives the number of arguments, and `argv` specifies each string. The `argc` is always set to at least one, where the first argument is always the complete command invoked.

That means if `argc` is set to `1` we can invoke the interpreter, otherwise we can run each of the arguments through the `builtin_load` function.

```c
/* Supplied with list of files */
if (argc >= 2) {

  /* loop over each supplied filename (starting from 1) */
  for (int i = 1; i < argc; i++) {

    /* Create an argument list with a single argument being the filename */
    lval* args = lval_add(lval_sexpr(), lval_str(argv[i]));

    /* Pass to builtin load and get the result */
    lval* x = builtin_load(e, args);

    /* If the result is an error be sure to print it */
    if (x->type == LVAL_ERR) { lval_println(x); }
    lval_del(x);
  }
}

```

It's now possible to write some basic program and try to invoke it using this method.

```
lispy example.lspy
```


Print Function
--------------

If we are running programs from the command line we might want them to output some data, rather than just define functions and other values. We can add a `print` function to our Lisp which makes use of our existing `lval_print` function.

This function prints each argument separated by a space and then prints a newline character to finish. It returns the empty expression.

```c
lval* builtin_print(lenv* e, lval* a) {

  /* Print each argument followed by a space */
  for (int i = 0; i < a->count; i++) {
    lval_print(a->cell[i]); putchar(' ');
  }

  /* Print a newline and delete arguments */
  putchar('\n');
  lval_del(a);

  return lval_sexpr();
}
```


Error Function
--------------

We can also make use of strings to add in an error reporting function. This can take as input a user supplied string and provide it as an error message for `lval_err`.

```c
lval* builtin_error(lenv* e, lval* a) {
  LASSERT_NUM("error", a, 1);
  LASSERT_TYPE("error", a, 0, LVAL_STR);

  /* Construct Error from first argument */
  lval* err = lval_err(a->cell[0]->str);

  /* Delete arguments and return */
  lval_del(a);
  return err;
}
```

The final step is to register these as builtins. Now finally we can start building up libraries and writing them to files!

```c
/* String Functions */
lenv_add_builtin(e, "load", builtin_load);
lenv_add_builtin(e, "error", builtin_error); lenv_add_builtin(e, "print", builtin_print);

```

```lispy
lispy> print "Hello World!"
"Hello World!"
()
lispy> error "This is an error"
Error: This is an error
lispy> load "hello.lspy"
"Hello World!"
()
lispy>

```


Finishing Up
------------

This is the last chapter in which we are going to explicitly work on our C implementation of Lisp. The result of this chapter will be the final state of your language implementation while I am still involved.

The final line count should clock in somewhere close to 1000 lines of code. Writing this amount of code is not trivial. If you've made it this far you've written a real program and started on a proper project. The skills you've learnt here should be transferable, and give you the confidence to seek out your own goals and targets. You now have a complex and beautiful program which you can interact and play with. This is something you should be proud of. Go show it off to your parents and friends!

In the next chapter we start using our Lisp to build up a standard library of common functions. After that I describe some possible improvements and directions in which the language should be taken. Although we've finished with my involvement this is really this is only the beginning. Thanks for following along, and good luck with whatever C you write in the future!


Reference
---------

<div class="panel-group alert alert-warning" id="accordion">
  <a href="strings.c">strings.c</a>
</div>


Bonus Marks
-----------

<div class="alert alert-warning">
  <ul class="list-group">
    <li class="list-group-item">&rsaquo; Adapt the builtin function `join` to work on strings.</li>
    <li class="list-group-item">&rsaquo; Adapt the builtin function `head` to work on strings.</li>
    <li class="list-group-item">&rsaquo; Adapt the builtin function `tail` to work on strings.</li>
    <li class="list-group-item">&rsaquo; Create a builtin function `read` that reads in and converts a string to a Q-expression.</li>
    <li class="list-group-item">&rsaquo; Create a builtin function `show` that can print the contents of strings as it is (unescaped).</li>
    <li class="list-group-item">&rsaquo; Create a special value `ok` to return instead of empty expressions `()`.</li>
    <li class="list-group-item">&rsaquo; Add functions to wrap all of C's file handling functions such as `fopen` and `fgets`.</li>
  </ul>
</div>