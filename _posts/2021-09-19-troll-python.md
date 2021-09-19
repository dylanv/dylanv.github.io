---
layout: post
title: A look at some interesting python features with the greatest script of all time
---

Recently I've been seeing some references to a truly spectacular troll post on reddit's r/learnlearnprogramming.
The [post](https://www.reddit.com/r/learnprogramming/comments/kegalv/my_first_python_script_coming_from_c_feedback/) is ostensibly a beginner c programmer
trying to learn python and asking for some feedback on his script.

Here it is in all it's glory:
``` python
#include <stdint.h>
#include <stdio.h>

#/* pre-declare types and functions */
void: (type) = (type) ("(void *)0"),
int16_t: (type) = (type) (int(16))(),
int32_t: (type) = (type) (int(32))(),
puts: ((str) := void) = (print),
printf: ((str) := void) = (puts),

#/* confirm function and variable declarations */
[[void]] = (void),
[[total]] = (int32_t),
[[i]] = (int32_t),
[[puts]] = (puts),
[[printf]] = (printf),

#/* ensure forwards and backwards compatibility */
import __future__ as __past__
#define FUTURE PAST
#define PAST FUTURE

#/* output a formatted string to stdout */
def printf(s, __VA_ARGS__) -> (void) :{
    (void) (puts((void).__mod__(s, __VA_ARGS__), end=((void).__new__)(void))),
}

#/*******************************************************
# * SUMMATION SCRIPT                                    *
# *******************************************************
# * Sum up all the natural numbers up to 6. The result  *
# * should be 0 + 1 + 2 + 3 + 4 + 5 + 6 = 21.           *
# *******************************************************/

#/* print numbers from 0 to 6 */
int32_t: i = 0,
while (i <= 6) :{
    (void) (printf(" %d\n", i)),
    (i := i + 1),
}

(void) (puts("__")),

#/* sum up numbers from 0 to 6 */
int32_t: total = 0,
int32_t: i = 0,
for (int32_t) in (i <- 0, i <= 6, ++i) :{
    (total := total + i),
}

(void) (printf("%d\n", total)),
```
I love this so much. Maybe my favorite part is our brave coder asking if everyone is mad because he replaced the semi-colons with commas. Gold.

Joking aside; this script uses a bunch of newer python features like [walrus operators](https://docs.python.org/3/whatsnew/3.8.html)(new in 3.8) and liberal abuse of [type hints](https://docs.python.org/3/library/typing.html).
The seems like a fun way to look at some less well known features and quirks of Python so let's break down this bad boy.

## Breaking it down

Starting off we have some comments to set the C-like ambiance.
``` python
#include <stdint.h>
#include <stdio.h>
```
These are simply comments and don't do anything.

Next up we is where things start to get fun. 
``` python
#/* pre-declare types and functions */
void: (type) = (type) ("(void *)0"),
int16_t: (type) = (type) (int(16))(),
int32_t: (type) = (type) (int(32))(),
```
Things really aren't as complicated as they look. Most of the complexity comes from enough extra parenthesis to make a lisp programmer blush.
If we strip away so of the unnecessary parenthesis and confusing bits we get
``` python
void = (str,)
```
They've created a variable `void` with a type hint saying our variable has the type `type` which is the function for getting the type of an object (it doesn't obviously but it looks fun).
We can see what `type` does when looking at the value `(type) ("(void *)0")` is just `type("(void *)0")` which is `str` the python built-in string type. The final comma means that the value is actually a tuple.

The next two lines are very similar but get actual values instead of a tuple containing a class.
``` python
int16_t: (type) = (type) (int(16))(),
int32_t: (type) = (type) (int(32))(),
```
`int(16)` just evaluates to `16`. Taking `type(16)` will give the builtin `int` type. The final brackets call this with no arguments: `int()` simply returning `0`. 
So these to lines both evaluate to a tuple with one item: `(0, )`. With out all the confusing bits we get:
``` python
int16_t = (0, )
int32_t = (0 ,)
```

Our next lines abuse the fact that as a full blooded object orientated language everything in Python is an object, including functions.
``` python
puts: ((str) := void) = (print),
printf: ((str) := void) = (puts),
```
If we ignore the type hints for now; we just create a variable that points to a tuple containing the built-in print function. We can ignore `printf` because it gets redefined later anyway
``` python
puts = (print,)
```
The type hint uses the walrus operator to introduce some nasty side effects. Our type hints are normal python code and are evaluated at runtime along with everything else, the walrus operator is used to do
an inline assignment to the built-in `str` type. This overloads it with what we had in `void` which was a tuple: `(str, )`. 

The docs do say "Try to limit use of the walrus operator to clean cases that reduce complexity and improve readability." and I'm not sure this qualifies. It does achieve the goal of making everything look very confusing but
I don't really like that it ends up breaking the `str` type. They could've grabbed `str` from the tuple `void` to avoid breaking it:
``` python
((str) := void[0])
```

So far every line has ended in a comma this has meant that everything has been a tuple with one item. This is not very convenient so next up everything gets unpacked:
``` python
#/* confirm function and variable declarations */
[[void]] = (void),
[[total]] = (int32_t),
[[i]] = (int32_t),
[[puts]] = (puts),
[[printf]] = (printf),
```
On the right hand side we have remarkable commitment to the _ending every line with a comma_ joke. But essentially, we have each line saying: a list of a list containing our variable is equal to a tuple of tuple containing our 
variable. Because Python is smart and cool it unpacks both sides and sees that `void = void` and we get rid of all the tuples.

Presented without comment:
``` python
import __future__ as __past__
#define FUTURE PAST
#define PAST FUTUR
```
Just _amazing_.

Now's where things get really fun. We've got a not quite perfect recreation of the printf function. ``` python
def printf(s, __VA_ARGS__) -> (void) :{
    (void) (puts((void).__mod__(s, __VA_ARGS__), end=((void).__new__)(void))),
}
```
Because we know that `void = str` and `puts = print` we can clean it up a bit:
``` python
def printf(s, __VA_ARGS__):
    {str(print(str.__mod__(s, __VA_ARGS__), end=str.__new__(str))),}
```
Python functions without a `return` statement return what ever the last line of the function evaluates. The curly braces: `{}` are used to create a set.
So our function returns a set with a single item, which is whatever the str representation of our print function is (it's `None`). That's not really important though
because what we really care about here is that `print` gets evaluated so the side-effect of this function is running print.

This is where we get to dig into the guts of python, everything in Python is an object and they have some [special attributes](https://docs.python.org/3/reference/datamodel.html), 
these hidden attributes are often called `dunders` because the are wrapped in double underscores. We have two here `__mod__` and `__new__`.

`__mod__` is the modulo `%` operator, famous from everyone's favorite interview question: fizz-buzz. In Python the modulo operator is also used for string formatting:
```
print("integer: %d" % 0)
>>> integer: 0
```
So `str.__mod__(s, __VA_ARGS__)` is just `s % _VA_ARGS__` (`__VA_ARGS__` is not a dunder it just a very badly named function argument.).

Lastly we have the `end` keyword argument to `print`. This is what gets appended to the end of the printed string, by default it is a newline. In this case it's essentially `None` so our print 
side-effect will not print on a newline each time (You can look up what `__new__` does if you're curious but here it isn't doing anything fun).

Now we use our mutant printf function to print out 0 to 6
``` python
#/* print numbers from 0 to 6 */
int32_t: i = 0,
while (i <= 6):
    {(void) (printf(" %d\n", i)), (i := i + 1),}
```
This is another nonsense line, earlier we assigned `i` to the value of `int32_t` (it was 0). So we already have `i = 0`. This just makes `int32_t` a tuple containing a single 0 item.
``` python
int32_t: i = 0,
```
In the while loop we once again use the trick of creating a set and getting side-effects from the creating the items.
The first item calls our mutant `printf` function essentially doing (note the newline character because we broke that by passing in `None` to `end` in `print` earlier.)
``` python
print(" %d\n" % i)
```
The second item uses another walrus operator to inplace increment `i`.

Finally we get a total:
``` python
for (int32_t) in (i <- 0, i <= 6, ++i) :{
    (total := total + i),
}
```
There is some more fun stuff here. First we notice that `int32_t` isn't used in the loop so let's get rid of it and replace the walrus shenanigans with a simple increment.
``` python
for _ in (i <- 0, i <= 6, ++i) :{
   total += i 
}
```
We're essentially saying add `i` to `total` 3 times. `i=7` after the printing loop and `total` was initialized to 0 at the beginning. So this only works because 3x7 happens to equal the sum from 1 to 6, it wouldn't work for any
other numbers.

Nevertheless the tuple `(i <- 0, i <= 6, ++i)` has some interesting Python quirks. 

The first item `i <- 0` looks like an assignment (as it would be in R for example), but it's actually just poorly formatted. More clearly it should be `i < -0`, of course `-0` is just `0`. 

The last one `++i` looks like an increment in many other languages, Python famously doesn't let you increment like this, so what's going on?
The `+` operator in the case is a [Unary operation](https://en.wikipedia.org/wiki/Unary_operation) (it has one argument, the normal case `a + b` would be the binary operation). The unary `+` is an identity operation (it just returns the input) and is included because it is the complement to the far more common unary `-` operation.
``` python
x = 5
x = -x  # The unary - operation
```
So `++i` is really `+(+i)` which is just `i`. (You would think `+(-4) = 4` but it doesn't because maths. (It actually makes more sense if you think of it as a contraction of `0 + x` so `+x = 0 + x`, likewise for `-`.))

## What we've learned
Python is a remarkably flexible language that'll let you do pretty much anything you want. Most of what we've looked at here isn't too useful day to day (except operators, the walrus operator and function objects I guess). 
You don't often need to dig down to the level of dunders but it's good to know they're there (I'm particulars fond of `__name__`). 

This script is an obvious joke but it does end up being a fun exercise for beginner programmers to unravel all its nonsense and hopefully you learn something along the way.