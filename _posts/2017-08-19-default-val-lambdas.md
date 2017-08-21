---
title: De-obfuscating Python one-liners
updated: 2017-08-19 19:26
title_stub: lambdas
---

Python, which you'll come to find out is my favorite language to play with, features a very handy [Programming FAQ][progfaq] that I recommend you read over, regardless of your level of experience. One of the questions reads,

> Is it possible to write obfuscated one-liners in Python?


As those who frequent the [puzzles and code golf][pcg] community on Stack Overflow will readily attest to, Python holds its own weight among the ad-hoc esoteric languages that are usually used in code-golfing challenges. In these challenges, obfuscation is a necessary by-product of cramming the most functionality into the fewest bytes of code. So, the answer to the above question is a resounding **yes**.

To a newcomer, however, code-golfed programs--a term synonymous with obfuscated code--may seem confusing and abstruse. Today I'll try to shine a light on this field by going through one of the examples listed in the FAQ and accompany my walkthrough with generalized tips for demystifying such code [^1].

## The Puzzle

The following outputs the first 10 [Fibonacci numbers][fibonacci] in a pithy 91 bytes:

```python
print(list(map(lambda x,f=lambda x,f:(f(x-1,f)+f(x-2,f)) if x>1 else 1:f(x,f), range(10))))
```

Let's outline our plan of attack:
1. **Simplify**
2. **Rewrite**
3. **Generalize**

Although I will go into detail later, here is an overview of each step. This plan is general and can be applied to other obfuscated code you may encounter... both in the appropriate forums (a la code golf) and in others decidedly [less so][enterprise] 😳.

For **Simplify**, we should test our understanding of the code by writing something that follows the same idea as the obfuscated code, but returns a type vastly simpler, or executes fewer lines. Simplify should precede Rewrite because we may hit a realization that reduces our workload.

During **Rewrite**, we want to break apart the code into as many variable assignments as we can. Lambdas, or anonymous functions, should be converted to normal, non-anonymous, functions. Those who are accustomed to functional programming may debate the last point, but I would argue that a majority are more familiar with the imperative approach.

Finally, on **Generalize** we typically want to extend the knowledge we gained from the exercise and see how we can apply it to other, related problems. With the overview out of the way, let's begin.

## Simplify

Assume our input `nums` is already initialized in all subsequent code snippets:

```python
nums = range(10)
```

We begin with the simplest lambda one-liner I can think of that is appropriate to the problem. We want a lambda that will accept two parameters. One of the params will be populated by the nums iterable, and the other will default to one, since it is not populated.

```python
simple_lamb = lambda x, f = 1: x + f
result = map(simple_lamb, nums)
print(result) # [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
```

For each element, the lambda adds one to it. But why didn't we write it like `lambda x: x + 1`?

While it may feel like we're a long way off, pattern-matching the above example to our problem reveals a nice similarity:

```python
simple_lamb = lambda x, f = 1: x + f
mean_lamb = lambda x, f = lambda x, f: (some stuff): f(x,f)
```

