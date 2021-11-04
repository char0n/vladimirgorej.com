---
layout: post
title: "Ramda Adjunct"
description: "Ramda Adjunct is the most popular and most comprehensive set of functional utilities for use with Ramda.js, providing a variety of useful, well-tested functions with excellent documentation. This article describes the history of Ramda Adjunct's creation."
date: 2017-02-23 10:00:00 +0200
image:
  path: assets/img/blog/ramda-adjunct.webp
  width: 819
  height: 600
  caption: Ramda Adjunct
  object-fit: cover
---

<p class="lead">
  Ramda Adjunct is the most popular and most comprehensive set of functional utilities for use with Ramda, providing a variety of useful, well-tested functions with excellent documentation.
  This article describes the history of Ramda Adjunct's creation. 
</p>

Functional point-free programming is all about composition. When I'm building an application nowadays,
I use composition from top to bottom. I compose functions to create modules, modules to compose features, 
and features to compose applications. For composing functions themselves, I use functional libraries.
Namely, my two most favorite [lodash](https://lodash.com/) and [Ramda](https://ramdajs.com/).

While using these functional libraries, I found out that in every new project I was working on, 
I tend to create the same set of aggregate functions that I missed either from lodash or Ramda.
Let me demonstrate this in a code snippet...

{% highlight javascript linenos %}
const { isString, negate } = require('lodash/fp');
const { isNil, complement } = require('ramda');

const isNotString = negate(isString);
const isNotNil = complement(isNil);
{% endhighlight %}

After realizing that I had to redefine these functions in every project, I opened my browser and
searched for extensions to lodash and Ramda. I decided to narrow my search to Ramda only because 
it is missing more functions related to my use-cases.

**My search criteria were:**

- timestamp of the last commit
- 100% test coverage
- quality documentation similar to Ramda

**The results**:

I could not find anything solid I could use. I found multiple fragmented libraries; some didn't have tests at all, 
some did not have any documentation, some were buggy, and some were focused on Promises, etc...
Of course, I could use a combination of multiple libraries, but again, I will not build software on 
libraries without any test coverage.

Hence [Ramda Adjunct](https://github.com/char0n/ramda-adjunct) was born.

<div class="list-group mb-3">
  <a href="https://github.com/char0n/ramda-adjunct" class="list-group-item list-group-item-action">
    <div class="d-flex w-100 justify-content-between">
      <h3 class="h5 mb-1"><i class="fab fa-github"></i> ramda-adjunct</h3>
    </div>
    <blockquote class="blockquote fs-6 mb-1">
      Ramda Adjunct is the most popular and most comprehensive set of functional utilities for use with Ramda, providing a variety of useful, well tested functions with excellent documentation.
    </blockquote>
    <script type="application/ld+json">
      {
        "@context": "https://schema.org",
        "@type": "SoftwareSourceCode",
        "author": { "@id": "{{ site.url }}" },
        "name": "ramda-adjunct",
        "abstract": "Ramda Adjunct is the most popular and most comprehensive set of functional utilities for use with Ramda, providing a variety of useful, well tested functions with excellent documentation.",
        "codeRepository": "https://github.com/char0n/ramda-adjunct"
      }
    </script>
  </a>
</div>

**Ramda Adjunct** is based on three main principles:
 
- centralization
- 100% test coverage
- impeccable documentation

### Centralization

All Ramda recipes and aggregate utils not present in Ramda are centralized here. 
There is no more need for everybody to create their own utils in their own codebases.

### 100% test coverage

Creating custom aggregate utils or implementing recipes from the Ramda wiki creates the defectiveness problem.
The problem is caused by the absence of any tests. Ramda Adjunct keeps [100% code coverage](https://app.codecov.io/gh/char0n/ramda-adjunct) and mimics the Ramda 
test patterns.

### Impeccable documentation

You cannot call a library great without excellent documentation. Ramda Adjunct generates its [documentation](https://char0n.github.io/ramda-adjunct/)
directly from its codebase and uses patterns found in Ramda and Lodash to document its API.

### What are the future plans?

My first priority for the near feature is to absorb obsolete and no longer maintained [Ramda Cookbook implementation](https://github.com/enten/rcb)
and implement all `Type` related functions that I'm missing in Ramda. 
After that, I would like to continue absorbing other `dead` libraries.
Ramda Adjunct repo is ready to receive pull requests. If you have some recipes you would like to share with the community,
please issue a pull request or [create an issue on GitHub](https://github.com/char0n/ramda-adjunct/issues/new/choose).