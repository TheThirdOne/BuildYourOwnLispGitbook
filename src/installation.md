Installation
============


Setup
-----

![capttop](img/cattop.png "Cat &bull; Install at own risk")

Before we can start programming in C we'll need to install a couple of things, and set up our environment so that we have everything we need. Because C is such a universal language this should hopefully be fairly simple. Essentially we need to install two main things. A *text editor* and a *compiler*.


Text Editor
-----------

A text editor is a program that allows you to edit text files in a way suitable for programming.

On **Linux** the text editor I recommend is [gedit](http://projects.gnome.org/gedit/). Whatever other basic text editor comes installed with your distribution will also work well. If you are a Vim or Emacs user these are fine to use. Please don't use an IDE. It isn't required for such a small project and won't help in understanding what is going on.

On **Mac** A simple text editor that can be used is [TextWrangler](http://www.barebones.com/products/textwrangler/). If you have a different preference this is fine, but please don't use XCode for text editing. This is a small project and using an IDE won't help you understand what is going on.

On **Windows** my text editor of choice is [Notepad++](http://notepad-plus-plus.org/). If you have another preference this is fine. Please *don't* use *Visual Studio* as it does not have proper support for C programming. It you attempt to use it you will run into many problems.


Compiler
--------

The compiler is a program that transforms the C source code into a program your computer can run. The installation process for these is different depending on what operating system you are running.

Compiling and running C programs is also going to require really basic usage of the command line. This I will not cover, so I am going to assume you have at least some familiarity with using the command line. If you are are worried about this then search online for information on using it, relevant to your operating system.

On **Linux** you can install a compiler by downloading some packages. If you are running Ubuntu or Debian you can install everything you need with the following command `sudo apt-get install build-essential`. If you are running Fedora or a similar Linux variant you can use this command `su -c "yum groupinstall development-tools"`.

On **Mac** you can install a compiler by downloading and installing the latest version of XCode from Apple. If you are unsure of how to do this you can search online for "installing xcode" and follow any advice shown. You will then need to install the *Command Line Tools*. On Mac OS X 10.9 this can be done by running the command `xcode-select --install` from the command line. On versions of Mac OS X prior to 10.9 this can be done by going to XCode Preferences, Downloads, and selecting *Command Line Tools* for Installation.

On **Windows** you can install a compiler by downloading and installing [MinGW.](http://www.mingw.org/") If you use the installer at some point it may present you with a list of possible packages. Make sure you pick at least `mingw32-base` and `msys-base`. Once installed you need to add the compiler and other programs to your system `PATH` variable. To do this follow [these instructions](href="http://www.computerhope.com/issues/ch000549.htm) appending the directory `;C:\MinGW\bin` to the variable called `PATH`. You can create this variable if it doesn't exist. You may need to restart `cmd.exe` for the changes to take effect. This will allow you to run a compiler from the command line `cmd.exe`. It will also install other programs which make `cmd.exe` act like a Unix command line.


Testing the Compiler
--------------------

To test if your C compiler is installed correctly type the following into the command line.

`cc --version`

If you get some information about the compiler version echoed back then it should be installed correctly. You are ready to go! If you get any sort of error about an unrecognised or not found command, then it is not ready. You may need to restart the command line or your computer for changes to take effect.


Hello World
-----------

Now that your environment is set up, start by opening your text editor and inputting the following program. Create a directory where you are going to put your work for this book, and save this file as `hello_world.c`. This is your first C program!

```c
#include <stdio.h>

int main(int argc, char** argv) {
  puts("Hello, world!");
  return 0;
}
```


This may look like a lot of crazy symbols that make very little sense. I'll try to explain it step by step.

In the first line we *include* what is called a *header*. This statement allows us to use the functions from `stdio.h`, the standard input and output library which comes included with C. One of the functions from this library is the `puts` function you see later on in the program.

Next we *declare* a function called `main`. This function is declared to output an `int`, and take as input an `int` called `argc` and a `char**` called `argv`. All C programs must contain this function. All programs start running from this function.

Inside `main` the `puts` function is *called* with the argument `"Hello, world!"`. This outputs the message `Hello, world!` to the command line. The function `puts` is short for *put string*. The second statement inside the function is `return 0;`. This tells the `main` function to finish and return `0`. When a C program returns `0` this indicates there have been no errors running the program.


Compilation
-----------

Before we can run this program we need to compile it. This will produce the actual *executable* we can run on our computer. Open up the command line and browse to the directory that `hello_world.c` is saved in. You can then compile your program using the following command.

`cc -std=c99 -Wall hello_world.c -o hello_world`

This compiles the code in `hello_world.c`, reporting any warnings, and outputs the program to a new file called `hello_world`. We use the `-std=c99` flag to tell the compiler which *version* or *standard* of C we are programming with. This lets the compiler ensure our code is standardized, so that people with different operating systems or compilers will be able to use our code.

If successful you should see the output file in the current directory. This can be run by typing `hello_world` (or just `hello_world` on Windows). If everything is correct you should see a friendly `Hello, world!` message appear.

**Congratulations!** You've just compiled and run your first C program.


Errors
------

If there are some problems with your C program the compilation process may fail. These issues can range from simple syntax errors, to other complicated problems that are hard to understand.

Sometimes the error message from the compiler will make sense, but if you are having trouble understanding it try searching online for it. You should see if you can find a concise explanation of what it means, and work out how to correct it. Remember this: there are many people before you who have struggled with exactly the same problems.

![smash](img/smash.png "Rage &bull; A poor debugging technique")

Sometimes there will be many compiler errors stemming from one source. Always work through compiler errors from first to last.

Sometimes the compiler will compile a program, but when you run it it will crash. Debugging C programs in this situation is hard. It can be an art far beyond the scope of this book.

My first port of call for debugging a crashing C program is to print out lots of information as the program is running. Using this method I can try to isolate exactly what part of the code is incorrect and what, if anything, is going wrong up until the crash. For beginners I would recommend this technique. It is a debugging technique which is *active*. This is the important thing. As long as you are doing *something*, and not just staring at the code, the process is less painful and the temptation to give up is lessened.

For people feeling more confident a program called `gdb` can be used to debug your C programs. This can be difficult and complicated to use, but it is also very powerful and can give you extremely valuable information and what went wrong and where. Information on how to use `gdb` can be found online.

On **Mac** the most recent versions of OS X don't come with `gdb`. Instead you can use `lldb` which does largely the same job.

On **Linux** or **Mac** `valgrind` can be used to aid the debugging of memory leaks and other more nasty errors. Valgrind is a tool that can save you hours, or even days, of debugging. It does not take much to get proficient at it, so investigating it is highly recommended. Information on how to use it can be found online.


Documentation
-------------

Through this book you may come across functions in some example code that you don't recognize. You might wonder what it does. In this case you will want to look toward the [online documentation](http://en.cppreference.com/w/c) of the standard library. This will explain all the functions included in the standard library, what they do, and how to use them.


Reference
---------

<div class="alert alert-warning">
  **What is this section for?**

  In this section I&#39;ll link to the code I've written for this particular chapter of the book. When finishing with a chapter your code should probably look similar to mine. This code can be used for reference if the explanation has been unclear.

  If you encounter a bug please do not copy and paste my code into your project. Try to track down the bug yourself and use my code as a reference to highlight what may be wrong, or where the error may lie.
</div>

<div class="panel-group alert alert-warning" id="accordion">
  <div class="panel panel-default">
    <div class="panel-heading">
      <h4 class="panel-title">
        <a data-toggle="collapse" data-parent="#accordion" href="#collapseOne">
          hello_world.c
        </a>
      </h4>
    </div>
    <div id="collapseOne" class="panel-collapse collapse">
      <div class="panel-body">
```c
#include <stdio.h>

int main(int argc, char** argv) {
  puts("Hello, world!");
  return 0;
}
```
      </div>
    </div>
  </div>
</div>


Bonus Marks
-----------

<div class="alert alert-warning">
  **What is this section for?**

  In this section I'll list some things to try for fun, and learning.

  It is good if you can attempt to do some of these challenges. Some will be easy, while some will be very difficult. For this reason don&#39;t worry if you can&#39;t figure them all out. Some might not even be possible!

  Many will require some research on the internet. This is an integral part of learning a new language so should not be avoided. The ability to teach yourself things is one of the most valuable skills in programming. See how many you can complete for each chapter. Move on when you get bored.
</div>

<div class="alert alert-warning">
  <ul class="list-group">
    <li class="list-group-item">&rsaquo; Change the `Hello World!` greeting given by your program to something different.</li>
    <li class="list-group-item">&rsaquo; What happens when no `main` function is given?</li>
    <li class="list-group-item">&rsaquo; Use the online documentation to lookup the `puts` function.</li>
    <li class="list-group-item">&rsaquo; Look up how to use `gdb` and run it with your program.</li>
  </ul>
</div>