Focusing on the ends of the mean lambda (we'll get to the meaty inner part shortly), you'll notice the lambdas aren't all that different: both accept one argument as an element from nums. The difference comes from the default parameter. The int object 1 (remember, _[everything is an object in Python][everything]_) is the default value the simple lambda assigns to the parameter f. In the mean lambda, a child lambda object (meanie jr?) is the default value[^2] assigned to the f parameter. 

Instead of using an int object as the default value, we can also introduce our own child lambda object as the default value to bring our simple lambda more in sync with the fibonacci lambda.

```python
simple_lamb = lambda x, f = lambda x: 2: f(x)
map(simple_lamb, nums) # [0, 2, 4, 6, 8, 10, 12, 14, 16, 18]
```

Not terribly exciting, I know. But we're getting closer in terms of functionality. Our child lambda accepts one parameter, x, and doubles it. When the parent lambda calls `f(x)` the output is just that argument doubled, which we see in the commented result.

## Rewrite

First, we start by de-lambdafying the code. Upon first inspection, you may be tempted to say that we are dealing with nested lambdas. We can analyze that claim by checking this and converting the inner lambda into a non-anonymous function I will christen `fib_weird`.

```python
def fib_weird(val, func):
	return fib_weird(x - 1, fib_weird) + fib_weird(x - 2, fib_weird) \
		if x > 1 \
		else 1

def main():
	mean_lamb = lambda x, f = fib_weird: f(x, f)
	map(mean_lamb, nums) # [1, 1, 2, 3, 5, 8, 13, 21, 34, 55]
```

I call it `fib_weird` because it's pretty similar to our typical recursive implementation of the Fibonacci sequence, 

```python
def fib_norm(val):
	return fib_norm(x - 1) + fib_norm(x - 2) if x > 1 else 1
```

except that `fib_weird` throws in a reference to itself. Both functions are correct since they satisfy the [recurrence relation][fibrecur], but why does the weird version need that additional function reference? We'll get to that in a moment, after we convert the parent lambda to a normal function.

```python
def fib_weird(x, f):
    return fib_weird(x - 1, fib_weird) + fib_weird(x - 2, fib_weird) \
        if x > 1 \
        else 1

def fib_parent(x, f = fib_weird):
	return f(x, f)

def main():
	map(fib_parent, nums) # [1, 1, 2, 3, 5, 8, 13, 21, 34, 55]
```

At this point, you might be wondering why we can't just use `fib_norm` as the default value for `f` in `fib_parent`. And you'd be totally correct, since this works fine:

```python
def fib_norm(x):
    return fib_norm(x - 1) + fib_norm(x - 2) \
        if x > 1 \
        else 1

def fib_parent(x, f = fib_norm):
	return f(x)

def main():
	map(fib_parent, nums) # [1, 1, 2, 3, 5, 8, 13, 21, 34, 55]
```

What's the deal? It's actually in the name: since lambdas are _[anonymous functions][anonymous]_ they, by definition, cannot refer to themselves since they don't have an identifier, unlike the normal function which is named `fib_norm` and can thus call itself later. That's why this throws a `NameError`, unsurprisingly:

```python
bad_lamb = lambda x, f = lambda x: func(x-1) + func(x-2) if x > 1 else 1: f(x)
map(bad_lamb, nums) # NameError: global name 'func' is not defined
```

## Generalize

Therefore, to implement recusion with lambda functions we must pass a reference to the function itself. That additional reference we see in `fib_weird` is necessitated by the function. It's anonymous, so if it wants to invoke itself, it needs a reference to itself since it has no other means of self-identification. Perhaps this anthropomorphization will help:

```text
me: (to function Frank) Who are you?
Frank: I'm Frank!
me: (to anonymous function Alan) Who are you?
Alan: I don't know.
(Alan checks ID inside his wallet)
Alan: Oh... I'm Alan!
```

Silly conversations aside, I also want to cover a question I raised during the **Rewrite** phase, about nested lambdas. As you might have guessed, the Fibonacci generator is not an example of that. Rather, we have one lambda that has a default value of another lambda, which I chose to differentiate by labeling them parent and child, respectively. That naming decision was arbitrary. As for an example of nested lambdas, you can check this out[^3]:

```python
(lambda f,g:lambda h,i:lambda x,y: f(h(x))*g(i(y)))\
(lambda k:k+1,lambda j:j-1)(lambda k:k+1, lambda j:j-1)(5,5) # 21
```

Now while your first reaction may be 😱 I think you'll be able to sort this one out. Since this is the **Generalize** phase, I will not be explaining this one. But, you can try applying the workflow you learned while deciphering code obfuscated by _default-valued lambdas_ to deciphering this case, where we have code obfuscated by _nested lambdas_. Since I'm feeling generous, however, I've included a slightly watered-down example you can take a look at as well. If you want the full experience, however, I recommend only looking at the code snippet above.

By the way, the inner, nested lambda satisfies the definition of a [closure][closure] because we have an nested function that accesses values from the local variables of the higher-order function. Perhaps I'll have a post about that...

```python
product = lambda f,g:lambda x,y:f(x)*g(y)
product(lambda k:k+1,lambda j:j-1)(5, 5) # 24
```

Anyway, I hoped you enjoyed reading this post. Python really is a powerful language that is great to use in both the puzzling and non-puzzling worlds.

[^1]: While obfuscated code is widely considered a form of antipattern, I think there are some benefits in understanding and writing such code, in that it improves logical thinking. Sometimes, regular code isn't much clearer either!
[^2]: This construct rests on the requirement that `nums` is a list of single integers. If instead it were a list of tuples, the default value would not be used.
[^3]: My example is _loosely_ based on this [SO Post][nested]

[progfaq]: https://docs.python.org/3/faq/programming.html
[pcg]: https://codegolf.stackexchange.com/
[fibonacci]: https://en.wikipedia.org/wiki/Fibonacci_number
[enterprise]: https://en.wikipedia.org/wiki/Enterprise_software
[everything]: https://pythoninternal.wordpress.com/2014/08/11/everythings-an-object/
[fibrecur]: https://en.wikipedia.org/wiki/Recurrence_relation#Fibonacci_numbers
[anonymous]: https://en.wikipedia.org/wiki/Anonymous_function
[nested]: https://stackoverflow.com/questions/36391807/understanding-nested-lambda-function-behaviour-in-python
[closure]: https://en.wikipedia.org/wiki/Closure_(computer_programming)
