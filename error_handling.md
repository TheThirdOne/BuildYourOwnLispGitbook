Error Handling
==============

Crashes
-------

Some of you may have noticed a problem with the previous chapter's program. Try entering this into the prompt and see what happens.

```lispy
Lispy Version 0.0.0.0.3
Press Ctrl+c to Exit

lispy> / 10 0
```

Ouch. The program crashed upon trying to divide by zero. It's okay if a program crashes during development, but our final program would hopefully never crash, and always explain to the user what went wrong.

At the moment our program can produce syntax errors but it still has no functionality for reporting errors in the evaluation of expressions. We need to build in some kind of error handling functionality to do this. It can be awkward in C, but if we start off on the right track, it will pay off later on when our system gets more complicated.


Lisp Value
----------

There are several ways to deal with errors in C, but in this context my preferred method is to make errors a possible result of evaluating an expression. Then we can say that, in Lispy, an expression will evaluate to *either* a *number*, or an *error*. For example `+ 1 2` will evaluate to a number, but `/ 10 0` will evaluate to an error.

For this we need a data structure that can act as either one thing or anything. For simplicity sake we are just going to use a `struct` with fields specific to each thing that can be represented, and a special field `type` to tell us exactly what fields are meaningful to access.

This we are going to call an `lval`, which stands for *Lisp Value*.

```c
/* Declare New lval Struct */
typedef struct {
  int type;
  long num;
  int err;
} lval;
```

Enumerations
------------

You'll notice the type of the fields `type`, and `err`, is `int`. This means they are represented by a single integer number.

The reason we pick `int` is because we will assign meaning to each integer value, to encode what we require. For example we can make a rule *"If `type` is `0` then the structure is a Number."*, or *"If `type` is `1` then the structure is an Error."* This is a good way of doing things. It is simple and effective.

But if we litter our code with stray `0` and `1` then it is going to become increasingly unclear as to what is happening. Instead we can use named constants that have been assigned these integer values. This gives the reader an indication as to *why* one might be comparing a number to `0` or `1` and *what* is meant in this context.

In C this is supported using something called an `enum`.

```c
/* Create Enumeration of Possible lval Types */
enum { LVAL_NUM, LVAL_ERR };
```

An `enum` is a declaration of variables which under the hood are automatically assigned integer constant values. Above is how we would declare some enumerated values for the `type` field.

We also want to declare an enumeration for the *error* field. We have three error cases in our particular program. There is division by zero, an unknown operator, or being passed a number that is too large to be represented internally using a `long`. These can be enumerated as follows.

```c
/* Create Enumeration of Possible Error Types */
enum { LERR_DIV_ZERO, LERR_BAD_OP, LERR_BAD_NUM };
```

Lisp Value Functions
--------------------

Our `lval` type is almost ready to go. Unlike the previous `long` type we have no current method for creating new instances of it. To do this we can declare two functions that construct an `lval` of either an *error* type or a *number* type.

```c
/* Create a new number type lval */
lval lval_num(long x) {
  lval v;
  v.type = LVAL_NUM;
  v.num = x;
  return v;
}

/* Create a new error type lval */
lval lval_err(int x) {
  lval v;
  v.type = LVAL_ERR;
  v.err = x;
  return v;
}
```

These functions first create an `lval` called `v`, and assign the fields before returning it.

Because our `lval` function can now be one of two things we can no longer just use `printf` to output it. We will want to do different behaviour depending upon the type of the `lval` that is given. There is a concise way to do this in C using the `switch` statement. This takes some value as input and compares it to other known values, known as *cases*. When the values are equal it executes the code that follows up until the next `break` statement.

Using this we can build a function that can print an `lval` of any type like this.

```c
/* Print an "lval" */
void lval_print(lval v) {
  switch (v.type) {
    /* In the case the type is a number print it, then 'break' out of the switch. */
    case LVAL_NUM: printf("%li", v.num); break;

    /* In the case the type is an error */
    case LVAL_ERR:
      /* Check What exact type of error it is and print it */
      if (v.err == LERR_DIV_ZERO) { printf("Error: Division By Zero!"); }
      if (v.err == LERR_BAD_OP)   { printf("Error: Invalid Operator!"); }
      if (v.err == LERR_BAD_NUM)  { printf("Error: Invalid Number!"); }
    break;
  }
}

/* Print an "lval" followed by a newline */
void lval_println(lval v) { lval_print(v); putchar('\n'); }
```


Evaluating Errors
-----------------

Now that we know how to work with the `lval` type, we need to change our evaluation functions to use it instead of `long`.

As well as changing the type signatures we need to change the functions such that they work correctly upon encountering either an *error* as input, or a *number* as input.

