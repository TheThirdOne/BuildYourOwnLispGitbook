Languages
=========


What is a Programming Language?
-------------------------------

A programming language is very similar to a real language. There is a structure behind it, and some rules which dictate what is, and isn't, a valid thing to say. When we read and write natural language, we are unconsciously learning these rules, and the same is true for programming languages. We can utilise these rules to understand others, and generate our own speech, or code.

In the 1950s the linguist *Noam Chomsky* formalised a number of [important observations](http://en.wikipedia.org/wiki/Chomsky_hierarchy) about languages. Many of these form the basis of our understanding of language today. One of these was the observation that natural languages are built up of recursive and repeated substructures.

![cat](img/cat.png "Cat &bull; cannot speak or program")

As an example of this, we can examine the phrase.

&rsaquo; `The cat walked on the carpet.`

Using the rules of English, the noun `cat` can be replaced by two nouns separated by `and`.

&rsaquo; <code>The <strong>cat and dog</strong> walked on the carpet.</code>

Each of these new nouns could in turn be replaced again. We could use the same rule as before, and replace `cat` with two new nouns joined with `and`. Or we could use a different rule and replace each of the nouns with an adjective and a noun, to add description to them.

&rsaquo; <code>The <strong>cat and mouse</strong> and dog walked on the carpet.</code>

&rsaquo; <code>The <strong>white cat</strong> and <strong>black dog</strong> walked on the carpet.</code>

These are just two examples, but English has many different rules for how types of words can be changed, manipulated and replaced.

We notice this exact behaviour in programming languages too. In C, the body of an `if` statement contains a list of new statements. Each of these new statements, could themselves be another `if` statement. These repeated structures and replacements are reflected in all parts of the language. These are sometimes called *re-write rules* because they tell you how one thing can be *re-written* as something else.

&rsaquo; `if (x > 5) { return x; }`

&rsaquo; <code>if (x > 5) { <strong>if (x > 10) { return x; }</strong> }</code>

The consequence of this observation by *Noam Chomsky* was important. It meant that although there is an infinite number of different things that can be said, or written down, in a particular language; it is still possible to process and understand all of them, with a finite number of these re-write rules. The name given to a set of these re-write rules is a *grammar*.

We can describe re-write rules in a number of ways. One way is textual. We could say something like, "a *sentence* must contain a *verb phrase*", or "a *verb phrase* can be either a *verb* or, an *adverb* and a *verb*". This method is good for humans but it is too vague for computers to understand. When programming we need to write down a more formal description of a grammar.

To write a programming language such as our Lisp we are going to need to understand grammars. For reading in the user input we need to write a *grammar* which describes it. Then we can use it along with our user input, to decide if the input is valid. We can also use it to build a structured internal representation, which will make the job of *understanding* it, and then *evaluating* it, performing the computations encoded within, much easier.

This is where a library called `mpc` comes in.


Parser Combinators
------------------

`mpc` is a *Parser Combinator* library written by yours truly. This means it is a library that allows you to build programs that understand and process particular languages. These are known as *parsers*. There are many different ways of building parsers, but the nice thing about using a *Parser Combinator* library is that it lets you build *parsers* easily, just by specifying the *grammar* ... sort of.

Many *Parser Combinator* libraries actually work by letting you write normal code that *looks a bit like* a grammar, not by actually specifying a grammar directly. In many situations this is fine, but sometimes it can get clunky and complicated. Luckily for us `mpc` allows for us to write normal code that just *looks like* a grammar, *or* we can use special notation to write a grammar directly!


Coding Grammars
---------------

So what does code that *looks like* a grammar..*look like*? Let us take a look at `mpc` by trying to write code for a grammar that recognizes [the language of Shiba Inu](http://knowyourmeme.com/memes/doge). More colloquially know as *Doge*. This language we are going to define as follows.

&rsaquo; An *Adjective* is either *"wow"*, *"many"*, *"so"* or *"such"*.
&rsaquo; A *Noun* is either *"lisp"*, *"language"*, *"c"*, *"book"* or *"build"*.
&rsaquo; A *Phrase* is an *Adjective* followed by a *Noun*.
&rsaquo; A *Doge* is zero or more *Phrases*.

We can start by trying to define *Adjective* and *Noun*. To do this we create two new parsers, represented by the type `mpc_parser_t*`, and we store them in the variables `Adjective` and `Noun`. We use the function `mpc_or` to create a parser where one of several options should be used, and the function `mpc_sym` to wrap our initial strings.

If you squint your eyes you could attempt to read the code as if it were the rules we specified above.

```c
/* Build a new parser 'Adjective' to recognize descriptions */
mpc_parser_t* Adjective = mpc_or(4,
  mpc_sym("wow"), mpc_sym("many"),
  mpc_sym("so"),  mpc_sym("such")
);

/* Build a new parser 'Noun' to recognize things */
mpc_parser_t* Noun = mpc_or(5,
  mpc_sym("lisp"), mpc_sym("language"),
  mpc_sym("c"),    mpc_sym("book"),
  mpc_sym("build")
);
```

<div class="alert alert-warning">
  **How can I access these `mpc` functions?**

  For now don't worry about compiling or running any of the sample code in this chapter. Just read it and see if you can understand the theory behind grammars. In the next chapter we'll get setup with `mpc` and use it for a language closer to our Lisp.
</div>

To define `Phrase` we can reference our existing parsers. We need to use the function `mpc_and`, that specifies one thing is required then another. As input we pass it `Adjective` and `Noun`, our previously defined parsers. This function also takes the arguments `mpcf_strfold` and `free`, which say how to join or delete the results of these parsers. Ignore these arguments for now.

```c
mpc_parser_t* Phrase = mpc_and(2, mpcf_strfold, Adjective, Noun, free);
```

To define *Doge* we must specify that *zero or more* of some parser is required. For this we need to use the function `mpc_many`. As before, this function requires the special variable `mpcf_strfold` to say how the results are joined together, which we can ignore.

```c
mpc_parser_t* Doge = mpc_many(mpcf_strfold, Phrase);
```

By creating a parser that looks for *zero or more* occurrences of another parser a cool thing has happened. Our `Doge` parser accepts inputs of any length. This means its language is *infinite*. Here are just some examples of possible strings `Doge` could accept. Just as we discovered in the first section of this chapter we have used a finite number of re-write rules to create an infinite language.

```c
"wow book such language many lisp"
"so c such build such language"
"many build wow c"
""
"wow lisp wow c many language"
"so c"
```

If we use more `mpc` functions, we can slowly build up parsers that parse more and more complicated languages. The code we use *sort of* reads like a grammar, but becomes much more messy with added complexity. Due to this, taking this approach isn't always an easy task. A whole set of helper functions that build on simple constructs to make frequent tasks easy are all documented on the [mpc repository](http://github.com/orangeduck/mpc). This is a good approach for complicated languages, as it allows for fine-grained control, but won't be required for our needs.


Natural Grammars
----------------

`mpc` lets us write grammars in a more natural form too. Rather than using C functions that look less like a grammar, we can specify the whole thing in one long string. When using this method we don&#39;t have to worry about how to join or discard inputs, with functions such as `mpcf_strfold`, or `free`. All of that is done automatically for us!

Here is how we would recreate the previous examples using this method.

```c
mpc_parser_t* Adjective = mpc_new("adjective");
mpc_parser_t* Noun      = mpc_new("noun");
mpc_parser_t* Phrase    = mpc_new("phrase");
mpc_parser_t* Doge      = mpc_new("doge");

mpca_lang(MPC_LANG_DEFAULT,
  "                                                                     \
    adjective : \"wow\" | \"many\" | \"so\" | \"such\";                 \
    noun      : \"lisp\" | \"language\" | \"c\" | \"book\" | \"build\"; \
    phrase    : <adjective> <noun>;                                     \
    doge      : <phrase>*;                                              \
  ",
  Adjective, Noun, Phrase, Doge);

/* Do some parsing here... */m

mpc_cleanup(4, Adjective, Noun, Phrase, Doge);
```

Without having an exactly understanding of the syntax for that long string, it should be obvious how much *clearer* the grammar is in this format. If we learn what all the special symbols mean we barely have to squint our eyes to read it as one.

Another thing to notice is that the process is now in two steps. First we create and name several rules using `mpc_new` and then we define them using `mpca_lang`.

The first argument to `mpca_lang` are the options flags. For this we just use the defaults. The second is a long multi-line string in C. This is the *grammar* specification. It consists of a number of *re-write rules*. Each rule has the name of the rule on the left, a colon `:`, and on the right it&#39;s definition terminated with a semicolon `;`.

The special symbols used to define the rules on the right hand side work as follows.

<table class='table'>
  <tr><td>`"ab"`</td><td>The string `ab` is required.</td></tr>
  <tr><td>`'a'`</td><td>The character `a` is required.</td></tr>
  <tr><td>`'a' 'b'`</td><td>First `'a'` is required, then `'b'` is required..</td></tr>
  <tr><td>`'a' | 'b'`</td><td>Either `'a'` is required, or `'b'` is required.</td></tr>
  <tr><td>`'a'*`</td><td>Zero or more `'a'` are required.</td></tr>
  <tr><td>`'a'+`</td><td>One or more `'a'` are required.</td></tr>
  <tr><td>`<abba>`</td><td>The rule called `abba` is required.</td></tr>
</table>

<div class="alert alert-warning">
  **Sounds familiar...**

  Did you notice that the description of what the input string to `mpca_lang` should look like sort of sounded like I was specifying a grammar? That&#39;s because it was! `mpc` uses itself internally to parse the input you give it to `mpca_lang`. It does it by specifying a *grammar* in code using the previous method. How neat is that..
</div>

Using what is described above verify that what I've written above is equal to what we specified in code.

This method of specifying a grammar is what we are going to use in the following chapters. It might seem overwhelming at first. Grammars can be difficult to understand right away. But as we continue you will become much more familiar with how to create and edit them.

This chapter is about theory, so if you are going to try some of the bonus marks, don't worry too much about correctness. Thinking in the right mindset is more important. Feel free to invent symbols and notation for certain concepts to make them simpler to write down. Some of the bonus marks also might require cyclic or recursive grammar stuctures, so don't worry if you have to use these!



Reference
---------

<div class="panel-group alert alert-warning" id="accordion">=
  <a href="src/doge_code.c">doge_code.c</a>
  <a href="src/doge_grammar.c">doge_grammar.c</a>
</div>


Bonus Marks
-----------

<div class="alert alert-warning">
  <ul class="list-group">
    <li class="list-group-item">&rsaquo; Write down some more examples of strings the `Doge` language contains.</li>
    <li class="list-group-item">&rsaquo; Why are there back slashes `\` in front of the quote marks `"` in the grammar?</li>
    <li class="list-group-item">&rsaquo; Why are there back slashes `\` at the end of the line in the grammar?</li>
    <li class="list-group-item">&rsaquo; Describe textually a grammar for decimal numbers such as `0.01` or `52.221`.</li>
    <li class="list-group-item">&rsaquo; Describe textually a grammar for web URLs such as `http://www.buildyourownlisp.com`.</li>
    <li class="list-group-item">&rsaquo; Describe textually a grammar for simple English sentences such as `the cat sat on the mat`.</li>
    <li class="list-group-item">&rsaquo; Describe more formally the above grammars. Use `|`, `*`, or any symbols of your own invention.</li>
    <li class="list-group-item">&rsaquo; If you are familiar with JSON, textually describe a grammar for it.</li>
  </ul>
</div>