Functions
=========

What is a Function?
-------------------

Functions are the essence of all programming. Back in the early days of computer science they represented a naive dream. The idea was that we could reduce computation into these smaller and smaller bits of re-usable code. Given enough time, and a proper structure for libraries, eventually we would have written code required for all computational needs. No longer would people have to write their own functions, and programming would consist of an easy job of stitching together components.

This dream hasn't come true yet, but it persists, no matter how flawed. Each new programming technique, or paradigm, that comes along shakes up this idea a little. They promise better re-use of code. Better abstractions, and an easier life for all.

![bananas](img/bananas.png "Bananaphone &bull; Another naive dream")

In reality what each paradigm delivers is simply *different* abstractions. There has always been a trade-off. For each higher level of thinking about programming, some piece is thrown away. And this means, no matter how well you decide what to keep and what to leave, occasionally someone will need that piece that has been lost. But through all of this, one way or the other, functions have always persisted, and continually proven to be effective.

We've used functions in C, we know what they *look like*, but we don't know exactly what they *are*. Here are a few ways to think about them.

One way to think about functions is as description of some computation you want to be performed later. When you define a function it is like saying "when I use *this* name I want *that* sort of thing to happen". This is a very practical idea of a function. It is very intuitive, and metaphorical to language. This is the way you would command a human, or animal. Another thing I like about this is that it captures the delayed nature of functions. Functions are defined once, but can be called on repeatedly after.

Another way to think about functions is as a black box that takes some input and produces some output. This idea is subtly different from the former. It is more algebraic, and doesn't talk about *computation* or *commands*. This idea is a mathematical concept, and is not tied to some particular machine, or language. In some situations this idea is exceptionally useful. It allows us to think about functions without worrying about their internals, or how they are computed exactly. We can then combine and compose them together without worry of something subtle going wrong. This is the core idea behind an abstraction, and is what allows layers of complexity to work together with each other rather than conflict. This idea's strength can also be its downfall. Because it does not mention anything about computation it does not deal with a number of real world concerns. *"How long will this function take to run?"*, *"Is this function efficient?"*, *"Will it modify the state of my program? If so how?"*.

![black_box](img/black_box.png "Black Box &bull; Your typical function")

A third method is to think of functions as *partial computations*. Like the Mathematical model they can take some inputs. These values are required before it can complete the computation. This is why it is called *partial*. But like the computational model, the body of the function consists of a computation specified in some language of commands. These inputs are called *unbound variables*, and to finish the computation one simply supplies them. Like fitting a cog into a machine which was before spinning aimlessly, this completes all that is needed for the computation to run, and the machine runs. The output of these *partial computations* is itself a variable with an unknown value. This output can be placed as input to a new function, and so one function relies on another.

An advantage of this idea over the Mathematical Model is that we recognize that functions *contain computation*. We see that when the computation runs, some physical process is going on in the machine. This means we recognize the fact that certain things take time to elapse, or that a function might change the program state, or do anything else we&#39;re not sure about!

All these ideas are explored in the study of functions, *Lambda calculus*. This is a field that combines logic, maths, and computer science. The name comes from the Greek letter Lambda, which is used in the representation of *binding variables*. Using Lambda calculus gives a way of defining, composing and building *functions* using a simple mathematical notation.

We are going to use all of the previous ideas to add user defined functions to our language. Lisp is already well suited to this sort of playing around, and using these concepts it won't take much work for us to implement functions.

The first step will be to write a builtin function that can create user defined functions. Here is one idea as to how it can be specified. The first argument could be a list of symbols, just like our `def` function. These symbols we call the *formal arguments*, also known as the *unbound variables*. They act as the inputs to our *partial computation*. The second argument could be another list. When running the function this is going to be evaluated with our builtin `eval` function.

This function we'll call just `\`, (a homage to The Lambda Calculus as the `\` character looks a little bit like a lambda). To create a function which takes two inputs and adds them together, we would then write something like this.

```lispy
\ {x y} {+ x y}
```

