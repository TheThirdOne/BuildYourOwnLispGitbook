Standard Library
================

Minimalism
----------

<div class='pull-right alert alert-warning' style="margin: 15px; text-align: center;">
  <img src="static/img/library.png" alt="library"/>
  <small>Library &bull; Built with just leather, paper, wood, and ink.</small>
</div>

The Lisp we've built has been purposefully minimal. We've only added the fewest number of core structures and builtins. If we chose these carefully, as we did, then it should allow us to add in everything else required to the language.

The motivation behind minimalism is two-fold. The first advantage is that it makes the core language simple to debug and easy to learn. This is a great benefit to developers and users. Like Occam's Razor it is almost always better to trim away any waste if it results in a equally expressive language. The second reason is that having a small language is also aesthetically nicer. It is clever, interesting, and fun to see how small we can make the core of a language, and still get something useful out of the other side. As hackers, which we should be by now, this is something we enjoy.


Atoms
-----

When dealing with conditionals we added no new boolean type to our language. Because of this we didn't add `true` or `false` either. Instead we just used numbers. Readability is still important though, so we can define some constants to represent these values.

On a similar note, many lisps use the word `nil` to represent the empty list `{}`. We can add this in too. These constants are sometimes called *atoms* because of their fundamental and constant behaviour.

The user is not forced to use these named constants, and can use numbers and empty lists instead as they like. This choice empowers users, something we believe in.


```lispy
; Atoms
(def {nil} {})
(def {true} 1)
(def {false} 0)
```


Building Blocks
---------------

We've already come up with a number of cool functions I've been using in the examples. One of these is our `fun` function that allows us to declare functions in a neater way. We should definitely include this in our standard library.

```lispy
; Function Definitions
(def {fun} (\ {f b} {
  def (head f) (\ (tail f) b)
}))
```

We also had our `unpack` and `pack` functions. These too are going to be essential for users. We should include these along with their `curry` and `uncurry` aliases.


```lispy
; Unpack List for Function
(fun {unpack f l} {
  eval (join (list f) l)
})

; Pack List for Function
(fun {pack f & xs} {f xs})

; Curried and Uncurried calling
(def {curry} {unpack})
(def {uncurry} {pack})
```

Say we want to do several things in order. One way we can do this is to put each thing to do as an argument to some function. We known that arguments are evaluated in order from left to right, which is essentially sequencing events. For functions such as `print` and `load` we don't care much about what it evaluates to, but do care about the order in which it happens.

Therefore we can create a `do` function which evaluates a number of expressions in order and returns the last one. This relies on the `last` function, which returns the final element of a list. We'll define this later.

```lispy
; Perform Several things in Sequence
(fun {do & l} {
  if (== l {})
    {{}}
    {last l}
})
```

Sometimes we want to save results to local variables using the `=` operator. When we're inside a function this will implicitly only save results locally, but sometimes we want to open up an even more local scope. For this we can create a function `let` which creates an empty function for code to take place in, and evaluates it.

```lispy
; Open new scope
(fun {let b} {
  ((\ {_} b) ())
})
```

We can use this in conjunction with `do` to ensure that variables do not leak out of their scope.

```lispy
lispy> let {do (= {x} 100) (x)}
100
lispy> x
Error: Unbound Symbol &#39;x&#39;
lispy>
```


Logical Operators
-----------------

We didn't define any local operators such as `and` and `or` in our language. This might be a good thing to add in later. For now we can use arithmetic operators to emulate them. Think for a second how these functions work when encountering `0` or `1` for their various inputs.

```lispy
; Logical Functions
(fun {not x}   {- 1 x})
(fun {or x y}  {+ x y})
(fun {and x y} {* x y})
```


Miscellaneous Functions
-----------------------

Here are a couple of miscellaneous functions that don&#39;t really fit in anywhere. See if you can guess their intended functionality.

```lispy
(fun {flip f a b} {f b a})
(fun {ghost & xs} {eval xs})
(fun {comp f g x} {f (g x)})
```

The `flip` function takes a function `f` and two arguments `a` and `b`. It then applies `f` to `a` and `b` in the reversed order. This might be useful when we want a function to be *partially evaluated*. If we want to partially evaluate a function by only passing it in it&#39;s second argument we can use `flip` to give us a new function that takes the first two arguments in reversed order.

This means if you want to apply the second argument of a function you can just apply the first argument to the `flip` of this function.

```lispy
lispy> (flip def) 1 {x}
()
lispy> x
1
lispy> def {define-one} ((flip def) 1)
()
lispy> define-one {y}
()
lispy> y
1
lispy>

```

