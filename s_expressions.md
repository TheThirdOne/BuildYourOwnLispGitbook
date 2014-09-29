S-Expressions
=============

Lists and Lisps
---------------

![lisp](img/lisp.png "ALL CAPS &bull; SO RIGHT YET SO WRONG")

Lisps are famous for having little distinction between data and code. They use the same structures to represent both. This allows them to do many powerful things which other languages cannot do. If we want this power for our programming language we're going to have to separate out the process of *reading* in input, and *evaluating* that input we have stored.

The final result of this chapter will only differ slightly in behaviour from the previous chapter. This is because we are going to spend time changing how things work internally. This is called *re-factoring* and it will make our life a lot easier later on. Like preparation for a meal, just because we're not putting food onto plates it doesn't mean we're wasting time. Sometimes the anticipation is even better than eating!

To store the program we will need to create an internal list structure that is built up recursively of numbers, symbols, and other lists. In Lisp, this structure is commonly called an S-Expression standing for *Symbolic Expression*. We will extend our `lval` structure to be able to represent it. The evaluation behaviour of S-Expressions is the behaviour typical of Lisps, that we are used to so far. To evaluate an S-Expression we look at the first item in the list, and take this to be the operator. We then look at all the other items in the list, and take these as operands to get the result.

By introducing S-Expressions we'll finally be entering the world of Lisp.


Pointers
--------

In C no concept of lists can be explored without dealing properly with pointers. Pointers are a famously misunderstood aspect of C. They are difficult to teach because while being conceptually very simple, they come with a lot of new terminology, and often no clear use-case. This makes them appear far more monstrous than they are. Luckily for us, we have a couple ideal use-cases, both of which are extremely typical in C, and will likely end up being how you use pointers 90% of the time.

The reason we need pointers in C is because of how function calling works. When you call a function in C the arguments are always passed *by value*. This means *a copy* of them is passed to the function you call. This is true for `int`, `long`, `char`, and user defined `struct` types such as `lval`. Most of the time this is great but occasionally it can cause issues.

A common problem is if we have a large struct containing many other sub structs we wish to pass around. Every time we call a function we must create another copy of it. Suddenly the amount of data that needs to be copied around just to call a function can become huge!

A second problem is this. When we define a `struct`, it is always a fixed size. It has a limited number of fields, and each of these fields must be a struct which itself is limited in size. If I want to call a function with just *a list of things*, where the number of *things* varies from call to call, clearly I can't use a `struct` to do this.