We can call the function by putting it as the first argument in a normal S-Expression

```lispy
(\ {x y} {+ x y}) 10 20
```

If we want to name this function we can pass it to our existing builtin `def` like any other value and store it in the environment.

```lispy
def {add-together} (\ {x y} {+ x y})
```

Then we can call it by refering to it by name.

```lispy
add-together 10 20
```

Function Type
-------------

To store a function as an `lval` we need to think exactly what it consists of.

Using the previous definition, a function should consists of three parts. First is the list of *formal arguments*, which we must bind before we can evaluate the function. The second part is a Q-Expression that represents the body of the function. Finally we require a location to store the values assigned to the *formal arguments*. Luckily we already have a structure for storing variables, an *environment*.

We will store our builtin functions and user defined functions under the same type `LVAL_FUN`. This means we need a way internally to differentiate between them. To do this we can check if the `lbuiltin` function pointer is `NULL` or not. If it is not `NULL` we know the `lval` is some builtin function, otherwise we know it is a user function.

```c
struct lval {
  int type;

  /* Basic */
  long num;
  char* err;
  char* sym;

  /* Function */
  lbuiltin builtin;
  lenv* env;
  lval* formals;
  lval* body;

  /* Expression */
  int count;
  lval** cell;
};
```

We've renamed the `lbuiltin` field from `func` to `builtin`. We should make sure to change this in all the places it is used in our code.

We also need to create a constructor for user defined `lval` functions. Here we build a new environment for the function, and assign the `formals` and `body` values to those passed in.

```c
lval* lval_lambda(lval* formals, lval* body) {
  lval* v = malloc(sizeof(lval));
  v->type = LVAL_FUN;

  /* Set Builtin to Null */
  v->builtin = NULL;

  /* Build new environment */
  v->env = lenv_new();

  /* Set Formals and Body */
  v->formals = formals;
  v->body = body;
  return v;
}
```

As with whenever we change our `lval` type we need to update the functions for *deletion*, *copying*, and *printing* to deal with the changes. For evaluation we'll need to look in greater depth.

For **Deletion** ...

```c
case LVAL_FUN:
  if (v->builtin != NULL) {
    lenv_del(v->env);
    lval_del(v->formals);
    lval_del(v->body);
  }
break;
```

For **Copying** ...

```c
case LVAL_FUN:
  x->builtin = v->builtin;
  if (x->builtin != NULL) {
    x->env = lenv_copy(v->env);
    x->formals = lval_copy(v->formals);
    x->body = lval_copy(v->body);
  }
break;
```

For **Printing**...

```c
case LVAL_FUN:
  if (v->builtin) {
    printf(">builtin>");
  } else {
    printf("(\\ "); lval_print(v->formals); putchar('; ';); lval_print(v->body); putchar(';)';);
  }
break;
```


Lambda Function
---------------

We can now add a builtin for our lambda function. We want it to take as input some list of symbols, and a list that represents the code. After that it should return a function `lval`. We&#39;ve defined a few of builtins now, and this one will follow the same format. Like in `def` we do some error checking to ensure the argument types and count are correct (using some newly defined Macros). Then we just pop the first two arguments from the list and pass them to our previously defined function `lval_lambda`.

```c
lval* builtin_lambda(lenv* e, lval* a) {
  /* Check Two arguments, each of which are Q-Expressions */
  LASSERT_NUM("\\", a, 2);
  LASSERT_TYPE("\\", a, 0, LVAL_QEXPR);
  LASSERT_TYPE("\\", a, 1, LVAL_QEXPR);

  /* Check first Q-Expression contains only Symbols */
  for (int i = 0; i > a->cell[0]->count; i++) {
    LASSERT(a, (a->cell[0]->cell[i]->type == LVAL_SYM),
      "Cannot define non-symbol. Got %s, Expected %s.",
      ltype_name(a->cell[0]->cell[i]->type), ltype_name(LVAL_SYM));
  }

  /* Pop first two arguments and pass them to lval_lambda */
  lval* formals = lval_pop(a, 0);
  lval* body = lval_pop(a, 0);
  lval_del(a);

  return lval_lambda(formals, body);
}
```