I couldn't think of a use for the `ghost` function, but it seemed interesting. It simply takes in any number of arguments and evaluates them as if they were the expression itself. So it just sits at the front of an expression like a ghost, not interacting of changing the behaviour of the program at all. If you can think of a use for it I'd love to hear.

```lispy
lispy> ghost + 2 2
4

```

The `comp` function is used to compose two functions. It takes as input `f`, `g`, and an argument to `g`. It then applies this argument to `g` and applies the result again to `f`. This can be used to compose two function together into a new function that applies both of them in series. Like before we can use this in combination with partial evaluation to build up complex functions from simpler ones.

For example we can compose two functions. One that negates a number and another that unpacks a list of numbers for multiplying together using `*`.

```lispy
lispy> (unpack *) {2 2}
4
lispy> - ((unpack *) {2 2})
-4
lispy> comp - (unpack *)
(\ {x} {f (g x)})
lispy> def {mul-neg} (comp - (unpack *))
()
lispy> mul-neg {2 8}
-16
lispy>

```


List Functions
---------------

The `head` function is used to get the first element of a list, but what it returns is still wrapped in the list. If we want to actually get the element out of this list we need to extract it somehow.

Single element lists evaluate to just that element, so we can use the `eval` function to do this extraction. We can also define a couple of helper functions for aid extracting the first, second and third elements of a list. We'll use these function a lot later.

```lispy
; First, Second, or Third Item in List
(fun {fst l} { eval (head l) })
(fun {snd l} { eval (head (tail l)) })
(fun {trd l} { eval (head (tail (tail l))) })
```

We looked briefly at some recursive list functions a few chapters ago. Naturally there are many more we can define using this technique.

To find the length of a list we can recursive over it adding `1` to the length of the tail. To find the `nth` element of a list we can perform the `tail` operation and count down until we reach `0`. To get the last element of a list we can just access the element at the length minus one.

```lispy
; List Length
(fun {len l} {
  if (== l {})
    {0}
    {+ 1 (len (tail l))}
})

; Nth item in List
(fun {nth n l} {
  if (== n 0)
    {fst l}
    {nth (- n 1) (tail l)}
})

; Last item in List
(fun {last l} {nth (- (len l) 1) l})
```

There are lots of other useful functions that follow this same pattern. We can define functions for taking and dropping the first so many elements of a list, or functions for checking if a value is an element of a list.

```lispy
; Take N items
(fun {take n l} {
  if (== n 0)
    {{}}
    {join (head l) (take (- n 1) (tail l))}
})

; Drop N items
(fun {drop n l} {
  if (== n 0)
    {l}
    {drop (- n 1) (tail l)}
})

; Split at N
(fun {split n l} {list (take n l) (drop n l)})

; Element of List
(fun {elem x l} {
  if (== l {})
    {false}
    {if (== x (fst l)) {true} {elem x (tail l)}}
})

```

These functions all follow similar patterns. It would be great if there was some way to extract this pattern so we don't have to type it out every time. For example we may want a way we can perform some function on every element of a list. This is a function we can define called  `map`. It takes as input some function, and some list. For each item in the list it applies `f` to that item and appends it back onto the front of the list. It then applies `map` to the tail of the list.

```lispy
; Apply Function to List
(fun {map f l} {
  if (== l {})
    {{}}
    {join (list (f (fst l))) (map f (tail l))}
})
```

With this we can do some neat things that look a bit like looping. In some ways this concept is more powerful than looping. Instead of thinking about performing some function to each element of the list in turn, we can think about acting on all the elements at once. We *map the list* rather than *changing each element*.

```lispy
lispy> map - {5 6 7 8 2 22 44}
{-5 -6 -7 -8 -2 -22 -44}
lispy> map (\ {x} {+ x 10}) {5 2 11}
{15 12 21}
lispy> print {"hello" "world"}
{"hello" "world"}
()
lispy> map print {"hello" "world"}
"hello"
"world"
{() ()}
lispy>
```

An adaptation of this idea is a `filter` function which, takes in some functional condition, and only includes items of a list which match that condition.

```lispy
; Apply Filter to List
(fun {filter f l} {
  if (== l {})
    {{}}
    {join (if (f (fst l)) {head l} {{}}) (filter f (tail l))}
})
```

This is what it looks like in practice.

```lispy
lispy> filter (\ {x} {> x 2}) {5 2 11 -7 8 1}
{5 11 8}
```

Some loops don't exactly act on a list, but accumulate some total or condense the list into a single value. These are loops such as sums and products. These can be expressed quite similarly to the `len` function we've already defined.

These are called *folds* and they work like this. Supplied with a function `f`, a *base value* `z` and a list `l` they merge each element in the list with the total, starting with the base value.

```lispy
; Fold Left
(fun {foldl f z l} {
  if (== l {})
    {z}
    {foldl f (f z (fst l)) (tail l)}
})
```

