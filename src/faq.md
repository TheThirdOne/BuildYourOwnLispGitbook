Frequently Asked Questions
==========================

<div class='pull-right alert alert-warning' style="margin: 15px; text-align: center;">
  <img src="img/me.png" alt="me"/>
  <small>Me &bull; Drinking a beer.</small>
</div>


Who are you?
------------

Hello, my name is Daniel Holden. I'm from the UK, and currently studying for a PhD at Edinburgh University. My research is in data driven tools for character animation.

You may know me from one of my other projects such as [Corange](http://libcello.org">libCello</a> or <a href="). As well as hacking on C, I enjoy graphics, game development, and theory of computation.

You can find my personal website [here](http://theorangeduck.com/">here</a>. Or you can follow me on twitter <a href="https://twitter.com/anorangeduck).


Why don't you teach arrays in this book?
----------------------------------------

With an already steep learning curve arrays seemed like a convenient omission to make. Teaching arrays in C is a very easy way to confuse a beginner about pointers, which are a far more important concept to learn. In C, the ways in which arrays and pointers are the same, and yet different, are subtle and numerous. Excluding fixed sized arrays, which have different behaviour anyway, pointers represent a superset of the behaviour of arrays, and so in the context of this book, teaching arrays would have been no more than teaching syntactic sugar.

Those interested in arrays are encouraged to find out more. The book [Learn C the Hard Way](http://c.learncodethehardway.org/) takes the opposite approach to me, and teaches arrays first, with pointers being described as a variation. For those interested in arrays this might prove useful.


Why do you use left-handed pointer syntax?
------------------------------------------

In this book I write the syntax for pointers in a left-handed way `int* x;`, rather than the standard right-handed convention `int *x;`.

Ultimately this distinction is one of personal preference, but the vast majority of C code, as well as the C standards, are written using the right-handed style. This is clearly the default, and most correct way to write pointers, and so my choice might seem odd.

I picked the left-handed version because I believe it is easier to teach to beginners. Having the asterisk on the left hand side emphasises *the type*. It is clearer to read, and makes it obvious that the asterisk is not a weird operator or modification to the variable. With the omission of arrays, and multi-variable declarations, this notation is also almost entirely consistent within this book, and when not, it is noted. K&amp;R themselves have [confusion](http://blog.golang.org/gos-declaration-syntax">admitted</a> the <a href="http://cm.bell-labs.com/cm/cs/who/dmr/chist.html) of the right-handed syntax, made worse by historical baggage and rogue compiler implementations of the early years. For a learning resource I believe picking the left-handed version was the best approach.

Once comfortable with the method behind C's declaration syntax, I encourage programmers to migrate toward the right-handed version.


Why are there no Macros in this Lisp?
-------------------------------------

By far the biggest gripe conventional Lisp programmers have with the Lisp in this book is the lack of Macros. Instead of Macros a new concept of Q-Expressions is used to delay evaluation. To conventional Lisp programmers Q-Expressions are confusing because their semantics differ subtly from the quote Macro.

I use Q-Expressions instead of Macros for a couple of reasons.

First of all I believe them to be easier for beginners than Macros. When evaluation is delayed it is always explicit, shown by the syntax, and not implicit in the function name. It also means that S-Expressions can never be returned by the prompt or seen in the wild. They are always evaluated.

Secondly it is more consistent. It no longer requires the concept of Macros, but instead transforms quoted expressions to become the dominant, more powerful concept that does everything needed by either. With Q-Expressions there are only functions and Expressions, and the language is even more homo-iconic than before.

Finally, Q-Expressions are distinctively more powerful than Macros. Using Q-Expressions it is possible to pass an argument to a function that evaluates to a Q-Expression, making input arguments capable of being dynamic. In conventional Lisps passing an expression to a Macro will always pause the evaluation, and so the arguments cannot be dynamic, only symbolic.