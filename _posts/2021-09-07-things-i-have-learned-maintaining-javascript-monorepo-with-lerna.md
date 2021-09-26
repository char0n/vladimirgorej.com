---
layout: post
title: "Things I have learned while maintaining JavaScript monorepo with Lerna"
description: "Monorepo management processes using Lerna and npm@7 workspaces"
date: 2021-09-07 15:15:00 +0300
image:
  path: assets/img/blog/taming-lerna.png
  width: 467
  height: 335
  caption: Taming monorepos with Lerna and npm@7 workspaces
---

<p class="lead">
  This is the second part of the article series dealing with <i>monorepo</i> management using <a href="https://github.com/lerna/lerna">Lerna</a>.
  <a href="https://www.linkedin.com/pulse/things-i-wish-had-known-when-started-javascript-monorepo-gorej/">The first part</a> dealt
  with <i>monorepo</i> management in depth using Lerna and <a href="https://docs.npmjs.com/cli/v7/configuring-npm/pacage-json#local-paths">npm local paths</a>
  long before npm@7 workspaces were introduced. This article will try to describe <i>monorepo</i> management practices
  in the world of <a href="https://docs.npmjs.com/cli/v7/using-npm/workspaces">npm@7</a> workspaces. Nevertheless, I highly recommend you read the first part
  of this series to be familiar with how things used to be.
</p>

## What is monorepo?

Let's first define what *monorepo* is:

<blockquote class="blockquote">
  <p>Monorepo is a software development strategy where code for many projects is stored in the same repository.</p>
</blockquote>

*Monorepo* code is usually divided into separate contexts, called packages. Every package contains code specific to its context and can depend on zero or more other packages within monorepo. Monorepo must have a mechanism to interconnect packages and allow us to use one package code within another.

In the following texts, I'm assuming you have your *monorepo* setup as described in the first part of this series.

## Dependency management

Dependency managed has been simplified significantly with npm workspaces. npm workspaces require npm@7.
This version of npm uses the [new format](https://github.blog/2021-02-02-npm-7-is-now-generally-available/#changes-to-the-lockfile) of a `package-lock.json` file.
You'll need to convert your `package-lock.json` file to the new format for workspaces to work correctly.

### A word on expressing dependencies between packages

We now use **asterisk** symbol to express dependencies between different *monorepo* packages.
Given that following are dependencies of `package-c` which, requires `package-a` and `package-b`,
this is how we express it in `package.json`:

{% highlight json linenos %}
"dependencies": {
  "ramda": "=0.27.0",
  "ramda-adjunct": "=2.27.0",
  "package-a": "*",
  "package-b": "*"
}
{% endhighlight %}

Notice that we no longer use explicit npm local paths but rather an **asterisk** symbol.
**Asterisk** symbol is translated to npm local paths under the hood in a `package-lock.json` file.

Before npm workspaces, we had to list all packages in the `dependencies` field of the main *monorepo* `package.json` file.

{% highlight json linenos %}
"dependencies": {
  "package-a": "file:packages/package-a",
  "package-b": "file:packages/package-b",
  "package-c": "file:packages/package-c"
}
{% endhighlight %}

This is no longer required. npm workspaces support recognizing *monorepo* packages by defining
a `workspaces` field in the main *monorepo* `package.json` file:

{% highlight json linenos %}
"workspaces": [
  "./packages/*"
]
{% endhighlight %}

We don't need to add a new package explicitly in the `dependency` field whenever we want to add one. This saves
time and prevents cryptic errors when we forget to add the package explicitly as a dependency.

### A word on development dependencies

It is no longer required to keep all development dependencies in main the *monorepo* `package.json` file.
Every *monorepo* package can now have its own development dependencies in a particular version, given
the specific needs. 

Before npm workspaces, the `devDependencies` field of every package `package.json` file was ignored.
Now, npm@7 recognizes it and installs the npm packages listed in `devDependencies`.

IMHO it still makes sense to keep certain `devDepedendies` (like testing frameworks) in the main
*monorepo* `package.json` file.

### A word on topology

npm workspaces are aware of *monorepo* package topology. Workspaces know that
`package-c` uses `package-a` and `package-b` as its dependencies. Let's run the following command, which runs build npm script
in all the *monorepo* packages:

{% highlight shell %}
 $ npm run build --workspaces
{% endhighlight %}

This command will run `npm run build` for all the *monorepo* packages.
But it seems that npm workspaces don't run the monorepo packages command in order as expected.
Let's say that `package-a` depends on `package-b` and `package-c` depends both on `package-a` and `package-b`.
The order of execution you get from running the command is:

* package-a
* package-b
* package-c

But the correct order of build should be:

* package-b
* package-a
* package-c

Because of this, **Lerna** still plays a significant role here.
Lerna will determine which packages have **build** npm script defined, and then it determines the proper execution order.

{% highlight shell %}
 $ lerna run build
{% endhighlight %}

Order of execution:

* package-b
* package-a
* package-c

### A word on installing a new dependency

npm workspaces completely simplified the process of adding a new production dependency. Let's say
we have a `package-a`, and we want to add a dependency of `ramda-adjunct@2.27.0`. The only thing 
we need to do is run the following command:

{% highlight shell %}
  $ npm install ramda-adjunct@2.27.0 --workspace=package-a --save
{% endhighlight %}

### A word on installing a new development dependency

As already stipulated, development dependencies can be installed either in top-level *monorepo* (suitable for testing frameworks)

{% highlight shell %}
 $ npm install mocha --save-dev
{% endhighlight %}

Or installed as a specific development dependency of a particular *monorepo* package

{% highlight shell %}
 $ npm install rimraf --workspace=package-a --save-dev
{% endhighlight %}


### A word on npm binaries

Before npm workspaces, to run a binary from one of the *monorepo* packages, we had to use the [link-parent-bin](https://www.npmjs.com/package/link-parent-bin)
npm package to link top-level npm binaries to all *monorepo* packages. This is no longer required as npm supports this natively.

### Closing words

Given all that has been written here, I think npm workspaces still doesn't replace Lerna. Instead,
they just significantly simplified how we work with Lerna. Lerna still plays a significant part in
change detection, releases, and running npm scripts in the proper order (given the package topology).

All the examples mentioned in the article can be found in the following Lerna monorepo repository I've setup-ed for you. Clone it, install it, play with it and have a lot of fun with it!

<div class="list-group mb-3">
  <a href="https://github.com/char0n/lerna-monorepo-taming" class="list-group-item list-group-item-action">
    <div class="d-flex w-100 justify-content-between">
      <h3 class="h5 mb-1"><i class="fab fa-github"></i> lerna-monorepo-taming</h3>
    </div>
    <blockquote class="blockquote fs-6 mb-1">
      This monorepo implements current practices and conventions used while building monorepos using Lerna.
    </blockquote>
    <script type="application/ld+json">
      {
        "@context": "https://schema.org",
        "@type": "SoftwareSourceCode",
        "author": { "@id": "https://vladimirgorej.com" },
        "name": "lerna-monorepo-taming",
        "abstract": "This monorepo implements current practices and conventions used while building monorepos using Lerna.",
        "codeRepository": "https://github.com/char0n/lerna-monorepo-taming"
      }
    </script>
  </a>
</div>

--- 

**Maintaining Lerna monorepo series:**

* Part 1: [Things I wish I had known when I started JavaScript monorepo with Lerna](https://www.linkedin.com/pulse/things-i-wish-had-known-when-started-javascript-monorepo-gorej/)
* Part 2: Things I have learned while maintaining JavaScript monorepo with Lerna
