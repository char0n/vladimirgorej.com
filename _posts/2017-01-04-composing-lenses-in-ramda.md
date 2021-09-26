---
layout: post
title: "Composing lenses in Ramda"
description: "Ramda contains the implementation of one of my favorite functional concepts - functional lenses. Lenses are part of category theory and allow us to focus on a particular piece/path of a complex data object."
date: 2017-01-04 10:00:00 +0200
image:
  path: assets/img/blog/composing-lenses-in-ramda.jpg
  width: 1280
  height: 720
  caption: Composing lenses in Ramda
---

<p class="lead">
  If you are a fan of point-free functional programming in Javascript, you will inevitably become a fan of <a href="https://ramdajs.com/">Ramda</a>.
  Ramda is a functional library with an emphasis on immutability and side-effect-free functions.
</p>

Ramda contains the implementation of one of my favorite functional concepts - **functional lenses**.
Lenses are part of category theory and allow us to focus on a particular piece/path of a complex data object.

{% highlight js linenos %}
import { lensPath, view } from 'ramda';

const complexObject = { level1: { level2: { prop1: 1, prop2: 2 } } };
const prop1Lens = lensPath(['level1', 'level2', 'prop1']);

console.assert(view(prop1Lens, complexObject) === 1);
{% endhighlight %}

As you can see, lenses are a pretty powerful concept. But as you went through the code, it seems that you are constrained to creating particular lenses for particular use-cases.
But that is a wrong assumption, as I found out myself. What I am about to show you is not documented even in Ramda documentation or its recipes.

Lenses are basically just ordinary functions, and as a regular function, they can be composed. 
Lenses themselves are **compo-sable**. I call them **combined lenses**. Here is the code snipped that is self-explanatory.

Lenses are basically just ordinary functions, and as an ordinary function, they can be composed. 
Lenses themselves are **compo-sable**. I call them **combined lenses**. Here is the code snipped that is self-explanatory.

{% highlight js linenos %}
import { lensPath, compose, view } from 'ramda';

// Generic lenses
// --------------

const enabledLens = lensPath(['enabled']);

// Combined lenses
// ---------------

const sshServiceLens = lensPath(['sshService']);
const sshServiceEnabledLens = compose(sshServiceLens, enabledLens);

const telnetServiceLens = lensPath(['telentService']);
const telnetServiceEnabledLens = compose(telnetServiceLens, enabledLens);

// Usage
// -----

const services = {
sshService: { enabled: true },
telnetService: { enabled: false },
};

console.assert(view(sshServiceEnabledLens, services) === true);
console.assert(view(telnetServiceEnabledLens, services) === false);
{% endhighlight %}

The key here is to always use **lensPath** instead of lensProp. 
If you use lensProp, you may end up in *TypeError*.
Use compose function from Ramda and compose the lenses from left to right ([why?](http://www.reddit.com/r/haskell/comments/23x3f3/lenses_dont_compose_backwards/)).

Lens composition from **generic** lenses to **combined** lenses is one step further in using the functional lenses. So don't waste your time and try it in your next project.

<div class="list-group mb-3">
  <a href="https://github.com/ramda/ramda" class="list-group-item list-group-item-action">
    <div class="d-flex w-100 justify-content-between">
      <h3 class="h5 mb-1"><i class="fab fa-github"></i> ramda</h3>
    </div>
    <blockquote class="blockquote fs-6 mb-1">
      &#128015; Practical functional Javascript
    </blockquote>
    <script type="application/ld+json">
      {
        "@context": "https://schema.org",
        "@type": "SoftwareSourceCode",
        "author": { "@id": "https://vladimirgorej.com" },
        "name": "ramda",
        "abstract": "Practical functional Javascript",
        "codeRepository": "https://github.com/ramda/ramda"
      }
    </script>
  </a>
</div>

**Functional Lenses in JavaScript series:**

- Part 1: [Functional lenses in JavaScript](https://www.linkedin.com/pulse/functional-lenses-javascript-vladim%C3%ADr-gorej/)
- Part 2: [Functional lenses in JavaScript - Isos](https://www.linkedin.com/pulse/functional-lenses-javascript-isos-vladim%C3%ADr-gorej/)
- Part 3: [Functional lenses in JavaScript - Traversables](https://www.linkedin.com/pulse/functional-lenses-javascript-traversables-vladim%C3%ADr-gorej/)
- Part 4: Functional lenses in JavaScript - Folds (unpublished)
- Bonus material: Composing lenses in Ramda