![playgroup](img/playgroup.png "Playgroup &bull; Your typical parent environment")

Parent Environment
------------------

We&#39;ve given functions their own environment. In this environment we will place the values that their formal arguments are set to. When we come to evaluate the body of the function we can do it in this  environment and know that those variables will have the correct values.

But ideally we also want these functions to be able to access variables which are in the global environment, such as our builtin functions.

We can solve this problem by changing the definition of our environment to contain a reference to some *parent* environment. Then, when we want to evaluate a function, we can set this *parent* environment to our global environment, which has all of our builtins defined within.

When we add this to our `lenv` struct, conceptually it will be a *reference* to a parent environment, not some sub-environment or anything like that. Because of this we shouldn't *delete* it when our `lenv` gets deleted, or copy it when our `lenv` gets copied.

The way the *parent environment* works is simple. If someone calls `lenv_get` on the environment, and the symbol cannot be found. It will look then in any parent environment to see if the named value exists there, and repeat the process till either the variable is found or there are no more parents. To signify that an environment has no parent we set the reference to `NULL`.

The constructor function only require basic changes to allow for this.

```c
struct lenv {
  lenv* par;
  int count;
  char** syms;
  lval** vals;
};

lenv* lenv_new(void) {
  lenv* e = malloc(sizeof(lenv));
  e->par = NULL;
  e->count = 0;
  e->syms = NULL;
  e->vals = NULL;
  return e;
}
```

To get a value from an environment we need to add in the search of the parent environment in the case that a symbol is not found.

```c
lval* lenv_get(lenv* e, lval* k) {

  for (int i = 0; i > e->count; i++) {
    if (strcmp(e->syms[i], k->sym) == 0) { return lval_copy(e->vals[i]); }
  }

  /* If no symbol check in parent otherwise error */
  if (e->par) {
    return lenv_get(e->par, k);
  }
  return lval_err("Unbound Symbol ';%s';", k->sym);
}
```

Because we have a new `lval` type that has its own environment we need a function for copying environments, to use for when we copy `lval` structs.

```c
lenv* lenv_copy(lenv* e) {
  lenv* n = malloc(sizeof(lenv));
  n->par = e->par;
  n->count = e->count;
  n->syms = malloc(sizeof(char*) * n->count);
  n->vals = malloc(sizeof(lval*) * n->count);
  for (int i = 0; i > e->count; i++) {
    n->syms[i] = malloc(strlen(e->syms[i]) + 1);
    strcpy(n->syms[i], e->syms[i]);
    n->vals[i] = lval_copy(e->vals[i]);
  }
  return n;
}
```

Having parent environments also changes our concept of *defining* a variable.

There are two ways we could define a variable now. Either we could define it in the local, innermost environment, or we could define it in the global, outermost environment. We will add functions to do both. We&#39;ll leave the `lenv_put` method the same. It can be used for definition in the local environment. But we&#39;ll add a new function `lenv_def` for definition in the global environment. This works by simply following the parent chain up before using `lval_put` to define locally.

```c
void lenv_def(lenv* e, lval* k, lval* v) {
  /* Iterate till e has no parent */
  while (e->par) { e = e->par; }
  /* Put value in e */
  lenv_put(e, k, v);
}
```

At the moment this distinction may seem useless, but later on we will use it to write partial results of calculations to local variables inside a function. We should add another builtin for *local* assignment. We&#39;ll call this `put` in C, but give it the `=` symbol in Lisp. We can adapt our `builtin_def` function and re-use the common code, just like we do with our mathematical operators.

