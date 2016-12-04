---
layout: post
title: "Passing Parameters in Java Explained [INCOMPLETE]"
date: 2016-12-03
detail: "A brief introduction to reference and value types and common pitfalls you may run into."
---
## Primer
I'm going to spoil the ending. Java passes parameters by value. _Exclusively_. That's the long and short of it. There's no "depends" clause here, and (un)fortunately for you, you don't get to choose how you want to pass parameters to a function.

In many of the explanations I've seen, various analogies are employed and the wording avoids the technical concept underlying a "reference" or "value" type. In my opinion, this attempt to skirt around the core idea is a disservice that needlessly complicates the explanation.

This concept I'm referring to is memory. If you already have a decent grasp of how virtual addresses work, then feel free to skip this section.

Let's take a look at the following line. What does it do?

{% highlight java linenos %}
int x = 7;
{% endhighlight %}

This an assignment statement. It tells the machine to store the value 7 at a storage location that we name `x`. The storage location in question is actually a chunk of memory. To be technical, it's really just a small chunk of your machine's [ram](/blog/2016/12/03/ram). This is convenient for us (humans) because later when we want to use our value 7, we don't need to have the variable's address on hand. Rather, we can simply use `x`, which is much easier to remember.