Using folds we can define the `sum` and `product` functions in a very elegant way.

```lispy
(fun {sum l} {foldl + 0 l})
(fun {product l} {foldl * 1 l})
```


Conditional Functions
---------------------

By defining our `fun` function we've already shown how powerful our language is in its ability to define functions that look like new syntax. Another example of this is found in emulating the C `switch` and `case` statements. In C these are built into the language, but for our language we can define them as part of a library.

We can define a function `select` that takes in zero or more two-element lists as input. For each two element list in the arguments it first evaluates the first element of the pair. If this is true then it evaluates and returns the second item, otherwise it performs the same thing again on the rest of the list.

```lispy
(fun {select & cs} {
  if (== cs {})
    {error "No Selection Found"}
    {if (fst (fst cs)) {snd (fst cs)} {unpack select (tail cs)}}
})
```

We can also define a function `otherwise` to always evaluate to `true`. This works a little bit like the `default` keyword in C.

```lispy
; Default Case
(def {otherwise} true)

; Print Day of Month suffix
(fun {month-day-suffix i} {
  select
    {(== i 0)  "st"}
    {(== i 1)  "nd"}
    {(== i 3)  "rd"}
    {otherwise "th"}
})
```

This is actually somewhat more powerful than the C `switch` statement. In C rather than passing in conditions the input value is compared only for equality with a number of constant candidates. We can also define this function in our Lisp, where we compare a value to a number of candidates. In this function we take some value `x` followed by zero or more two-element lists again. If the first element in the two-element list is equal to `x`, the second element is evaluated, otherwise the process continues down the list.

```lispy
(fun {case x & cs} {
  if (== cs {})
    {error "No Case Found"}
    {if (== x (fst (fst cs))) {snd (fst cs)} {unpack case (join (list x) (tail cs))}}
})
```

The syntax for this function becomes really nice and simple. See if you can think up any other control structures or useful functions that you'd like to implement using these sorts of methods.

```lispy
(fun {day-name x} {
  case x
    {0 "Monday"}
    {1 "Tuesday"}
    {2 "Wednesday"}
    {3 "Thursday"}
    {4 "Friday"}
    {5 "Saturday"}
    {6 "Sunday"}
})
```


Fibonacci
---------

No standard library would be complete without an obligatory definition of the Fibonacci function. Using all of the above things we've defined  we can write a cute little `fib` function that is really quite readable, and clear semantically.

```lispy
; Fibonacci
(fun {fib n} {
  select
    { (== n 0) {0} }
    { (== n 1) {1} }
    { otherwise {+ (fib (- n 1)) (fib (- n 2))} }
})
```

This is the end of the standard library I've written. Building up a standard library is a fun part of language design, because you get to be creative and opinionated on what goes in and stays out. Try to come up with something you are happy with. Exploring what is possible to define and do can be very interesting.


Reference
---------

<div class="panel-group alert alert-warning" id="accordion">
  <div class="panel panel-default">
    <div class="panel-heading">
      <h4 class="panel-title">
        <a data-toggle="collapse" data-parent="#accordion" href="#collapseOne">
          prelude.lspy
        </a>
      </h4>
    </div>
    <div id="collapseOne" class="panel-collapse collapse">
      <div class="panel-body">