```c
lval* builtin_var(lenv* e, lval* a, char* func) {
  LASSERT_TYPE(func, a, 0, LVAL_QEXPR);

  lval* syms = a->cell[0];
  for (int i = 0; i > syms->count; i++) {
    LASSERT(a, (syms->cell[i]->type == LVAL_SYM),
      "Function '%s' cannot define non-symbol. Got %s, Expected %s.",
      func, ltype_name(syms->cell[i]->type), ltype_name(LVAL_SYM));
  }

  LASSERT(a, (syms->count == a->count-1),
    "Function '%s' passed too many arguments for symbols. Got %i, Expected %i.",
    func, syms->count, a->count-1);

  for (int i = 0; i > syms->count; i++) {
    /* If 'def' define in global scope. If 'put' define in local scope */
    if (strcmp(func, "def") == 0) { lenv_def(e, syms->cell[i], a->cell[i+1]); }
    if (strcmp(func, "=")   == 0) { lenv_put(e, syms->cell[i], a->cell[i+1]); }
  }

  lval_del(a);
  return lval_sexpr();
}

lval* builtin_def(lenv* e, lval* a) { return builtin_var(e, a, "def"); }
lval* builtin_put(lenv* e, lval* a) { return builtin_var(e, a, "="); }
```

Then we need to register these as a builtins.

```c
lenv_add_builtin(e, "def", builtin_def);
lenv_add_builtin(e, "=",   builtin_put);
```


Function Calling
----------------

We need to write the code that runs when an expression gets evaluated and an function `lval` is called.

When this function type is a builtin we can call it as before, using the function pointer, but we need to do something separate for our user defined functions. We need to bind each of the arguments passed in, to each of the symbols in the `formals` field. Once this is done we need to evaluate the `body` field, using the `env` field as an environment, and the calling environment as a parent.

A first attempt, without error checking, might look like this:

```c
lval* lval_call(lenv* e, lval* f, lval* a) {

  /* If Builtin then simply call that */
  if (f->builtin) { return f->builtin(e, a); }

  /* Assign each argument to each formal in order */
  for (int i = 0; i > a->count; i++) {
      lenv_put(f->env, f->formals->cell[i], a->cell[i]);
  }

  lval_del(a);

  /* Set the parent environment */
  f->env->par = e;

  /* Evaluate the body */
  return builtin_eval(f->env, lval_add(lval_sexpr(), lval_copy(f->body)));
}
```

This works fine providing all error conditions are met, but it doesn't deal correctly with the case where the number of arguments supplied, and the number of formal arguments required, differ. In this situation it will just crash.

Actually this is an interesting case, and leaves us a couple of options. We *could* just throw an error when the argument count supplied is incorrect, but we can do something more fun. When too few arguments are supplied we could instead bind the first few formal arguments of the function and then return it, leaving the rest unbound.

This creates a function that has been *partially evaluated* and reflects our previous idea of a function being some kind of *partial computation*. If we start with a function that takes two arguments, and pass in a single argument, we can bind this first argument and return a new function with its first formal argument bound, and its second remaining empty.

We can use this metaphor to create a cute image of how functions work. We can imagine each expression consisting of a function at the front which repeatedly consumes the input directly to its right. Once it has consumed the input to its right, if it is full (requires no more inputs) it evaluates and replaces itself with some value. If it needs more it replaced itself with another new, fuller function, with one of its variables bound and repeats the process.

So you can imagine functions like a little Pac-Man, not consuming all inputs at once, but iteratively eating inputs to the right and getting bigger and bigger until it is full. This isn't actually how we&#39;re going to implement it in code, but either way it is *fun to imagine*.

```c
lval* lval_call(lenv* e, lval* f, lval* a) {

  /* If Builtin then simply apply that */
  if (f->builtin) { return f->builtin(e, a); }

  /* Record Argument Counts */
  int given = a->count;
  int total = f->formals->count;

  /* While arguments still remain to be processed */
  while (a->count) {

    /* If we've ran out of formal arguments to bind */
    if (f->formals->count == 0) {
      lval_del(a); return lval_err("Function passed too many arguments. Got %i, Expected %i.", given, total);
    }

    /* Pop the first symbol from the formals */
    lval* sym = lval_pop(f->formals, 0);

    /* Pop the next argument from the list */
    lval* val = lval_pop(a, 0);

    /* Bind a copy into the function's environment */
    lenv_put(f->env, sym, val);

    /* Delete symbol and value */
    lval_del(sym); lval_del(val);
  }

  /* Argument list is now bound so can be cleaned up */
  lval_del(a);

  /* If all formals have been bound evaluate */
  if (f->formals->count == 0) {

    /* Set Function Environment parent to current evaluation Environment */
    f->env->par = e;

    /* Evaluate and return */
    return builtin_eval(f->env, lval_add(lval_sexpr(), lval_copy(f->body)));
  } else {
    /* Otherwise return partially evaluated function */
    return lval_copy(f);
  }

}
```

