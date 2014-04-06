Introduction
============

About
------

In this book you'll learn the C programming language, and at the same time learn how to build *your very own programming language*, a minimal Lisp, in under 1000 lines of code! We'll be using a library to do some of the initial work, so I'm cheating a little bit on the line count, but the rest of the code will be completely original, and you really will create a powerful little Lisp by the end.

This book is somewhat inspired by online tutorials such as [ Write Yourself a Scheme in 48 Hours](http://en.wikibooks.org/wiki/Write_Yourself_a_Scheme_in_48_Hours) which go through the steps of building a programming language from scratch. I wrote this book to show that this kind of fun and creative project is a great way to learn a language, and not limited to abstract high-level languages, or experienced programmers.

Many people are keen to learn C, but have nowhere to start. Well here is your excuse. If you follow this book I can promise that, in the worst case, you'll get a cool new programming language to play with, and hopefully you'll become an experienced C programmer too!


Who this is for
---------------

This book is for anyone wanting to learn C, or who has once wondered how to build their own programming language. This book is not suitable as a first programming language book, but anyone with at least some minimal programming experience, in any language, should find something new and interesting inside.

<div class='pull-right alert alert-warning' style="margin: 15px; text-align: center;">
  <img src="static/img/ada.png" alt="ada"/>
  <small>Ada Lovelace &bull; Your typical <a href="http://areyouabrogrammer.com/">brogrammer</a>.</small>
</div>

I've tried to make this book as friendly as possible to beginners. I welcome beginners the most because they have so much to discover! But beginners will also find this book hard. We will be covering lots of new concepts, and essentially learning two new programming languages at once.

If you look for help you may find people are not patient with you. You may find that, rather than help, they take the time to express how much *they* know about the subject. Esteemed people with lots of experience might tell you that you are wrong. The subtext to their tone might be that you should stop now, rather than inflict your bad code on the world.

After a couple of engagements like this you may decide that you are *not a programmer*, or *don't really like programming*, or that you just *don't get it*. You may have thought that you *once enjoyed* the idea of building your own programming language, but now you have realized that it is too abstract and you *don't care any more*. Now you are concerned with your other passion, and any insight that may have been playful, joyful or interesting will now have become an obstacle.

For this I can only apologize. Programmers can be hostile, macho, arrogant, insecure, and aggressive. There is no excuse for this behaviour. Know that I am on your side. No one *gets it* at first. Everyone struggles and doubts their abilities. Please don't give up or let the joy be sucked out of the creative experience. Be proud of what you create no matter what it is. People like me don't want you to stop programming. We want to hear your voice, and what you have to say, even if you do not shout as loud as others.


Why learn C
-----------

C is one of the most popular and influential programming languages in the world. It is the language of choice for development on Linux, and has been used extensively in the creation of OS X and to some extent Microsoft Windows. It is used on micro-computers too. Your fridge and car probably run on it. In modern software development, the use of C may be escapable, but its legacy is not. Anyone wanting to make a career out of software development would be smart to learn C.

<div class='pull-left alert alert-warning' style="margin: 15px; text-align: center;">
  <img src="static/img/fridge.png" alt="fridge"/>
  <small>A fridge &bull; Your typical C user</small>
</div>

But C is not about software development and careers. C is about **freedom**. It rose to fame on the back of technologies of collaboration and freedom - Unix, Linux, and The Libre Software Movement. It personifies the idea of personal liberty within computing. It wills you to take control of the technology affecting your life.

In this day and age, when technology is more powerful than ever, this could not be more important.

The ideology of freedom is reflected in the nature of C itself. There is little C hides from you, including its warts and flaws. There is little C stops you from doing, including breaking your programs in horrible ways. When programming in C you do not stand on a path, but a plane of decision, and C dares you to decide what to do.

C is also the language of fun and learning. Before the mainstream media got hold of it we had a word for this. *Hacking*. The philosophy that glorifies what is fun and clever. Nothing to do with the illegal unauthorised access of other peoples' computers. Hacking is the philosophy of exploration, personal expression, pushing boundaries, and breaking the rules. It stands against hierarchy and bureaucracy. It celebrates the individual. Hacking baits you with fun, learning, and glory. Hacking is the promise that with a computer and access to the internet, you have the agency to change the world.

To want to master C, is to care about what is powerful, fun, clever, and free. To become a programmer with all the vast powers of technology at his or her fingertips. And the responsibility to say *"I will do good with this"*.

How to learn C
--------------

There is no way around the fact that C is a difficult language. It has many concepts that are unfamiliar, and it makes no attempts to help a new user. In this book I am *not* going to cover in detail things like the syntax of the language, or how to write loops and conditional statements.

I will, on the other hand, show you how to build a *real world* program in C. This approach is always more difficult for the reader, but hopefully will teach you many implicit things a traditional approach cannot. I can't make any promise that this book will make you a confident user of C. What I can promise, is that those 1000 lines of code are going to be packed with content - and you will learn *something* worthwhile.

This book consists of 16 short chapters. How you complete these is up to you. It may well be possible to blast through this book over a weekend, or to take it more slowly, and do a chapter or two each evening over a week. It shouldn't take very long to complete, and will hopefully leave you with a taste for developing your language further.


Why build a Lisp
----------------

The language we are going to be building in this book is a Lisp. This is a family of programming languages characterized by the fact that all their computation is represented by *lists*. This may sound scarier than it is. Lisps are actually very easy, distinctive, and powerful languages.

<div class='pull-right alert alert-warning' style="margin: 15px; text-align: center;">
  <img src="static/img/mike.png" alt="mike"/>
  <small>Mike Tyson &bull; Your typical Lisp user</small>
</div>

Building a Lisp is a great project for so many reasons. It puts you in the shoes of language designers, and gives you an appreciation for the whole process of programming, from language all the way down to machine. It teaches you about functional programming, and different ways to view computation out of the norm. The final product you are rewarded with provides a template for future thoughts and developments, giving you a starting ground for trying new things. It simply isn't possible to comprehend the creativity and cleverness that goes into programming and computer science until you explore languages themselves.

The type of Lisp we'll be building is one I've invented for the purposes of this book. I've designed it for minimalism, simplicity and clarity, and I've become quite fond of it along the way. I hope you come to like it too. Conceptually, syntactically, and in implementation this brand of Lisp has a number of serious differences to other major brands of Lisp. So much so that I'm sure I will be getting e-mails from Lisp programmers telling me it *isn't a Lisp* because it *doesn't do/have/look-like this or that*.

I've not made this Lisp different to confuse beginners or to spread untruths. I've made it different because different is good.

If you are looking to learn about the semantics and behaviours of conventional Lisps, and how to program them, this book may not be for you. What this book offers instead is new and unique concepts, self expression, creativity, and fun. Whatever your motivation, heed this disclaimer now. Not everything I say will be objectively correct or true! You will have to decide that for yourselves.


Your own Lisp
-------------

The best way to follow this book is to, as the title says, write *your own* Lisp. If you are feeling confident enough I want you to add your own features, modifications and changes. Your Lisp should suit you and your own philosophy on what is right or true. Throughout the book I'll be giving description and insight, but with it I'll be providing *a lot* of code. This will make it easy to follow along by copy and pasting each section into your program without really understanding. *Please do not do this!*.

Type out each piece of sample code yourself. This is called *The Hard Way*. Not because it is hard technically, but because it requires discipline. By doing things *The Hard Way* you will come to understand the reasoning behind what you are typing. Ideally things will click as you follow it along character by character. When *reading* you may have an intuition as to why it *looks* right, or what *may* be going on, but this will not always translate to a real understanding unless you do the *writing* yourself!

In a perfect world you would use my code as a reference - an instruction booklet and guide as to building the programming language you always dreamed of. In reality this isn't practical or viable. But the base philosophy remains. If you want to change something, do it.