Evaluation
==========


Trees
-----

Now we can read input, and we have it structured internally, but we are still unable to evaluate it. In this chapter we add the code that evaluates this structure and actually performs the computations encoded within.

This internal structure is what we saw printed out by the program in the previous chapter. It is called an *Abstract Syntax Tree*, and it represents the structure of the program based on the input entered by the user. At the leaves of this tree are numbers and operators - the actual data to be processed. At the branches are the rules used to produce this part of the tree - the information on how to traverse and evaluate it.

![tree](img/tree.png "Abstract Christmas Tree &bull; A seasonal variation")

Before working out exactly how we are going to do this traversal, lets see exactly how this structure is defined internally. If we peek inside `mpc.h` we can have a look at the definition of `mpc_ast_t`, which is the data structure we got from the parse.

```c
typedef struct mpc_ast_t {
  char* tag;
  char* contents;
  int children_num;
  struct mpc_ast_t** children;
} mpc_ast_t;
```

This struct has a number of fields we can access. Lets take a look at them one by one.

The first field is `tag`. When we printed out the tree this was the information that precluded the contents of the node. It was a string containing a list of all the rules used to parse that particular item. For example `expr|number|regex`.

This `tag` field is going to be important as it lets us see what type of thing the node is.

The second field is `contents`. This will contain the actual contents of the node such as `'*'`, `'('` or `'5'`. You'll notice for branches this is empty, but for leaves we can use it to find the operator or number to use.

Finally we see two fields that are going to help us traverse the tree. These are `children_num` and `children`. The first field tells us how many children a node has, and the second is an array of these children.

The type of the `children` field is `mpc_ast_t**`. This is a double pointer type. It isn't as scary as it looks and will be explained in greater detail in later chapters. For now you can think of it as a list of the child nodes of the this tree.

We can access a child node, by accessing this field using array notation. This is done by writing the field name `children` and suffixing it with square brackets containing the index of the child to access. For example to access the first child of the node we can use `children[0]`. Notice that C counts its array indices from `0`.

Because the type `mpc_ast_t*` is a *pointer* to a struct, there is a slightly different syntax to access its fields. We need to use an arrow `->` instead of a dot `.`. There is no fundamental reason for this switch in operators, so for now just remember that field access of pointer types uses an arrow.

```c
/* Load AST from output */
mpc_ast_t* a = r.output;
printf("Tag: %s\n", a->tag);
printf("Contents: %s\n", a->contents);
printf("Number of children: %i\n", a->children_num);

/* Get First Child */
mpc_ast_t* c0 = a->children[0];
printf("First Child Tag: %s\n", c0->tag);
printf("First Child Contents: %s\n", c0->contents);
printf("First Child Number of children: %i\n", c0->children_num);
```


Recursion
---------

There is a funny thing about this tree structure. It refers to itself. Each of its children are themselves trees again, and the children of those children are trees yet again. Just like our languages, and re-write rules, data in this structure contains repeated substructures, that resemble their parents.

![recursion](img/recursion.png "Recursion &bull; Dangerous in a fire")

This pattern of repeated substructures could go on and on. Clearly if we want a function which can work on all possible trees we can't look just a couple of nodes down, we have to define it to work on trees of any depth.

Luckily we can do this, by exploiting the nature of how these substructures repeat, and using a technique called *recursion*.

Put simply a *recursive function* is one that calls itself as some part of its calculation.

It sounds weird for a function to be defined in terms of itself. But consider that functions can give different outputs when supplied with different inputs. If we give some changed, or different inputs to a recursive call to the same function, and provide a way for this function to not call itself again under certain conditions, we can be more confident this *recursive function* is doing something useful.

As an example we can write a recursive function which will count the number of nodes in our tree structure.

To begin we work out how it will act in the most simple case, if the input tree has no children. In this case we know the result is simply one. Now we can go on to define the more complex case, if the tree has one or more children. In this case the result will be one (for the node itself), plus the number of nodes in all of those children.

But how do we find the number of nodes in all of the children? Well we can use the function we are in the process defining! *Yeah, Recursion.*

In C we might write it something like this.

```c
int number_of_nodes(mpc_ast_t* t) {
  if (t->children_num == 0) { return 1; }
  if (t->children_num >= 1) {
    int total = 1;
    for (int i = 0; i < t->children_num; i++) {
      total = total + number_of_nodes(t->children[i]);
    }
    return total;
  }
}
```

Recursive functions are weird because they require an odd leap of faith. First we have to assume we have some function which does something correctly already, and then we have to go about using this function, to write the initial function we assumed we had!

Like most things, recursive functions almost always end up following a similar pattern. First a *base case* is defined. This is the case that ends the recursion, such as `t->children_num == 0` in our previous example. After this the *recursive case* is defined, such as `t->children_num >= 1` in our previous example, which breaks down a computation into smaller parts, and calls itself recursively to compute those parts, before combining them together.

Recursive functions can take some thought, so pause now and ensure you understand them before continuing onto other chapters because we'll be making good use of them in the rest of the book. If you are still uncertain, you can check out some of the bonus marks for this chapter to practice.