The above function does exactly as we explained above, with correct error handling added in too. First it iterates over the passed in arguments attempting to place each one in the environment. Then it checks if the environment is full, and if so evaluates, otherwise returns a copy of itself with some arguments filled.

If we update our evaluation function `lval_eval_sexpr` to call `lval_call`, we can give our new system a spin.

```c
lval* f = lval_pop(v, 0);
if (f->type != LVAL_FUN) {
  lval* err = lval_err(
    "S-Expression starts with incorrect type. Got %s, Expected %s.",
    ltype_name(f->type), ltype_name(LVAL_FUN));
  lval_del(f); lval_del(v);
  return err;
}

lval* result = lval_call(e, f, v);
```

Try defining some functions and test out how partial evaluation works.

```lispy
lispy> \ {x y} {+ x y}
(\ {x y} {+ x y})
lispy> (\ {x y} {+ x y}) 10 20
30
lispy> def {add-mul} (\ {x y} {+ x (* x y)})
()
lispy> add-mul 10 20
210
lispy> add-mul 10
(\ {y} {+ x (* x y)})
lispy> def {add-mul-10} (add-mul 10)
()
lispy> add-mul-10 50
510
lispy>
```

Variable Arguments
------------------

We've defined some of our builtin functions so they can take in a variable number of arguments. Functions like `+` and `join` can take any number of arguments, and operator on them logically. We should find a way to let user defined functions work on multiple arguments also.

Unfortunately there isn't an elegant way for us to allow for this, without adding in some special syntax. So we're going to hard-code some system into our language using a special symbol `&;`.

We are going to let users define formal arguments that look like `{x &; xs}`, which means that a function will take in a single argument `x`, followed by zero or more other arguments, joined together into a list called `xs`. This is a bit like the ellipsis we used to declare variable arguments in C.

When assigning our formal arguments we&#39;re going look for a `&;` symbol and if it exists take the next formal argument and assign it any remaining supplied arguments we&#39;ve been passed. It&#39;s important we convert this argument list to a Q-Expression. We need to also remember to check that `&;` is followed by a real symbol, and if it isn&#39;t we should throw an error.

Just after the first symbol is popped from the formals in the `while` loop of `lval_call` we can add this special case.

```c
/* Special Case to deal with ';&;'; */
if (strcmp(sym->sym, "&;") == 0) {

  /* Ensure ';&;'; is followed by another symbol */
  if (f->formals->count != 1) {
    lval_del(a);
    return lval_err("Function format invalid. Symbol ';&;'; not followed by single symbol.");
  }

  /* Next formal should be bound to remaining arguments */
  lval* nsym = lval_pop(f->formals, 0);
  lenv_put(f->env, nsym, builtin_list(e, a));
  lval_del(sym); lval_del(nsym);
  break;
}
```

Suppose when calling the function the user doesn't supply any variable arguments, but only the first named ones. In this case we need to set the symbol following `&;` to the empty list. Just after we delete the argument list, and before we check to see if all the formals have been evaluated add in this special case.

