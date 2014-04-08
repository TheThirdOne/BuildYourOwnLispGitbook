Bonus Projects
==============


Only the Beginning
------------------

Although we've done a lot with our Lisp, it is still some way off from a fully complete, production strength programming language. If you tried to use it for any sufficiently large project there are a number of issues you would eventually run into, and improvements you&#39;d have to make. Solving these problems would be what would bring it more into the scope of a fully fledged programming language.

Here are some of these issues you would likely encounter, potential solutions to these problems, and some other fun ideas for other improvements too. Some may take a few hundred lines of code, others a few thousand. The choice of what to tackle is up to you. If you've become fond of your language you may enjoy doing some of these projects.


Native Types
------------

Currently our language only wraps the native C `long` and `char*` types. This is pretty limiting if you want to do any kind of useful computation. Our operations on these data types are also pretty limited. Ideally our language should wrap all of the native C types and allow for methods of manipulating them. One of the most important additions would be the ability to manipulate decimal numbers. For this you could wrap the `double` type and relevant operations. With more than one number type we need to make sure the arithmetic operators such as `+` and `-` work on them all, and them in combination.

Adding support for native types should be interesting for people wishing to do computation with decimal and floating-point numbers in their language.


User Defined Types
------------------

As well as adding support for native types it would be good to give users the ability to add their own new types, just like how we use structs in C. The syntax or method you use to do this would be up to you. This is a really essential part making our language usable for any reasonably sized project.

This task may be interesting to anyone who has a specific idea of how they would like to develop the language, and what they want a final design to look like.


<div class='pull-right alert alert-warning' style="margin: 15px; text-align: center;">
  <img src="img/list.png" alt="list"/>
  <small>Important List &bull; Play! BE HAPPY and go home.</small>
</div>

List Literal
------------

Some lisps use square brackets `[]` to give a literal notation for lists of evaluated values lists. This syntactic sugar for writing something like `list 100 (+ 10 20) 300`. Instead it lets you write `[100 (+ 10 20) 300]`. In some situations this is clearly nicer, but it does use up the `[]` characters which could possibly be used for more interesting purposes.

This should be a simple addition for people looking to try out adding extra syntax.


Operating System Interaction
----------------------------

One essential part of bootstrapping a language is to give it proper abilities for opening, reading, and writing files. This means wrapping all the C functions such as `fread`, `fwrite`, `fgetc`, etc in Lisp equivalents. This is a fairly straight forward task, but does require writing quite a large number of wrapper functions. This is why we've not done it for our language so far.

On a similar note it would be great to give our language access to whatever operating systems calls are appropriate. We should give it the ability to change directory, list files in a directory and that sort of thing. This is an easy task but again requires a lot of wrapping of C functions. It is essential for any real practical use of this language as something like a scripting language.

People who wish to make use of their language for doing simple scripting tasks and string manipulation may be interested in implementing this project.


Macros
------

Many other Lisps allow you to write things like `(def x 100)` to define the value `100` to `x`. In our lisp this wouldn't work because it would attempt to evaluate the `x` to whatever value was stored as `x` in the environment. In other Lisps these functions are called *macros*, and when encountered they stop the evaluation of their arguments, and manipulate them un-evaluated. They let you write things that look like normal function calls, but actually do complex and interesting things.

These are kind of a fun thing to have in a language. They make it so you can add a little bit of *magic* to some of the workings. In many cases this can make syntax nicer or allow a user to not repeat themselves.

Personally I like how our language handles things like `def` and `if` without resorting to macros, but if you dislike how it works currently, and want it to be more similar to conventional Lisps, this might be something you are interested in implementing.


Variable Hashtable
------------------

At the moment when we lookup variable names in our language we just do a linear search over all of the variables in the environment. This gets more and more inefficient the more variables we have defined.

A more efficient way to do this is to implement a *Hash Table*. This technique converts the variable name to some integer and uses this to index into an array of a known size to find the value associated with this symbol. This is a really important data structure in programming and will crop up everywhere because of its fantastic performance under heavy loads.

Anyone who is interested in learning more about data structures and algorithms would be smart to take a shot at implementing this data structure or one of its variations.


Pool Allocation
---------------

Our Lisp is very simple, it is not fast. Its performance is relative to some scripting languages such as Python and Ruby. Most of the performance overhead in our program comes from the fact that doing almost anything requires us to construct and destruct `lval`. We therefore call `malloc` very often. This is a slow function as it requires the operating system to do some management for us. When doing calculations there is lots of copying, allocation and deallocation of `lval` types.

