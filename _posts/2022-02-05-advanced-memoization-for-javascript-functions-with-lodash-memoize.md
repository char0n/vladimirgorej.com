---
layout: post
title: "Advanced memoization for JavaScript functions with lodash.memoize"
description: "In computing, memoization or memoisation is an optimization technique used primarily to speed up computer programs by storing the results of expensive function calls and returning the cached result when the same inputs occur again."
date: 2022-02-05 20:00:00 +0200
image:
  path: assets/img/blog/lodash.webp
  width: 400
  height: 400
  caption: Copyright JS Foundation and other contributors
  object-fit: scale-down
---

<blockquote class="blockquote lead">
  <p>
    "In computing, memoization or memoisation is an optimization technique used primarily to speed up 
    computer programs by storing the results of expensive function calls and returning the cached result
    when the same inputs occur again."
  </p>
  <footer class="blockquote-footer"><cite title="Wikipedia on Memoization">Wikipedia on Memoization</cite></footer>
</blockquote>

[Lodash](https://lodash.com/) is a modern JavaScript utility library delivering modularity, performance & extras.
It's a ubiquitous library, and your project probably uses it, even if you're not aware of it.
Lodash contains an implementation of memoization in the form of [memoize](https://lodash.com/docs/4.17.15#memoize) function. 

By default, memoize function only uses the first argument provided to the memoized function as the cache key. 
Here is the simplest possible example of memoization:

{% highlight javascript linenos %}
const { memoize } = require('lodash');

const func = (arg1) => `My name is ${arg1}`
const funcM = memoize(func);

console.dir(funcM('John')); // => 'My name is John'
console.dir(funcM('John')); // => 'My name is John' (cache hit)
console.dir(funcM.cache.size); // => 1
{% endhighlight %}

It's important to realize that the cache key of the memoized function is determined using [SameValueZero](https://tc39.es/ecma262/multipage/abstract-operations.html#sec-samevaluezero) comparison:

{% highlight javascript linenos %}
const { memoize } = require('lodash');

const func = (arg1) => JSON.stringify(arg1)
const funcM = memoize(func);

console.dir(funcM({a: 1})); // => '{"a":1}'
console.dir(funcM({a: 1})); // => '{"a":1}'
console.dir(funcM.cache.size); // => 2
{% endhighlight %}

Now, what if we introduce more arguments to our func? 

{% highlight javascript linenos %}
const { memoize } = require('lodash');

const func = (arg1, arg2) => `${arg1} ${arg2}`;
const funcM = memoize(func);

console.dir(funcM('John', 'Doe')); // => "John Doe"
console.dir(funcM('John', 'FooBar')); // => "John Doe" (cache hit)
console.dir(funcM.cache.size); // => 2
{% endhighlight %}

We can see that there is an obvious problem. As stated above, lodash only uses the first argument
as the cache key. To fix this, we have to use resolver function:

{% highlight javascript linenos %}
const { memoize } = require('lodash');

const func = (arg1, arg2) => `${arg1} ${arg2}`;
const resolver = (arg1, arg2) => JSON.stringify([arg1, arg2]);
const funcM = memoize(func, resolver);

console.dir(funcM('John', 'Doe')); // => "John Doe"
console.dir(funcM('John', 'FooBar')); // => "John FooBar"
console.dir(funcM.cache.size); // => 2
{% endhighlight %}

This resolver solution obviously wouldn't scale. What if our func needs to consume complex objects as arguments?
That's fine unless the object arguments are huge or contain a cyclic reference.

When arguments are huge objects, we'll spend too much CPU time serializing those objects into JSON strings 
to determine the cache key. And instead of the SameValueZero comparison for objects, we'll be comparing huge strings.
When object arguments contain a cyclic reference, we'll not be able to serialize them into JSON string and determine the cache key.

Lodash has one remaining trick up its sleeve to overcome these two problems - being able to influence how 
cache keys are maintained. To achieve our goal, we'll have to create a specialization of `lodash.memoize`;
the function called `memoizeN`:

{% highlight javascript linenos %}
const { memoize } from "lodash";

const sameValueZero = (x, y) => x === y || (Number.isNaN(x) && Number.isNaN(y));
const shallowArrayEquals = (a) => (b) => {
  return Array.isArray(a) && Array.isArray(b)
    && a.length === b.length
    && a.every((val, index) => sameValueZero(val, b[index]));
};
const list = (...args) => args;

class Cache extends Map {
  delete(key) {
    const keys = Array.from(this.keys());
    const foundKey = keys.find(shallowArrayEquals(key));
    return super.delete(foundKey);
  }
  get(key) {
    const keys = Array.from(this.keys());
    const foundKey = keys.find(shallowArrayEquals(key));
    return super.get(foundKey);
  }
  has(key) {
    const keys = Array.from(this.keys());
    return keys.findIndex(shallowArrayEquals(key)) !== -1;
  }
}

const memoizeN = (fn, { resolver = list } = {}) => {
  const { Cache: OriginalCache } = memoize;
  memoize.Cache = Cache;
  const memoized = memoize(fn, resolver);
  memoize.Cache = OriginalCache;
  return memoized;
};
{% endhighlight %}

The real trick here is that the memoized function parameters are transformed into a list, and this list forms a cache key.
We override the `lodash.memoize.Cache` class on an ad-hoc basis with our implementation capable of doing cache key
lookups for our list cache keys.

Observe the following example:

{% highlight javascript linenos %}
const complexObj = {a: 1};
const simpleObj = {b: 2};

const func = (arg1, arg2) => arg2;
const resolver = (arg1, arg2) => [arg1, JSON.stringify(arg2)];
const funcM = memoizeN(func, { resolver });

console.dir(funcM(complexObj, simpleObj)); // => {b: 2}
console.dir(funcM(complexObj, {b: 2})); // => {b: 2} (cache hit)
console.dir(funcM({a: 1}, {b: 2})); // => {b: 2}
console.dir(funcM.cache.size); // => 2
{% endhighlight %}

We intend to memoize func using both arguments to form the cache key. For the first argument,
use the SameValueZero comparison on the original value. For the second one, use SameValueZero comparison
on JSON stringified value.

We can go even further with the specialization to make `memoizeN` consume a list of comparator functions
applied to appropriate arguments instead of abusing a resolver to serialize some arguments.

The implementation will look as follows:

{% highlight javascript linenos %}
const memoize = require('lodash/memoize');
const isEqual = require('lodash/isEqual');

const sameValueZero = (x, y) => x === y || (Number.isNaN(x) && Number.isNaN(y));
const makeShallowArrayEquals = (comparators) => (a) => (b) => {
  return Array.isArray(a) && Array.isArray(b)
    && a.length === b.length
    && a.every((val, index) => {
      const comparator = typeof comparators === 'function' ? comparators : comparators[index] || sameValueZero;
      return comparator(val, b[index]);
    });
};
const list = (...args) => args;

class Cache extends Map {
  shallowArrayEquals = makeShallowArrayEquals(sameValueZero);

  delete(key) {
    const keys = Array.from(this.keys());
    const foundKey = keys.find(this.shallowArrayEquals(key));
    return super.delete(foundKey);
  }
  get(key) {
    const keys = Array.from(this.keys());
    const foundKey = keys.find(this.shallowArrayEquals(key));
    return super.get(foundKey);
  }
  has(key) {
    const keys = Array.from(this.keys());
    return keys.findIndex(this.shallowArrayEquals(key)) !== -1;
  }
}

const memoizeN = (fn, { resolver = list, comparators = sameValueZero }) => {
  const { Cache: OriginalCache } = memoize;
  memoize.Cache = Cache;
  const memoized = memoize(fn, resolver || list);
  memoized.cache.shallowArrayEquals = makeShallowArrayEquals(comparators);
  memoize.Cache = OriginalCache;
  return memoized;
};

module.exports = memoizeN;
{% endhighlight %}

This version of memoizeN applies a specific comparator algorithm for each argument:

{% highlight javascript linenos %}
const obj1 = {a: 1};
const obj2 = {b: 2};
const obj3 = {c: 3};

const func = (arg1, arg2, arg3) => arg3;
const strictEquality = (x, y) => x === y;
const sameValueZero = (x, y) => x === y || (Number.isNaN(x) && Number.isNaN(y));
const deepEquality = lodash.isEqual;
const comparators = [strictEquality, sameValueZero, deepEquality];
const funcM = memoizeN(func, { comparators });

console.dir(funcM(obj1, obj2, obj3)); // => {c: 3}
console.dir(funcM(obj1, obj2, {c: 3})); // => {c: 3} (cache hit)
console.dir(funcM(obj1, {b: 2}, obj3); // => {c: 3}
console.dir(funcM.cache.size); // => 2
{% endhighlight %}

If we define `comparators` option as a function, the memoized function will use it to compare every argument. 
When comparators are not provided, SameValueZero comparison is used for all arguments.

{% highlight javascript linenos %}
const obj1 = {a: 1};
const obj2 = {b: 2};
const obj3 = {c: 3};

const func = (arg1, arg2, arg3) => arg3;
const sameValueZero = (x, y) => x === y || (Number.isNaN(x) && Number.isNaN(y));
const funcM = memoizeN(func, { comparators: sameValueZero }); // equivalen with memoizeN(func)

console.dir(funcM(obj1, obj2, obj3)); // => {c: 3}
console.dir(funcM(obj1, obj2, obj3)); // => {c: 3} // cache hit
console.dir(funcM.cache.size); // => 1
{% endhighlight %}

I've extracted `memoizeN` into [GitHub Gist](https://gist.github.com/char0n/2e6c77d38cf00faacceaf37e46b76a32) installable via npm. 
If you prefer to use `memoizeN` as npm package,
here is how to achieve that:

{% highlight shell %}
 $ npm install gist:2e6c77d38cf00faacceaf37e46b76a32 --save-dev
{% endhighlight %}

This command will create the following entry inside your dependencies:

{% highlight json %}
"dependencies": {
  "@char0n/memoizeN": "gist:2e6c77d38cf00faacceaf37e46b76a32"
}
{% endhighlight %}

Then use it via CommonJS or ESM:

{% highlight javascript %}
const memoizeN = require('@char0n/memoizeN');
// or
import memoizeN from '@char0n/memoizeN';
{% endhighlight %}

<div class="list-group mb-3">
  <a href="https://gist.github.com/char0n/2e6c77d38cf00faacceaf37e46b76a32" class="list-group-item list-group-item-action">
    <div class="d-flex w-100 justify-content-between">
      <h3 class="h5 mb-1"><i class="fab fa-github"></i> memoizeN</h3>
    </div>
    <blockquote class="blockquote fs-6 mb-1">
      Advanced memoization for JavaScript functions with lodash.memoize.
    </blockquote>
    <script type="application/ld+json">
      {
        "@context": "https://schema.org",
        "@type": "SoftwareSourceCode",
        "author": { "@id": "{{ site.url }}" },
        "name": "memoizeN",
        "abstract": "Advanced memoization for JavaScript functions with lodash.memoize.",
        "codeRepository": "https://gist.github.com/char0n/2e6c77d38cf00faacceaf37e46b76a32"
      }
    </script>
  </a>
</div>

If you need even more advanced functionality like LRU cache, TTL, reference counting, and others,
I suggest you look at [memoizee](https://github.com/medikoo/memoizee). memizee is probably the most
feature-rich implementation of memoization for JavaScript.

---

<div class="alert alert-info" role="alert">
  All code snippets in this article are licensed under <a href="https://www.apache.org/licenses/LICENSE-2.0">Apache 2.0 license</a> using the following copyright notice: <strong>Copyright 2022 Vladimír Gorej</strong>.
</div>