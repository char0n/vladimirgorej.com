---
layout: post
title: "Lifting functions into monadic context in JavaScript"
description: "A practical pattern for taming imperative edge cases: lifting plain functions into a monadic context (Maybe) using monet.js and ramda-adjunct's liftFN, and why fantasy-land compatibility matters."
date: 2017-04-23 10:00:00 +0200
canonical_url: https://www.linkedin.com/pulse/lifting-functions-monadic-context-javascript-vladim%C3%ADr-gorej/
image:
  path: assets/img/blog/lifting-functions-into-monadic-context-in-javascript.webp
  width: 1280
  height: 720
  caption: Lifting functions into monadic context in JavaScript
  object-fit: cover
---

<p class="lead">
  Lifting functions into the monadic context of algebraic structures is a quite practical pattern. This article will prove its practicality,
  and when you finish reading it, you will have another FP tool to conquer the complexities of imperative code and create yet more elegant
  functional code. I will assume the reader's basic understanding of the concepts of functional programming and <a href="https://github.com/fantasyland/fantasy-land" target="_blank" rel="noopener noreferrer">algebraic structures</a>.
  I will use <a href="http://cwmyers.github.io/monet.js/" target="_blank" rel="noopener noreferrer">monet.js</a> as a JavaScript implementation of a <a href="https://github.com/fantasyland/fantasy-land#monad" target="_blank" rel="noopener noreferrer">Monad</a> algebraic structure
  and <a href="http://ramdajs.com/" target="_blank" rel="noopener noreferrer">ramda</a> as a functional library containing some nifty utils to manipulate Monads.
</p>

Let's say we have a programming challenge. We want to add two numbers together to create another number. Well, it's not really a challenge for most of you; it's quite a primitive problem to solve.

{% highlight javascript linenos %}
// add :: (Number, Number) -> Number
const add = (a, b) => a + b;
{% endhighlight %}

Here, there it is. Quite simple. Same type of input and output. It is very easy to understand and reason with. In category theory we call this endomorphism. Only numbers should go into this function, and only numbers should come out.

Now we need some function to parse user input and run it through our `add` function. As it turns out, there is already such a function in JavaScript. It's called `parseInt`.

{% highlight javascript linenos %}
const userInputA = '1';
const userInputB = '2';

const a = parseInt(userInputA, 10); //=> Number(1)
const b = parseInt(userInputB, 10); //=> Number(2)
{% endhighlight %}

But what if our user input contains some junk instead of numbers? We may end up with the following.

{% highlight javascript linenos %}
const userInputA = 'junk';
const userInputB = '2';

const a = parseInt(userInputA, 10); //=> NaN
const b = parseInt(userInputB, 10); //=> Number(2)
{% endhighlight %}

Houston, we have a problem! Constant `a` now contains `NaN`. In category theory, our function `add` can process input from the category Number. But `NaN` is not part of that category, so the behavior of `add` will be undefined.

## Maybe monad to the rescue

`Maybe` is a monad that contains some value or nothing. Basically, it is a type-safe container for our parsed value. We can lift JavaScript's `parseInt` function to return a `Maybe` monad.

{% highlight javascript linenos %}
const { Maybe } = require('monet');

const userInputA = 'junk';
const userInputB = '2';

// safeParseInt :: (Any, Number) -> Maybe Number
const safeParseInt = (value, radix = 10) => {
  const parsed = parseInt(value, radix);
  return isNaN(parsed) ? Maybe.Nothing() : Maybe.Some(parsed);
};

const a = safeParseInt(userInputA); //=> Maybe.Nothing()
const b = safeParseInt(userInputB); //=> Maybe.Some(2)
{% endhighlight %}

Now that we have our parsed values locked in type-safe containers, we need to add them together. But how do we do that? Actually, `Maybe` implements the <a href="https://github.com/fantasyland/fantasy-land#apply" target="_blank" rel="noopener noreferrer">Apply spec</a> and therefore contains an `ap` method that allows us to combine multiple monads together and map them over a function.