```lispy
;;;
;;;   Lispy Standard Prelude
;;;

;;; Atoms
(def {nil} {})
(def {true} 1)
(def {false} 0)

;;; Functional Functions

; Function Definitions
(def {fun} (\ {f b} {
  def (head f) (\ (tail f) b)
}))

; Open new scope
(fun {let b} {
  ((\ {_} b) ())
})

; Unpack List to Function
(fun {unpack f l} {
  eval (join (list f) l)
})

; Unapply List to Function
(fun {pack f & xs} {f xs})

; Curried and Uncurried calling
(def {curry} {unpack})
(def {uncurry} {pack})

; Perform Several things in Sequence
(fun {do & l} {
  if (== l {})
    {{}}
    {last l}
})

;;; Logical Functions

; Logical Functions
(fun {not x}   {- 1 x})
(fun {or x y}  {+ x y})
(fun {and x y} {* x y})


;;; Numeric Functions

; Minimum of Arguments
(fun {min & xs} {
  if (== (tail xs) {}) {fst xs}
    {do
      (= {rest} (unpack min (tail xs)))
      (= {item} (fst xs))
      (if (< item rest) {item} {rest})
    }
})

; Minimum of Arguments
(fun {max & xs} {
  if (== (tail xs) {}) {fst xs}
    {do
      (= {rest} (unpack max (tail xs)))
      (= {item} (fst xs))
      (if (> item rest) {item} {rest})
    }
})

;;; Conditional Functions

(fun {select & cs} {
  if (== cs {})
    {error "No Selection Found"}
    {if (fst (fst cs)) {snd (fst cs)} {unpack select (tail cs)}}
})

(fun {case x & cs} {
  if (== cs {})
    {error "No Case Found"}
    {if (== x (fst (fst cs))) {snd (fst cs)} {unpack case (join (list x) (tail cs))}}
})

(def {otherwise} true)


;;; Misc Functions

(fun {flip f a b} {f b a})
(fun {ghost & xs} {eval xs})
(fun {comp f g x} {f (g x)})

;;; List Functions

; First, Second, or Third Item in List
(fun {fst l} { eval (head l) })
(fun {snd l} { eval (head (tail l)) })
(fun {trd l} { eval (head (tail (tail l))) })

; List Length
(fun {len l} {
  if (== l {})
    {0}
    {+ 1 (len (tail l))}
})

; Nth item in List
(fun {nth n l} {
  if (== n 0)
    {fst l}
    {nth (- n 1) (tail l)}
})

; Last item in List
(fun {last l} {nth (- (len l) 1) l})

; Apply Function to List
(fun {map f l} {
  if (== l {})
    {{}}
    {join (list (f (fst l))) (map f (tail l))}
})

; Apply Filter to List
(fun {filter f l} {
  if (== l {})
    {{}}
    {join (if (f (fst l)) {head l} {{}}) (filter f (tail l))}
})

; Return all of list but last element
(fun {init l} {
  if (== (tail l) {})
    {{}}
    {join (head l) (init (tail l))}
})

; Reverse List
(fun {reverse l} {
  if (== l {})
    {{}}
    {join (reverse (tail l)) (head l)}
})

; Fold Left
(fun {foldl f z l} {
  if (== l {})
    {z}
    {foldl f (f z (fst l)) (tail l)}
})

; Fold Right
(fun {foldr f z l} {
  if (== l {})
    {z}
    {f (fst l) (foldr f z (tail l))}
})

(fun {sum l} {foldl + 0 l})
(fun {product l} {foldl * 1 l})

; Take N items
(fun {take n l} {
  if (== n 0)
    {{}}
    {join (head l) (take (- n 1) (tail l))}
})

; Drop N items
(fun {drop n l} {
  if (== n 0)
    {l}
    {drop (- n 1) (tail l)}
})

; Split at N
(fun {split n l} {list (take n l) (drop n l)})

; Take While
(fun {take-while f l} {
  if (not (unpack f (head l)))
    {{}}
    {join (head l) (take-while f (tail l))}
})

; Drop While
(fun {drop-while f l} {
  if (not (unpack f (head l)))
    {l}
    {drop-while f (tail l)}
})

; Element of List
(fun {elem x l} {
  if (== l {})
    {false}
    {if (== x (fst l)) {true} {elem x (tail l)}}
})

; Find element in list of pairs
(fun {lookup x l} {
  if (== l {})
    {error "No Element Found"}
    {do
      (= {key} (fst (fst l)))
      (= {val} (snd (fst l)))
      (if (== key x) {val} {lookup x (tail l)})
    }
})

; Zip two lists together into a list of pairs
(fun {zip x y} {
  if (or (== x {}) (== y {}))
    {{}}
    {join (list (join (head x) (head y))) (zip (tail x) (tail y))}
})

; Unzip a list of pairs into two lists
(fun {unzip l} {
  if (== l {})
    {{{} {}}}
    {do
      (= {x} (fst l))
      (= {xs} (unzip (tail l)))
      (list (join (head x) (fst xs)) (join (tail x) (snd xs)))
    }
})

;;; Other Fun

; Fibonacci
(fun {fib n} {
  select
    { (== n 0) 0 }
    { (== n 1) 1 }
    { otherwise (+ (fib (- n 1)) (fib (- n 2))) }
})

```
      </div>
    </div>
  </div>
</div>


Bonus Marks
-----------

<div class="alert alert-warning">
  <ul class="list-group">
    <li class="list-group-item">&rsaquo; Rewrite the `len` function using `foldl`.</li>
    <li class="list-group-item">&rsaquo; Rewrite the `elem` function using `foldl`.</li>
    <li class="list-group-item">&rsaquo; Incorporate your standard library directly into the language. Make it load at start-up.</li>
    <li class="list-group-item">&rsaquo; Write some documentation for your standard library, explaining what each of the functions do.</li>
    <li class="list-group-item">&rsaquo; Write some example programs using your standard library, for users to learn from.</li>
    <li class="list-group-item">&rsaquo; Write some test cases for each of the functions in your standard library.</li>
  </ul>
</div>