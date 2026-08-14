---
layout: post
title: "Function currying with default params in JavaScript"
description: "Currying and default function parameters don't play well together in JavaScript. Here's why, and two ways to work around it."
date: 2017-02-05 10:00:00 +0200
canonical_url: https://www.linkedin.com/pulse/function-currying-default-params-javascript-vladim%C3%ADr-gorej/
image:
  path: assets/img/blog/function-currying-with-default-params-in-javascript.webp
  width: 800
  height: 533
  caption: Function currying with default params in JavaScript
  object-fit: cover
---

<p class="lead">
  Currying is a very powerful concept in functional programming. Once you understand how it works and what it does, you won't be able to live without it anymore.
  I first read about currying five years ago, while reading a book about JavaScript design patterns. It seemed cool, but I could not imagine a real-world use case for it.
  About seven months ago, when I became "fully functional," I started using it extensively, and nowadays, I use it daily and cannot imagine writing code without it anymore.
</p>

## What is currying?

To borrow a phrase from <a href="http://www.vasinov.com/blog/on-currying-and-partial-function-application/" target="_blank" rel="noopener noreferrer">Vasily Vasinov's article</a>: "Currying is the process of decomposing a function of multiple arguments into a chained sequence of functions of one argument."
There are various implementations of currying, but I prefer the ones from <a href="http://lodash.com/" target="_blank" rel="noopener noreferrer">lodash</a> or <a href="http://ramdajs.com/" target="_blank" rel="noopener noreferrer">ramda</a>. I use them alternately, and they seem to have an identical implementation. Let's see some code now.

{% highlight javascript linenos %}
const { curry } = require('lodash/fp');

const compute = curry((a, b, c) => a + b + c);

compute(1)(2)(3); // 6
{% endhighlight %}

## How do I use it?

Yeah, that seemed cool. But how do I use this in a real-world application? Again, let me show you some code.

{% highlight javascript linenos %}
const { curry } = require('lodash/fp');
const { assoc } = require('ramda');

const updateProductLastView = curry((timestamp, product) => {
  return product.map(assoc('lastViewed', timestamp));
});

server.route({
  method: 'GET',
  path: '/api/products/{id}',
  handler(request, reply) {
    const { productRepository } = request;

    reply(
      productRepository.findById(request.params.id)
        .then(updateProductLastView(Date.now()))
        .then(asyncTap(productRepository.update))
    );
  },
});
{% endhighlight %}

What is happening here? We have a route definition for a <a href="http://hapijs.com/" target="_blank" rel="noopener noreferrer">hapi.js</a> API resource that returns a product by its ID.
We defined a curried function, `updateProductLastView`, and then we use it in the route definition. Notice how we call the function with only
one parameter, and then let the promise apply the second parameter (the product object returned from the database) by calling the function
returned by `updateProductLastView(Date.now())`.

## What are the limits?

About a month ago, I hit a use case involving <a href="http://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Functions/Default_parameters" target="_blank" rel="noopener noreferrer">default function params</a> where applying the currying concept was a no-go. Then I stumbled upon
<a href="http://www.vasinov.com/blog/on-currying-and-partial-function-application/" target="_blank" rel="noopener noreferrer">Vasily Vasinov's article</a> about currying and partial application in Scala. While reading it, I realized that Vasily introduces a Scala concept
called implicits to deal with default params in curried functions. I immediately had a eureka moment and started experimenting in JavaScript.

{% highlight javascript linenos %}
const { curry, curryN } = require('lodash/fp');

const computeV1 = curry((a, b = 2, c) => a + b + c)
const computeV2 = curryN(3, (a, b = 2, c) => a + b + c);

/*
  This doesn't work as expected. The function is executed immediately
  after invoking (1) and you end up with:
  a = 1
  b = 2
  c = undefined
  TypeError: computeV1(...) is not a function
*/
computeV1(1)()(3);

/*
  Let's try to fix this by enforcing the arity to 3. The result is a function.
  It doesn't matter how many additional invocations you do, you always
  end up with a function as a result: computeV2(1)()(3)()()()()()().....
*/
computeV2(1)()(3);
{% endhighlight %}

It doesn't matter whether you use curry from ramda or lodash, currying a function with default parameters will not work. The reason is that a
function with a default param reports its arity based on the position of the first default param.

{% highlight javascript linenos %}
const fn1 = (a, b = 1, c) => a + b + c;
const fn2 = (a, b, c = 2) => a + b + c;
const fn3 = (a = 1, b, c) => a + b + c;

fn1.length; //=> 1
fn2.length; //=> 2
fn3.length; //=> 0
{% endhighlight %}

## The solution #1

OK, so currying and default params just don't play along. We don't have implicits in JavaScript like Scala does. So what do we do?
Let's get back to basics and write this function without default params.

{% highlight javascript linenos %}
const { curry, defaultTo } = require('lodash/fp');

const computeV3 = curry((a, b, c) => {
  const defaultB = defaultTo(2, b);
  return a + defaultB + c;
})

computeV3(1)(null)(3);
computeV3(1)(undefined)(3);
computeV3(1)()(3); // This doesn't work
{% endhighlight %}

Handle your default value in the body of the function itself and call the function with an empty value: in this case, `null` or `undefined`.
Calling the function without arguments is like not calling it at all (you end up in the `computeV2` scenario).

## The solution #2

While the above solution works, it forces you to forget the notion of default params, which is unfortunate. A more elegant solution to this
problem is to use `curryN`, either from ramda or lodash, and provide the arity of the curried function explicitly. If you want to apply the
default value of the param, you must explicitly pass the `undefined` value at the right position (note that `null` doesn't work in this use case).

{% highlight javascript linenos %}
const { curryN } = require('lodash/fp');
/* const { curryN } = require('ramda'); */

const foo = curryN(3, (a, b = 2, c) => a + b + c);

foo(1)(undefined)(3); //=> 6
{% endhighlight %}

In my opinion, this solution is cleaner and more elegant, and you can make a convention in your codebase to use `curryN` instead of `curry`.
Then, you don't need to pay attention to how you define your functions or whether they have default params.

## Update (18.03.2017)

It seems there is a more elegant solution to this problem, conceived by Kyle Simpson. Using the <a href="http://github.com/getify/fpo" target="_blank" rel="noopener noreferrer">FPO</a> library, your code can look like this:

{% highlight javascript linenos %}
function foo(x, y = 2, z) {
    console.log( x, y, z );
}

var g = FPO.curry( {
    fn: FPO.apply( {fn: foo} ),
    n: 2    // use `2` here for currying-count to allow skipping
} );

g( {z: 3} )( {x: 1} );
// 1 2 3
{% endhighlight %}

---

<div class="alert alert-info" role="alert">
  All code snippets in this article are licensed under <a href="https://www.apache.org/licenses/LICENSE-2.0" target="_blank" rel="noopener noreferrer">Apache 2.0 license</a> using the following copyright notice: <strong>Copyright 2017 Vladimír Gorej</strong>.
</div>
