---
layout: post
title:  "The Unnecessary Specialization Antipattern"
date:   2021-01-28
---

The Unnecessary Specialization Antipattern

When a coder sits down to write, there’s some particular problem they’re trying to solve. They’ll bang out some code that clears that specific roadblock, and then move on.

Solving that specific problem is fine, but sometimes it’s just as easy (or even easier) to solve a more general problem, of which their original problem was a special case. When code gets checked in that passes this opportunity for generalization by, I call it the *Unnecessary Specialization Antipattern*.

Here’s a simple example:

```python
def timestamped_print(clock, *args):
  time = clock.now()
  print(‘[%s]: ‘, time, end=’’)
  print(*args)
```

This function prints messages to the console along with the current time, using the passed-in `Clock`. Compare that with the following:

```python
def generalized_timestamped_print(time, *args):
  print(‘[%s]: ‘, time, end=’’)
  print(*args)
```

This version solves a more general problem: it can print times that aren’t “right now”, and it doesn’t require the caller to have a `Clock`, just a `Time`. On top of that, the implementation is actually simpler!

Obviously this is a toy example, but out in the wild there are plenty of functions that take file paths when they could take `File`s, or `File`s when they could take `Read`s, etc. 

Not every bit of code that _could_ be generalized is an example of Unnecessary Specialization. If you keep making a function more and more general, eventually you’ll have a Universal Turing Machine. Pinpointing the _right_ degree of generality is an art, but as a rough guideline: a) if the code for the general solution is at least as readable as the code for the special case, and b) if your special case can be clearly and trivially solved using the general solution, generalize it.

Some hints that you might consider generalizing your code:

* One of your parameters gets used exactly once, right at the beginning of a function. Let the caller do that for you.
* Your parameters have lots of methods you’re not using. See if a smaller interface would work just as well.
* Your tests need to do gymnastics in order to set up the input to your code. Perhaps a more general version of your code will be easier to test.

Some dangers associated with overgeneralization:

* Using more general tools can obscure _why_ you’re using them. For instance, in a linear algebra scenario, left-multiplying a column vector by a row vector full of `1`s does the same thing as `sum()`, but the latter conveys intent much better.
* Even if the _current_ version of the code can be generalized, it’s possible you’ll eventually want to make changes that only work in the special case. For instance, your function only calls `read()` on one of its parameters, but you’re planning an optimization that will use `seek()` as well. In that case, leave it specialized; it’s easier to generalize code after the fact than it is to specialize it.
* With complex projects, supporting the more general case may make the code _locally_ simpler, but increase the complexity of the system overall. For instance, on a social networking site, allowing a user to upload a profile picture is a special case of allowing them to upload an HTML snippet to inject into the page, but implementing the latter would be disastrous for the security and UX teams.
