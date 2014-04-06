Basics
======


Overview
---------

<!--
<div class='pull-right alert alert-warning' style="margin: 15px; text-align: center;">
  <img src="static/img/crash.png" alt="crash"/>
  <small>Crash &bull; of course</small>
</div>
-->

<!--
<div class='pull-right alert alert-warning' style="margin: 15px; text-align: center;">
  <img src="static/img/breakfast.png" alt="breakfast"/>
  <small>Fried Breakfast &bull; The basics in life.</small>
</div>
-->

<div class='pull-right alert alert-warning' style="margin: 15px; text-align: center;">
  <img src="static/img/programs.png" alt="programs"/>
  <small>Programs &bull; Useful for the theatre.</small>
</div>

In this chapter I've prepared a quick overview of the basic features of C. There are very few *features* in C, and the syntax is relatively simple. But this doesn't mean it is easy. All the depth hides below the surface. Because of this we're going to cover the *features* and *syntax* fairly quickly now, and see them in greater depth as we continue.

The goal of this chapter is get everyone on the same page. People totally new to C should therefore read it in some consideration, and take some time over it, while those with some existing experience may find it easier to skim and return to later as required.


Programs
---------

A program in C consists of only *function definitions* and *structure definitions*.

Therefore a source file is simply a list of *functions* and *types*. These functions can call each other or themselves, and can use any data types that have been declared or are built into the language.

It is possible to call functions in other libraries, or to use their data types. This is how layers of complexity are accumulated in C programming.

As we saw in the previous chapter, the execution of a C program always starts in the function called `main`. From here it calls more and more functions, to perform all the actions it requires.


Variables
---------

Functions in C consists of manipulating *Variables*. These are items of data which we give a name to.

Every variable in C has an explicit *type*. These types are declared by ourselves or built into the language. We can declare a new variable by writing the name of its type, followed by its name, and optionally setting it to some value using `=`. This declaration is called a *statement*, and we terminate all *statements* in C with a semicolon `;`.

To create a new `int` called `count` we could write the following...

```c
int count;
```

Or to declare it and set the value...

```c
int count = 10;
```

Here are some descriptions and examples of some of the built in types.

<table class="table">
  <tr><td>`void`</td>  <td>Empty Type</td>                         <td></td></tr>
  <tr><td>`char`</td>  <td>Single Character/Byte</td>              <td>`char last_initial = 'H';`</td></tr>
  <tr><td>`int`</td>   <td>Integer</td>                            <td>`int age = 23;`</td></tr>
  <tr><td>`long`</td>  <td>Integer that can hold larger values</td><td>`long age_of_universe = 13798000000;`</td></tr>
  <tr><td>`float`</td> <td>Decimal Number</td>                     <td>`float liters_per_pint = 0.568f;`</td></tr>
  <tr><td>`double`</td><td>Decimal Number with more precision</td> <td>`double speed_of_swallow = 0.01072896;`</td></tr>
</table>


Function Declarations
---------------------

A Function is a computation that manipulates variables, and optionally changes the state of the program. It takes as input some variables and returns some single variable as output.

To declare a function we write the type of the variable it returns, the name of the function, and then in parenthesis a list of the variables it takes as input , separated by commas. The contents of the function is put inside curly brackets `{}`, and lists all of the statements the function executes, terminated by semicolons `;`. A `return` statement is used to let the function finish and output a variable.

For example a function that takes two `int` variables called `x` and `y` and adds them together could look like this.

```c
int add_together(int x, int y) {
  int result = x + y;
  return result;
}
```


We call functions by writting their name and putting the arguments to the function in parenthesis, separated by commas. For example to call the above function and store the result in a variable `added` we would write the following.

```
int added = add_together(10, 18);
```


Structure Declarations
----------------------


Structures are used to declare new types*. Structures are several variables bundled together into a single package.

We can use structure to represent more complex data types. For example to represent a point in 2D space we could create a structure called `point` that packs together two `float` (decimal) values called `x` and `y`. To declare structures we can use the `struct` keyword in conjunction with the `typedef` keyword. Our declaration would look like this.

```c
typedef struct {
  float x;
  float y;
} point;
```

We should place this definition above any functions that wish to use it. This type is no different to the built in types, and we can use it in all the same ways. To access an individual field we use a dot `.`, followed by the name of the field, such as `x`.

```c
point p;
p.x = 0.1;
p.y = 10.0;

float length = sqrt(p.x * p.x + p.y * p.y);
```