To get around these issues the developers of C (or y'know...someone) came up with a clever idea. They imagined computer memory as a single huge list of bytes. In this list each byte can be given a global index, or position. A bit like a house number. The first byte is numbered `0`, the second is `1`, etc.

In this case, all the data in the computer, including the structs and variables used in the currently running program, start at some index in this huge list. If, rather than copying the data itself to a function, we instead copy a number representing the *index* at where this data starts, the function being called can look up any amount of data it wants.

By using *addresses* instead of the actual data, we can allow a function to access and modify some location in memory without having to copy any data. Functions can also use pointers to do other cool stuff, like output data to some address given as input.

Because the total size of computer memory is fixed, the number of bytes needed to represent an address is always the same. But if we keep track of it, the number of bytes the address points to can grow and shrink. This means we can create a variable sized data-structure and still sort of *pass* it to a function, which can inspect and modify it.

So a pointer is just a number. A number representing the starting index of some data in memory. The type of the pointer hints to us, and the compiler, what type of data might be accessible at this location.

We can declare pointer types by suffixing existing ones with the the `*` character. We've seen some examples of this already with `mpc_parser_t*`, `mpc_ast_t*`, or `char*`.

To create a pointer to some data, we need to get its index, or *address*. To get the address of a some data we use the *address of* operator `&`. Again you've seen this before when we passed in a pointer to `mpc_parse` so it would output into our `mpc_result_t`.

Finally to get the data at an address, called *dereferencing*, we use the `*` operator on the left hand size of a variable. To get the data at the field of a pointer to a struct we use the arrow `->`. This you saw in chapter 7.


The Stack &amp; The Heap
------------------------

I said that memory can be visualized of as one long list of bytes. Actually it is better to imagine it split into two sections. These sections are called *The Stack* and *The Heap*.

Some of you may have heard tales of these mysterious locations, such as *"the stack grows down but the heap grows up"*, or *"there can be many stacks, but only one heap"*. These sorts of things don't matter much. Dealing with the stack and the heap in C can be complex, but it doesn't have to be a mystery. In essence they are just two sections of memory used for two different tasks.

The Stack
---------

![building](img/building.png "The Stack &bull; Like what you do with bricks")
  
The Stack is the memory where your program lives. It is where all of your temporary variables and data structures exist as you manipulate and edit them. Each time you call a function a new area of the stack is put aside for it to use. Into this area are put local variables, copies of any arguments passed to the function, as well as some bookkeeping data such as who the caller was, and what to do when finished. When the function is done the area it used is unallocated, ready for use again by someone else.

I like to think of the stack as a building site. Each time we need to do something new we corner off a section of space, enough for our tools and materials, and set to work. We can still go to other parts of the site, or go off-site, if we need certain things, but all our work is done in this section. Once we are done with some task, we take what we've constructed to a new place and clean up that section of the space we've been using to make it.


The Heap
--------

![storage](img/storage.png "The Heap &bull; U LOCK. KEEP KEY")

The Heap is a section of memory put aside for storage of objects with a longer lifespan. Memory in this area has to be manually allocated and deallocated. To allocate new memory the `malloc` function is used. This function takes as input the number of bytes required, and returns back a pointer to a new block of memory with that many bytes set aside.

When done with the memory at that location it must be released again. To do this the pointer received from `malloc` should be passed to the `free` function.

Using the Heap is trickier than the Stack because it requires the programmer to remember to call `free` and to call it correctly. If he or she doesn't, the program may continuously allocate more and more memory. This is called a *memory leak*. A easy rule to avoid this is to ensure for each `malloc` there is a corresponding (and only one corresponding) `free`. If this can always be ensured the program should be handling The Heap correctly.

I Imagine the Heap like a huge U-Store-It. We can call up the reception with `malloc` and request a number of boxes. With these boxes we can do what we want, and we know they will persist no matter how messy the building site gets. We can take things to and from the U-Store-It and the building site. It is useful to store materials and large objects which we only need to retrieve once in a while. The only problem is we need to remember to call the receptionist again with `free` when we are done. Otherwise soon we'll have requested all the boxes, have no space, and run up a huge bill.


Parsing Expressions
-------------------

Because we're now thinking in S-Expressions, and not Polish Notation we need to update our parser. The syntax for S-Expressions is simple. It is just a number of other Expressions between parenthesis, where an Expression can be a Number, Operator, or other S-Expression . We can modify our existing parse rules to reflect this. We also are going to rename our `operator` rule to `symbol`. This is in anticipation of adding more operators, variables and functions later.

```c
mpc_parser_t* Number = mpc_new("number");
mpc_parser_t* Symbol = mpc_new("symbol");
mpc_parser_t* Sexpr  = mpc_new("sexpr");
mpc_parser_t* Expr   = mpc_new("expr");
mpc_parser_t* Lispy  = mpc_new("lispy");

mpca_lang(MPC_LANG_DEFAULT,
  "                                          \
    number : /-?[0-9]+/ ;                    \
    symbol : '+' | '-' | '*' | '/' ;         \
    sexpr  : '(' <expr>* ')' ;               \
    expr   : <number> | <symbol> | <sexpr> ; \
    lispy  : /^/ <expr>* /$/ ;               \
  ",
  Number, Symbol, Sexpr, Expr, Lispy);
```

We should also remember to cleanup these rules on exit.

```c
mpc_cleanup(5, Number, Symbol, Sexpr, Expr, Lispy);
```


Expression Structure
--------------------

We need a way to store S-Expressions internally as `lval`. This means we'll also need to store internally *Symbols* and *Numbers*. We're going to add two new `lval` types to the enumeration. The first new type is `LVAL_SYM`, which we're going to use to represent operators such as `+`. The second new type is `LVAL_SEXPR` which we're going to use to represent S-Expressions.

```c
/* Add SYM and SEXPR as possible lval types. */
enum { LVAL_ERR, LVAL_NUM, LVAL_SYM, LVAL_SEXPR };

```

S-Expressions are variable length *lists* of other S-Expressions. As we learnt at the beginning of this chapter we can't create variable length structs, so we are going to need to use a pointer. We are going to create a pointer field `cell` which points to a location where we store a list of `lval*`. These are pointers to the other individual `lval`. Our field should therefore be a double pointer type `lval**`. A *pointer to `lval` pointers*.

We will also need to keep track of how many `lval*` are in this list, so we add an extra field `count` to record this.

<div class="alert alert-warning">
  **Are there ever pointers to pointers to pointers?**

  There is an old programming joke (and by *programming joke* I mean *silly anecdote*), which says you can rate C programmers by how many stars are on their pointers.

  Beginners program might only use `char*` or the odd `int*`, so they were called *one star programmer*. Most intermediate programs contain double pointer types such as `lval**`. These programmers are therefore called *two star programmers*. To spot a triple pointer is something special. You would be viewing the work of someone grand and terrible, writing code not meant to be read with mortal eyes. As such being called a *three star programmer* is rarely a compliment.

  As far as I know, a quadruple pointer has never been seen in the wild.
</div>

To represent symbols we're going to use a string. We're also going to change the representation of errors to a string. This means we can store a unique error message rather than just an error code. This will make our error reporting better and more flexible, and we can get rid of the original error `enum`. Our updated `lval` struct looks like this.

```c
typedef struct lval {
  int type;

  long num;

  /* Error and Symbol types have some string data */
  char* err;
  char* sym;

  /* Count and Pointer to a list of "lval*" */
  int count;
  struct lval** cell;

} lval;
```

<div class="alert alert-warning">
  **What is that `struct` keyword doing there?**

  Our new definition of `lval` needs to contain a reference to itself. This means we have to slightly change how it is defined. Before we open the curly brackets we can put the name of the struct, and then refer to this inside using `struct lval`. Even though a struct can refer to its own type, it must only contain pointers to its own type, not its own type directly. Otherwise the size of the struct would refer to itself, and grow infinite in size when you tried to calculate it!
</div>

Constructors &amp; Destructors
------------------------------

We can change our `lval` construction functions to return pointers to an `lval`, rather than one directly. This will make keeping track of `lval` variables easier. For this we need to use `malloc` with the `sizeof` function to allocate enough space for the `lval` struct, and then to fill in the fields with the relevant information using the arrow operator `->`.

When we construct an `lval` its fields may contain pointers to other things that have been allocated on the heap. This means we need to be careful. Whenever we are done with an `lval` we also need to delete the things it points to on the heap. We will have to make a rule to ourselves. Whenever we free the memory allocated for an `lval`, we also free all the things it points to.

```c
/* Construct a pointer to a new Number lval */
lval* lval_num(long x) {
  lval* v = malloc(sizeof(lval));
  v->type = LVAL_NUM;
  v->num = x;
  return v;
}

/* Construct a pointer to a new Error lval */
lval* lval_err(char* m) {
  lval* v = malloc(sizeof(lval));
  v->type = LVAL_ERR;
  v->err = malloc(strlen(m) + 1);
  strcpy(v->err, m);
  return v;
}

/* Construct a pointer to a new Symbol lval */
lval* lval_sym(char* s) {
  lval* v = malloc(sizeof(lval));
  v->type = LVAL_SYM;
  v->sym = malloc(strlen(s) + 1);
  strcpy(v->sym, s);
  return v;
}

/* A pointer to a new empty Sexpr lval */
lval* lval_sexpr(void) {
  lval* v = malloc(sizeof(lval));
  v->type = LVAL_SEXPR;
  v->count = 0;
  v->cell = NULL;
  return v;
}

```

<div class="alert alert-warning">
  **What is `NULL`?**

  `NULL` is a special constant that points to memory location `0`. In many places it is used as a convention to signify some non-value or empty data. Above we use it to specify that we have a data pointer, but that it doesn't point to any number of data items. You will see `NULL` crop up a lot more later on so always remember to look out for it.
</div>

<div class="alert alert-warning">
  **Why are you using `strlen(s) + 1`?**

  In C strings are *null terminated*. This means that the final character of them is always the zero character `\0`. This is a convention in C to signal the end of a string. It is important that all strings are stored this way otherwise programs will break in nasty ways.

  The `strlen` function only returns the number of bytes in a string *excluding* the null terminator. This is why we need to add one, to ensure there is enough allocated space for it all!
</div>

We now need a special function to delete `lval*`. This should call `free` on the pointer itself to release the memory acquired from `malloc`, but more importantly it should inspect the type of the `lval`, and release any memory pointed to by its fields. If we match every call to one of the above construction functions with `lval_del` we can ensure we will get no memory leaks.

```c
void lval_del(lval* v) {

  switch (v->type) {
    /* Do nothing special for number type */
    case LVAL_NUM: break;

    /* For Err or Sym free the string data */
    case LVAL_ERR: free(v->err); break;
    case LVAL_SYM: free(v->sym); break;

    /* If Sexpr then delete all elements inside */
    case LVAL_SEXPR:
      for (int i = 0; i < v->count; i++) {
        lval_del(v->cell[i]);
      }
      /* Also free the memory allocated to contain the pointers */
      free(v->cell);
    break;
  }

  /* Finally free the memory allocated for the "lval" struct itself */
  free(v);
}
```


Reading Expressions
-------------------

First we are going to *read* in the program and construct an `lval*` that represents it all. Then we are going to *evaluate* this `lval*` to get the result of our program. This first stage should convert the *abstract syntax tree* into an S-Expression, and the second stage should evaluate this S-Expression using our normal Lisp rules.

To complete the first stage we can recursively look at each node of the tree, and construct different `lval*` types depending on the `tag` and `contents` fields of the node.

If the given node is tagged as a `number` or `symbol`, then we use our constructors to return an `lval*` directly for those types. If the given node is the `root`, or an `sexpr`, then we create an empty S-Expression `lval` and slowly add each valid sub-expression contained in the tree.

To add an element to an S-Expression we can create a function `lval_add`. This function increases the count of the Expression list by one, and then uses `realloc` to reallocate the amount of space required by `v->cell`. This new space can be used to store the extra `lval*` required. Using this new space it sets the final value of the list with `v->cell[v->count-1]` to the value `lval* x` passed in.

<div class="alert alert-warning">
  **Don't Lisps use <a href="http://en.wikipedia.org/wiki/Cons">Cons cells</a>?**

  Other Lisps have a slightly different definition of what an S-Expression is. In most other Lisps S-Expressions are defined inductively as either an *atom* such as a symbol of number, or two other S-Expressions joined, or *cons*, together.

  This naturally leads to an implementation using *linked lists*, a different data structure to the one we are using. I choose to represent S-Expressions as a variable sized array in this book for the purposes of simplicity, but it is important to be aware that the official definition, and typical implementation are both subtly different.
</div>

```c
lval* lval_add(lval* v, lval* x) {
  v->count++;
  v->cell = realloc(v->cell, sizeof(lval*) * v->count);
  v->cell[v->count-1] = x;
  return v;
}

lval* lval_read_num(mpc_ast_t* t) {
  long x = strtol(t->contents, NULL, 10);
  return errno != ERANGE ? lval_num(x) : lval_err("invalid number");
}

lval* lval_read(mpc_ast_t* t) {

  /* If Symbol or Number return conversion to that type */
  if (strstr(t->tag, "number")) { return lval_read_num(t); }
  if (strstr(t->tag, "symbol")) { return lval_sym(t->contents); }

  /* If root (>) or sexpr then create empty list */
  lval* x = NULL;
  if (strcmp(t->tag, ">") == 0) { x = lval_sexpr(); }
  if (strstr(t->tag, "sexpr"))  { x = lval_sexpr(); }

  /* Fill this list with any valid expression contained within */
  for (int i = 0; i < t->children_num; i++) {
    if (strcmp(t->children[i]->contents, "(") == 0) { continue; }
    if (strcmp(t->children[i]->contents, ")") == 0) { continue; }
    if (strcmp(t->children[i]->contents, "}") == 0) { continue; }
    if (strcmp(t->children[i]->contents, "{") == 0) { continue; }
    if (strcmp(t->children[i]->tag,  "regex") == 0) { continue; }
    x = lval_add(x, lval_read(t->children[i]));
  }

  return x;
}
```

Printing Expressions
--------------------

We are so close to trying out all of our new changes! We need to modify our print function to print out S-Expressions types. Using this we can double check that the *reading* phase is working correctly by printing out the S-Expressions we read in and verifying they match those we input.

To print out S-Expressions we can create another function that loops over all the sub-expressions of an expression and prints these individually separated by spaces, just like how they are input.

```c
void lval_expr_print(lval* v, char open, char close) {
  putchar(open);
  for (int i = 0; i < v->count; i++) {

    /* Print Value contained within */
    lval_print(v->cell[i]);

    /* Don't print trailing space if last element */
    if (i != (v->count-1)) {
      putchar(' ');
    }
  }
  putchar(close);
}

void lval_print(lval* v) {
  switch (v->type) {
    case LVAL_NUM:   printf("%li", v->num); break;
    case LVAL_ERR:   printf("Error: %s", v->err); break;
    case LVAL_SYM:   printf("%s", v->sym); break;
    case LVAL_SEXPR: lval_expr_print(v, '(', ')'); break;
  }
}

void lval_println(lval* v) { lval_print(v); putchar('\n'); }
```

<div class="alert alert-warning">
  **I can't declare these functions because they call each other.**

  The `lval_expr_print` function calls the `lval_print` function and vice-versa. There is no way we can order them in the source file to resolve this dependency. Instead we need to *forward declare* one of them. This is declaring a function without giving it a body. It lets other functions call it, while allowing you to define it properly later on. To write a forward declaration write the function definition but instead of the body put a semicolon `;`. In this example we should put `void lval_print(lval* v);` somewhere in the source file before `lval_expr_print`.

  You'll definitely run into this later, and I won't always alert you about it, so try to remember how to fix it!
</div>


In our main loop, we can remove the evaluation for now, and instead try reading in the result and printing out what we have read.

```c
lval* x = lval_read(r.output);
lval_println(x);
lval_del(x);
```

If this is successful you should see something like the following when entering input to your program.

```lispy
lispy> + 2 2
(+ 2 2)
lispy> + 2 (* 7 6) (* 2 5)
(+ 2 (* 7 6) (* 2 5))
lispy> *     55     101  (+ 0 0 0)
(* 55 101 (+ 0 0 0))
lispy>
```


Evaluating Expressions
----------------------

The behaviour of our evaluation function is largely the same as before. We need to adapt it to deal with `lval*` and our more relaxed definition of what constitutes an expression. We can think of our evaluation function as a kind of transformer. It takes in some `lval*` and transforms it in some way to some new `lval*`. In some cases it can just return exactly the same thing. In other cases it may modify the input `lval*` and the return it. In many cases it will delete the input, and return something completely different. If we are going to return something new we must always remember to delete the `lval*` we get as input.

For S-Expressions we first evaluate all the children of the S-Expression. If any of these children are errors we return the first error we encounter using a function we'll define later called `lval_take`.

If the S-Expression has no children we just return it directly. This corresponds to the empty expression, denoted by `()`. We also check for single expressions. These are expressions with only one child such as `(5)`. In this case we return the single expression contained within the parenthesis.

If neither of these are the case we know we have a valid expression with more than one child. In this case we separate the first element of the expression using a function we'll define later called `lval_pop`. We then check this is a *symbol* and not anything else. If it is a symbol we check what symbol it is, and pass it, and the arguments, to a function `builtin_op` which does our calculations. If the first element is not a symbol we delete it, and the values passed into the evaluation function, returning an error.

To evaluate all other types we just return them directly back.

```c
lval* lval_eval_sexpr(lval* v) {

  /* Evaluate Children */
  for (int i = 0; i < v->count; i++) {
    v->cell[i] = lval_eval(v->cell[i]);
  }

  /* Error Checking */
  for (int i = 0; i < v->count; i++) {
    if (v->cell[i]->type == LVAL_ERR) { return lval_take(v, i); }
  }

  /* Empty Expression */
  if (v->count == 0) { return v; }

  /* Single Expression */
  if (v->count == 1) { return lval_take(v, 0); }

  /* Ensure First Element is Symbol */
  lval* f = lval_pop(v, 0);
  if (f->type != LVAL_SYM) {
    lval_del(f); lval_del(v);
    return lval_err("S-expression Does not start with symbol!");
  }

  /* Call builtin with operator */
  lval* result = builtin_op(v, f->sym);
  lval_del(f);
  return result;
}

lval* lval_eval(lval* v) {
  /* Evaluate Sexpressions */
  if (v->type == LVAL_SEXPR) { return lval_eval_sexpr(v); }
  /* All other lval types remain the same */
  return v;
}
```

There are two functions we've used and not defined in the above code. These are `lval_pop` and `lval_take`. These are general purpose functions for manipulating S-Expression `lval` types which we'll make use of here and in the future.

The `lval_pop` function extracts a single element from an S-Expression at index `i` and shifts the rest of the list backward so that it no longer contains that `lval*`. It then returns the extracted value. Notice that it doesn't delete the input list. It is like taking an element from a list and popping it out, leaving what remains. This means both the element popped and the old list need to be deleted at some point with `lval_del`.

The `lval_take` function is similar to `lval_pop` but it deletes the list it has extracted the element from. This is like taking an element from the list and deleting the rest. It is a slight variation on `lval_pop` but it makes our code easier to read in some places. Unlike `lval_pop`, only the expression you take from the list needs to be deleted by `lval_del`.

```c
lval* lval_pop(lval* v, int i) {
  /* Find the item at "i" */
  lval* x = v->cell[i];

  /* Shift the memory following the item at "i" over the top of it */
  memmove(&v->cell[i], &v->cell[i+1], sizeof(lval*) * (v->count-i-1));

  /* Decrease the count of items in the list */
  v->count--;

  /* Reallocate the memory used */
  v->cell = realloc(v->cell, sizeof(lval*) * v->count);
  return x;
}

lval* lval_take(lval* v, int i) {
  lval* x = lval_pop(v, i);
  lval_del(v);
  return x;
}
```

We also need to define the evaluation function `builtin_op`. This is like the `eval_op` function used in our previous chapter but modified to take a single `lval*` representing a list of all the arguments to operate on. It needs to do some more rigorous error checking. If any of the inputs are a non-number `lval*` we need to return an error.

First it checks that all the arguments input are numbers. It then pops the first argument to start. If there are no more sub-expressions and the operator is subtraction it performs unary negation on this first number. This makes expressions such as `(- 5)` evaluate correctly.

If there are more arguments it constantly pops the next one from the list and performs arithmetic depending on which operator we're meant to be using. If a zero is encountered on division it deletes the temporary `x`, `y`, and the argument list `a`, and returns an error.

If there have been no errors the input arguments are deleted and the new expression returned.

```c
lval* builtin_op(lval* a, char* op) {

  /* Ensure all arguments are numbers */
  for (int i = 0; i < a->count; i++) {
    if (a->cell[i]->type != LVAL_NUM) {
      lval_del(a);
      return lval_err("Cannot operator on non number!");
    }
  }

  /* Pop the first element */
  lval* x = lval_pop(a, 0);

  /* If no arguments and sub then perform unary negation */
  if ((strcmp(op, "-") == 0) && a->count == 0) { x->num = -x->num; }

  /* While there are still elements remaining */
  while (a->count > 0) {

    /* Pop the next element */
    lval* y = lval_pop(a, 0);

    /* Perform operation */
    if (strcmp(op, "+") == 0) { x->num += y->num; }
    if (strcmp(op, "-") == 0) { x->num -= y->num; }
    if (strcmp(op, "*") == 0) { x->num *= y->num; }
    if (strcmp(op, "/") == 0) {
      if (y->num == 0) {
        lval_del(x); lval_del(y); lval_del(a);
        x = lval_err("Division By Zero!"); break;
      } else {
        x->num /= y->num;
      }
    }

    /* Delete element now finished with */
    lval_del(y);
  }

  /* Delete input expression and return result */
  lval_del(a);
  return x;
}
```

This completes our evaluation functions. We just need to change `main` again so it passes the input through this evaluation before printing it.

```c
lval* x = lval_eval(lval_read(r.output));
lval_println(x);
lval_del(x);
```

Now you should now be able to evaluate expressions correctly in the same way as in the previous chapter. Take a little breather and have a play around with the new evaluation. Make sure everything is working correctly, and the behaviour is as expected. In the next chapter we're going to make great use of these changes to implement some cool new features.

```lispy
lispy> + 1 (* 7 5) 3
39
lispy> (- 100)
-100
lispy>
()
lispy> /
/
lispy> (/ ())
Error: Cannot operator on non number!
lispy>
```


Reference
---------

<div class="panel-group alert alert-warning" id="accordion">
  <a href="src/s_expressions.c">s_expressions.c</a>
</div>

Bonus Marks
-----------

<div class="alert alert-warning">
  <ul class="list-group">
    <li class="list-group-item">&rsaquo; Give an example of a variable in our program that lives on *The Stack*.</li>
    <li class="list-group-item">&rsaquo; Give an example of a variable in our program that points to *The Heap*.</li>
    <li class="list-group-item">&rsaquo; What does the `strcpy` function do?</li>
    <li class="list-group-item">&rsaquo; What does the `realloc` function do?</li>
    <li class="list-group-item">&rsaquo; What does the `memmove` function do?</li>
    <li class="list-group-item">&rsaquo; How does `memmove` differ from `memcpy`?</li>
    <li class="list-group-item">&rsaquo; Extend parsing and evaluation to support the remainder operator `%`.</li>
    <li class="list-group-item">&rsaquo; Extend parsing and evaluation to support decimal types using a `double` field.</li>
  </ul>
</div>