In our `eval_op` function, if we encounter an error we should return it right away, and only do computation if both the arguments are numbers. We should modify our code to return an error rather than attempt to divide by zero. This will fix the crash from right at the beginning of this chapter.

```c
lval eval_op(lval x, char* op, lval y) {

  /* If either value is an error return it */
  if (x.type == LVAL_ERR) { return x; }
  if (y.type == LVAL_ERR) { return y; }

  /* Otherwise do maths on the number values */
  if (strcmp(op, "+") == 0) { return lval_num(x.num + y.num); }
  if (strcmp(op, "-") == 0) { return lval_num(x.num - y.num); }
  if (strcmp(op, "*") == 0) { return lval_num(x.num * y.num); }
  if (strcmp(op, "/") == 0) {
    /* If second operand is zero return error instead of result */
    return y.num == 0 ? lval_err(LERR_DIV_ZERO) : lval_num(x.num / y.num);
  }

  return lval_err(LERR_BAD_OP);
}
```

<div class="alert alert-warning">
  **What is that `?` doing there?**

  You'll notice that for division to check if the second argument is zero we use a question mark symbol `?`, followed by a colon `:`. This is called the *ternary operator*, and it allows you to write conditional expressions on one line.

  It works something like this. `<condition> ? <then> : <else>`. In other words, if the condition is true it returns what follows the `?`, otherwise it returns what follows `:`.

  Some people dislike this operator because they believe it makes code unclear. If you are unfamiliar with the ternary operator, initially, you may feel awkward to use it; but once you get to know it there are rarely problems.
</div>

We need to give a similar treatment to our `eval` function. In this case because we've defined `eval_op` to robustly handle errors we just need to add the error conditions to our number conversion function.

In this case we use the `strtol` function to convert from string to `long`. This allows us to check a special variable `errno` to ensure the conversion goes correctly. This is a more robust way to convert numbers than our previous method using `atoi`.

```c
lval eval(mpc_ast_t* t) {

  if (strstr(t->tag, "number")) {
    /* Check if there is some error in conversion */
    long x = strtol(t->contents, NULL, 10);
    return errno != ERANGE ? lval_num(x) : lval_err(LERR_BAD_NUM);
  }

  char* op = t->children[1]->contents;
  lval x = eval(t->children[2]);

  int i = 3;
  while (strstr(t->children[i]->tag, "expr")) {
    x = eval_op(x, op, eval(t->children[i]));
    i++;
  }

  return x;
}
```

The final small step is to change how we print the result found by our evaluation to use our newly defined printing function which can print any type of `lval`.

```c
lval result = eval(r.output);
lval_println(result);
mpc_ast_delete(r.output);
```

And we are done! Try running this new program and make sure there are no crashes when dividing by zero!

```lispy
lispy> / 10 0
Error: Division By Zero!
lispy> / 10 2
5
```


Plumbing
--------

![plumbing](img/plumbing.png "Plumbing &bull; Harder than you think")

Some of you who have gotten this far in the book may feel uncomfortable with how it is progressing. You may feel you've managed to follow instructions well enough, but don't have a clear understanding of all of the underlying mechanisms going on behind the scenes.

If this is the case I want to reassure you that you are doing well. If you don't understand the internals it's because I have not explained everything in sufficient depth. This is okay.

To be able to progress and get code to work under these conditions is a really great skill in programming, and if you've made it this far it shows you have it.

In programming we call this *plumbing*. Roughly speaking this is following instructions to try and tie together a bunch libraries or components, without fully understanding how they work internally.

It requires *faith* and *intuition*. *Faith* is required to believe that if the stars align, and every incantation is correctly performed for this magical machine, the right thing will really happen. And *intuition* is required to work out what has gone wrong, and how to fix things when they don't go as planned.

Unfortunately these can't be taught directly, so if you've made it this far then *congratulations!* You've made it over a difficult hump, and in the following chapters I promise we'll finish up with the plumbing, and actually start to programming that feels fresh and wholesome.


Reference
---------

<div class="panel-group alert alert-warning" id="accordion">
  <a href="src/error_handling.c">error_handling.c</a>
</div>

Bonus Marks
-----------

<div class="alert alert-warning">
  <ul class="list-group">
    <li class="list-group-item">&rsaquo; How do you give an `enum` a name?</li>
    <li class="list-group-item">&rsaquo; What are `union` data types and how do they work?</li>
    <li class="list-group-item">&rsaquo; Can you change how `lval` is defined to use `union` instead of `struct`?</li>
    <li class="list-group-item">&rsaquo; What are the advantages over using a `union` instead of `struct`?</li>
    <li class="list-group-item">&rsaquo; Extend parsing and evaluation to support the remainder operator `%`.</li>
    <li class="list-group-item">&rsaquo; Extend parsing and evaluation to support decimal types using a `double` field.</li>
  </ul>
</div>