If we wish to reduce this overhead we need to lower the number of calls to `malloc`. One method of doing this is to call `malloc` once at the beginning of the program, allocating a large pool of memory. We should then replace all our `malloc` calls with calls to some function that slices and dices up this memory for use in the program. This means that we are emulating some of the behaviour of the operating system, but in a faster local way. This idea is called *memory pool allocation* and is a common technique used in game development, and other performance sensitive applications.

This can be tricky to implement correctly, but conceptually does not need to be complex. If you want a quick method for getting large gains in performance looking into this might interest you.


Garbage Collection
------------------

Almost all other implementations of Lisps assign variables differently to ours. They do not store a copy of a value in the environment, but actually a pointer, or reference, to it directly. Because pointers are used, rather than copies, just like in C, there is much less overhead required when using large data structures.

<div class='pull-left alert alert-warning' style="margin: 15px; text-align: center;">
  <img src="img/garbage.png" alt="garbage"/>
  <small>Garbage Collection &bull; Pick up that can.</small>
</div>

If we store pointers to values, rather than copies, we need to ensure that the data pointed to is not deleted before some other value tries to make use of it. Instead we want it to get deleted when there are no longer any references to it. One method to do this, called *Mark and Sweep*, is to monitor those values that are in the environment, as well as every value that has been allocated. When a variable is put into the environment it, and everything it references, is *marked*. Then, when we wish to free memory, we can then iterate over every value that has been allocated, and delete any that are not marked.

This is called *Garbage Collection* and is an integral part to many programming languages. As with pool allocation, implementing a *Garbage Collector* does not need to be complicated, but it does need to be done carefully, in particularly if you wish to make it efficient. Implementing this would be essential to making this language practical for working with large amounts of data. A particularly good tutorial on implementing a garbage collector in C can be found <a href="http://journal.stuffwithstuff.com/2013/12/08/babys-first-garbage-collector/">here</a>.

This should interest anyone who is concerned with the language's performance and wishes to change the semantics of how variables are stored and modified in the language.


Tail Call Optimisation
----------------------

Our programming language uses *recursion* to do its looping. This is conceptually a really neat way to do it, but practically it is quite poor. Recursive functions call themselves to collect all of the partial results of a computation, and only then combine all the results together. This is a wasteful way of doing computation when partial results can be accumulated as some total over a loop. This is particularly problematic for loops that are intended to run for many, or infinite, iterations.

Some recursive functions can be automatically converted to corresponding `while` loops, which accumulate totals step by step, rather than altogether. This automatic conversion is called *tail call optimisation* and is an essential optimisation for programs that do a lot of looping using recursion.

People who are interested in compiler optimisations and the correspondences between different forms of computation might find this project interesting.


Lexical Scoping
---------------

When our language tries to lookup a variable that has been undefined it throws an error. It would be better if it could tell us which variables are undefined before evaluating the program. This would let us avoid typos and other annoying bugs. Finding these issues before the program is run is called *lexical scoping*, and uses the rules for variable definition to try and infer which variables are defined and which aren't at each point in the program, without doing any evaluation.

This could be a difficult task to get exactly right, but should be interesting to anyone who wants to make their programming language more safe to use, and less bug-prone.


<div class='pull-right alert alert-warning' style="margin: 15px; text-align: center;">
  <img src="img/static.png" alt="static"/>
  <small>Static Electricity &bull; A hair-raising alternative.</small>
</div>

Static Typing
-------------

Every value in our program has an associated type with it. This we know before any evaluation has taken place. Our builtin functions also only take certain types as input. We should be able to use this information to infer the types of new user defined functions and values. We can also use this information to check that functions are being called with the correct types before we run the program. This will reduce any errors stemming from calling functions with incorrect types before evaluation. This checking is called *static typing*.

Type systems are a really interesting and fundamental part of computer science. They are currently the best method we know of detecting errors before running a program. Anyone interesting in programming language safety and type systems should find this project really interesting.


Conclusion
----------

Many thanks for following along with this book. I hope you've found something of interest in its pages. If you did enjoy it please tell your friends about it! If you are going to continue developing your language then best of luck and I hope you learn many more cool things about C, programming languages, and computer science.

Most of all I hope you've had fun building your own Lisp. Until next time!