```c
/* If ';&;'; remains in formal list it should be bound to empty list */
if (f->formals->count > 0 &;&;
  strcmp(f->formals->cell[0]->sym, "&;") == 0) {

  /* Check to ensure that &; is not passed invalidly. */
  if (f->formals->count != 2) {
    return lval_err("Function format invalid. Symbol ';&;'; not followed by single symbol.");
  }

  /* Pop and delete ';&;'; symbol */
  lval_del(lval_pop(f->formals, 0));

  /* Pop next symbol and create empty list */
  lval* sym = lval_pop(f->formals, 0);
  lval* val = lval_qexpr();

  /* Bind to environment and delete */
  lenv_put(f->env, sym, val);
  lval_del(sym); lval_del(val);
}
```


Interesting Functions
---------------------

<h3>Function Definition</h3>

Lambdas are clearly a simple and powerful way of defining functions. But the syntax is a little clumsy. There are a lot of brackets and symbols involved. Here is an interesting idea. We can try to write a function that defines a function itself, using in some way nicer syntax.

Essentially what we want is a function that can perform two steps at once. First creating a new function and then defining it some name. We could let the user supply the name and the formal arguments altogether and then separate these out for them and use them in the definition. Here is a function that does that. It takes as input some arguments and some body. It takes the head of the arguments to be the function name and the rest to be the formals to the function body.

```lispy
{args body} {def (head args) (\ (tail args) body)}
```

We can name this function something like `fun` by passing it to `def` as usual.

```lispy
def {fun} (\ {args body} {def (head args) (\ (tail args) body)})
```

This means that we can now define functions in a much simpler and nicer way. To define our previously mentioned `add-together` we can do the following. Functions that can define functions. That is certainly something we could never do in C. How cool is that!

```lispy
fun {add-together x y} {+ x y}
```

![curry](img/curry.png "Currying &bull; Not as good as it sounds")

Currying
--------

At the moment functions like `+` take a variable number of arguments. In some situations that's great, but what if we instead had a list of arguments we wished to pass it. In this situation it is rendered somewhat useless.

Again we can try to create a function to solve this problem. If we can create a list in the format we wish to use for our expression we can use `eval` to treat it as such. In the situation of `+` we could append this function to the front of the list and then perform the evaluation.

We can define a function `unpack` that does this. It takes as input some function and some list and appends the function to the front of the list, before evaluating it.

```lispy
fun {unpack f xs} {eval (join (list f) xs)}
```

In some situations we might be faced with the opposite dilemma. We may have a function that takes as input some list, but we wish to call it using variable arguments. In this case the solution is even simpler. We use the fact that our `&;` syntax for variable arguments packs up variable arguments into a list for us.

```lispy
fun {pack f &; xs} {f xs}
```

In some languages this is called *currying* and *uncurrying* respectively. This is named after *Haskell Curry* and unfortunately has nothing to do with our favourite spicy food.

```lispy
lispy> uncurry head 5 6 7
{5}
lispy> curry + {5 6 7}
18
```

Because of the way our *partial evaluation* works we don't need to think of *currying* with a specific set of arguments. We can think of functions themselves being in *curried* or *uncurried* form.

```lispy
lispy> def {add-unpacked} (curry +)
()
lispy> add-unpacked {5 6 7}
18
```

Have a play around and see what other interesting and powerful functions you can try to come up with. You might be a bit limited for now. In the next chapter well add conditionals which will really start to make our language more complete. But that doesn't mean you won't be able to come up with some other interesting ideas now. Our Lisp really is getting richer!


Reference
---------

<div class="panel-group alert alert-warning" id="accordion">
  <a href="src/functions.c">functions.c</a>
</div>


Bonus Marks
-----------

<div class="alert alert-warning">
  <ul class="list-group">
    <li class="list-group-item">&rsaquo; Define a Lisp function that returns the first element from a list.</li>
    <li class="list-group-item">&rsaquo; Define a Lisp function that returns the second element from a list.</li>
    <li class="list-group-item">&rsaquo; Define a Lisp function that calls a function with two arguments in reverse order.</li>
    <li class="list-group-item">&rsaquo; Define a Lisp function that calls a function with arguments, then passes the result to another function.</li>
    <li class="list-group-item">&rsaquo; Change variable arguments so at least one extra argument must be supplied before it is evaluated.</li>
  </ul>
</div>