Pointers
---------

<div class='pull-right alert alert-warning' style="margin: 15px; text-align: center;">
  <img src="static/img/pointer.png" alt="pointer"/>
  <small>Pointer &bull; A short haired one</small>
</div>

A pointer is a variation on a normal type where the type name is suffixed with an asterisk. For example we could declare a *pointer to an integer* by writing `int*`. We already saw a pointer type `char** argv`. This is a *pointer to pointers to characters*, and is used as input to `main` function.

Pointers are used for a whole number of different things such as for strings or lists. These are a difficult part of C and will be explained in much greater detail in later chapters. We won't make use of them for a while, so for now it is good to simply know they exist, and how to spot them. Don't let them scare you off!


Strings
-------

In C strings are represented by the pointer type `char*`. Under the hood they are stored as a list of characters, where the final character is a special character called the *null terminator*. Strings are a complicated, and important part of C, which we'll learn to use effectively in the next few chapters.

Strings can also be declared literally by putting text between quotation marks. We used this in the previous chapter with our string `"Hello, World!"`. For now, remember that if you see `char*`, you can read it as a *string*.


Conditionals
------------

Conditional statements let the program perform some code only if certain conditions are met.

To perform code under some condition we use the `if` statement. This is written as `if` followed by some condition in parenthesis, followed by the code to execute in curly brackets. An `if` statement can be followed by an optional `else` statement, followed by other statements in curly brackets. The code in these brackets will be performed in the case the conditional is false.

We can test for multiple conditions using the logical operators `||` for *or*, and `&&` for *and*.

Inside a conditional statement's parenthesis any value that is not `0` will evaluate to true. This is important to remember as many conditions use this to check things implicitly.

If we wished to check if an `int` called `x` was greater than `10` and less then `100`, we would write the following.

```c
if (x > 10 && x < 100) {
  puts("x is greater than ten and less than one hundred!");
} else {
  puts("x is either less than eleven or greater than ninety-nine!");
}
```





Loops
-----

Loops allow for some code to be repeated until some condition becomes false, or some counter elapses.

There are two main loops in C. The first is a `while` loop. This loop repeatedly executes a block of code until some condition becomes false. It is written as `while` followed by some condition in parenthesis, followed by the code to execute in curly brackets. For example a loop that counts downward from `10` to `1` could be written as follows.

```c
int i = 10;
while (i > 0) {
  puts("Loop Iteration");
  i = i - 1;
}
```

The second kind of loop is a `for` loop. Rather than a condition, this loop requires three expressions separated by semicolons `;`. These are an *initialiser*, a *condition* and an *incrementer*. The *initialiser* is performed before the loop starts, the *condition* is checked at the end of each iteration of the loop, and if false the loop is exited. The *incrementer* is performed before the next iteration of the loop. These loops are often used for counting as they are more compact than the `while` loop.

For example to write a loop that counts up from `0` to `9` we might write the following. In this case the `++` operator increments the variable `i`.

```c
for (int i = 0; i < 10; i++) {
  puts("Loop Iteration");
}
```


Bonus Marks
-----------

<div class="alert alert-warning">
  <ul class="list-group">
    <li class="list-group-item">&rsaquo; Use a `for` loop to print out `Hello World!` five times.</li>
    <li class="list-group-item">&rsaquo; Use a `while` loop to print out `Hello World!` five times.</li>
    <li class="list-group-item">&rsaquo; Declare a function that outputs `Hello World!` `n` number of times. Call this from `main`.</li>
    <li class="list-group-item">&rsaquo; What built in types are there other than the ones listed?</li>
    <li class="list-group-item">&rsaquo; What other conditional operators are there other than *greater than* `&gt;`, and *less than* `&lt;`?</li>
    <li class="list-group-item">&rsaquo; What other mathematical operators are there other than *add* `+`, and *subtract* `-`?</li>
    <li class="list-group-item">&rsaquo; What is the `+=` operator, and how does it work?</li>
    <li class="list-group-item">&rsaquo; What is the `do` loop, and how does it work?</li>
    <li class="list-group-item">&rsaquo; What is the `switch` statement and how does it work?</li>
    <li class="list-group-item">&rsaquo; What is the `break` keyword and what does it do?</li>
    <li class="list-group-item">&rsaquo; What is the `continue` keyword and what does it do?</li>
    <li class="list-group-item">&rsaquo; What does the `typedef` keyword do exactly?</li>
  </ul>
</div>