{% highlight javascript linenos %}
const { curry } = require('ramda');
const { Maybe } = require('monet');

const userInputA = '1';
const userInputB = '2';

// safeParseInt :: (Any, Number) -> Maybe Number
const safeParseInt = (value, radix = 10) => {
  const parsed = parseInt(value, radix);
  return isNaN(parsed) ? Maybe.Nothing() : Maybe.Some(parsed);
};

const a = safeParseInt(userInputA); //=> Maybe.Some(1)
const b = safeParseInt(userInputB); //=> Maybe.Some(2)

// add :: (Number, Number) -> Number
const add = (a, b) => a + b;
// curried version of add
// addC :: Number -> Number -> Number
const addC = curry(add);

const added = a.ap(b.map(add)); //=> Maybe.Some(3)
{% endhighlight %}

This seems quite ugly and impractical. Let us create a better version of this code.

{% highlight javascript linenos %}
const { liftFN } = require('ramda-adjunct');
const { Maybe } = require('monet');

const userInputA = '1';
const userInputB = '2';

// safeParseInt :: (Any, Number) -> Maybe Number
const safeParseInt = (value, radix = 10) => {
  const parsed = parseInt(value, radix);
  return isNaN(parsed) ? Maybe.Nothing() : Maybe.Some(parsed);
};

const a = safeParseInt(userInputA); //=> Maybe.Some(1)
const b = safeParseInt(userInputB); //=> Maybe.Some(2)

// addM :: Apply Number => Number -> Number -> Number
const addM = liftFN(2, add);

const added = addM(a, b); //=> Maybe.Some(3)
{% endhighlight %}

What have we just done? Well, we used <a href="http://char0n.github.io/ramda-adjunct/1.3.2/RA.html#.liftFN" target="_blank" rel="noopener noreferrer">liftFN</a> from <a href="https://github.com/char0n/ramda-adjunct" target="_blank" rel="noopener noreferrer">ramda-adjunct</a>. It takes an arity and a function and returns a homomorphic, autocurried function lifted into monadic context with the specified arity. Or in layman's terms: it takes the inputs as monads, unwraps the values from the monads, applies them to the original function, wraps the result into a compatible monad, and returns it. This is what I call <strong>heavy lifting</strong>.

## Insight into heavy lifting

Ramda also contains a <a href="http://ramdajs.com/docs/#liftN" target="_blank" rel="noopener noreferrer">liftN</a> util. Why didn't we use it? Well, there's a thing called the <a href="https://github.com/fantasyland/fantasy-land#apply" target="_blank" rel="noopener noreferrer">fantasy-land specification</a>. This specification tells us how to implement algebraic structures so that they can be interoperable. Ramda <= 0.23.0 implements an obsolete version of this spec, and that is why it is not compatible with monet.js. Today I have released `liftFN`, which acts as an interop for Ramda <= 0.23.0 and monet.js <= 0.8.1. The next version of Ramda will once again be compatible with the fantasy-land specification, and the next version of monet.js will also be compatible with the fantasy-land spec thanks to PR #112, making it compatible with Ramda. For the time being, use <a href="http://char0n.github.io/ramda-adjunct/1.3.2/RA.html#.liftFN" target="_blank" rel="noopener noreferrer">liftFN</a> from <a href="https://github.com/char0n/ramda-adjunct" target="_blank" rel="noopener noreferrer">ramda-adjunct</a> to circumvent this inconvenience.

This is the topmost level the elevator goes today ;] Remember to write pure, testable functions that work with JavaScript's native types, and lift them later into monadic context (if you need to). And compose, compose, compose...!

---

<div class="alert alert-info" role="alert">
  All code snippets in this article are licensed under <a href="https://www.apache.org/licenses/LICENSE-2.0" target="_blank" rel="noopener noreferrer">Apache 2.0 license</a> using the following copyright notice: <strong>Copyright 2017 Vladimír Gorej</strong>.
</div>