Evaluation
----------

To evaluate the parse tree we are going to write a recursive function. But before we get started, let us try and see what observations we can make about the structure of the tree we get as input. Try printing out some expressions using your program from the previous chapter. See what you can notice.

```lispy
lispy> + 1 (* 5 4)
>:
  regex:
  operator|char: '+'
  expr|number|regex: '1'
  expr|>:
    char: '('
    operator|char: '*'
    expr|number|regex: '5'
    expr|number|regex: '4'
    char: ')'
  regex:
```

One observation is that if a node is tagged with `number` it is always a number, has no children, and we can just convert the contents to an integer. This will act as the *base case* in our recursion.

If a node is tagged with `expr`, and is *not* a `number`, we need to look at its second child (the first child is always `'('`) and see which operator it is. Then we need to apply this operator to the *evaluation* of the remaining children, excluding the final child which is always `')'`. This is our *recursive case*. This also needs to be done for the root node.

When we evaluate our tree, just like when counting the nodes, we'll need to accumulate the result. To represent this result we'll use the C type `long` which means a *long* *integer*.

To detect the tag of a node, or to get a number from a node, we will need to make use of the `tag` and `contents` fields. These are *string* fields, so we are going to have to learn a couple of string functions first.

<table class='table'>
  <tr><td>`atoi`</td><td>Converts a `char*` to a `long`.</td></tr>
  <tr><td>`strcmp`</td><td>Takes as input two `char*` and if they are equal it returns `0`.</td></tr>
  <tr><td>`strstr`</td><td>Takes as input two `char*` and returns a pointer to the location of the second in the first, or `0` if the second is not a sub-string of the first.</td></tr>
</table>

We can use `strcmp` to check which operator to use, and `strstr` to check if a tag contains some substring. Altogether our recursive evaluation function looks like this.

```c
long eval(mpc_ast_t* t) {

  /* If tagged as number return it directly, otherwise expression. */
  if (strstr(t->tag, "number")) { return atoi(t->contents); }

  /* The operator is always second child. */
  char* op = t->children[1]->contents;

  /* We store the third child in `x` */
  long x = eval(t->children[2]);

  /* Iterate the remaining children, combining using our operator */
  int i = 3;
  while (strstr(t->children[i]->tag, "expr")) {
    x = eval_op(x, op, eval(t->children[i]));
    i++;
  }

  return x;
}
```

We can define the `eval_op` function as follows. It takes in a number, an operator string, and another number. It tests for which operator is passed in, an performs the corresponding C operation on the inputs.

```c
/* Use operator string to see which operation to perform */
long eval_op(long x, char* op, long y) {
  if (strcmp(op, "+") == 0) { return x + y; }
  if (strcmp(op, "-") == 0) { return x - y; }
  if (strcmp(op, "*") == 0) { return x * y; }
  if (strcmp(op, "/") == 0) { return x / y; }
  return 0;
}
```


Printing
--------

Instead of printing the tree we now want to print the result of the evaluation. Therefore we need to pass the tree into our `eval` function, and print the result we get using `printf` and the specifier `%li`, which is used for `long` type.

We also need to remember to delete the output tree after we are done evaluating it.

```c
long result = eval(r.output);
printf("%li\n", result);
mpc_ast_delete(r.output);
```

If all of this is successful we should be able to do some basic maths with our new programming language!

```lispy
Lispy Version 0.0.0.0.3
Press Ctrl+c to Exit

lispy> + 5 6
11
lispy> - (* 10 10) (+ 1 1 1)
97
lispy> - (/ 10 2) 20
-15
lispy>
```


Reference
---------

<div class="panel-group alert alert-warning" id="accordion">
  <a href="src/evaluation.c">evaluation.c</a>
</div>

Bonus Marks
-----------

<div class="alert alert-warning">
  <ul class="list-group">
    <li class="list-group-item">&rsaquo; Write a recursive function to compute the number of leaves of a tree.</li>
    <li class="list-group-item">&rsaquo; Write a recursive function to compute the number of branches of a tree.</li>
    <li class="list-group-item">&rsaquo; Write a recursive function to compute the most number of children spanning from one branch of a tree.</li>
    <li class="list-group-item">&rsaquo; How would you use `strstr` to see if a node was tagged as an `expr`.</li>
    <li class="list-group-item">&rsaquo; How would you use `strcmp` to see if a node had the contents `'('` or `')'`.</li>
    <li class="list-group-item">&rsaquo; Add the operator `%`, which returns the remainder of division. For example `% 10 6` is `4`.</li>
    <li class="list-group-item">&rsaquo; Add the operator `^`, which raises one number to another. For example `^ 4 2` is `16`.</li>
    <li class="list-group-item">&rsaquo; Add the function `min`, which returns the smallest number. For example `min 1 5 3` is `1`.</li>
    <li class="list-group-item">&rsaquo; Add the function `max`, which returns the biggest number. For example `max 1 5 3` is `5`.</li>
    <li class="list-group-item">&rsaquo; Change the minus operator `-` so that when it receives one argument it negates it.</li>
  </ul>
</div>