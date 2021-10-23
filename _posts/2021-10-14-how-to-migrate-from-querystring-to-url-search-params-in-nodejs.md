---
layout: post
title: "How to migrate from querystring to URLSearchParams in Node.js"
description: "Node.js marked the querystring as legacy API and recommends using URLSearchParams. But doesn't give us any clue how to actually migrate. This article fills this void and provides a migration guide."
date: 2021-10-14 10:00:00 +0300
image:
  path: assets/img/blog/querystring-migration.webp
  width: 870
  height: 580
  caption: Photo by Maksim Shutov (@maksimshutov)
  object-fit: fill
---

<p class="lead">
  Node.js marked the <a href="https://nodejs.org/docs/latest-v16.x/api/querystring.html">querystring</a> as legacy API in version 14.x, 
  and recommends using <a href="https://nodejs.org/docs/latest-v16.x/api/url.html#url_class_urlsearchparams">URLSearchParams</a>. 
  But doesn't give us any clue how to actually migrate. This article fills this void and provides a migration guide.
</p>

### querystring

*querystring* API is quite simple and straightforward:

<dl class="row">
  <dt class="col-sm-3">querystring.parse()</dt>
  <dd class="col-sm-9">Parses a URL query string into a collection of key and value pairs. <span class="text-muted">querystring.decode()</span> is an alias.</dd>

  <dt class="col-sm-3">querystring.stringify()</dt>
  <dd class="col-sm-9">Produces a URL query string from a given obj by iterating through the object's "own properties". <span class="text-muted">querystring.encode()</span> is an alias.</dd>

  <dt class="col-sm-3">querystring.escape()</dt>
  <dd class="col-sm-9">Performs URL percent-encoding on the given string in a manner that is optimized for the specific requirements of URL query strings. Is used by <span class="text-muted">querystring.stringify()</span>.</dd>

  <dt class="col-sm-3">querystring.unescape()</dt>
  <dd class="col-sm-9">Performs decoding of URL percent-encoded characters on the given string. Is used by <span class="text-muted">querystring.parse().</span></dd>
</dl>

##### Parse a URL query string

{% highlight javascript %}
const { parse } = require('querystring');

parse('foo=bar&abc=xyz&abc=123');

// returns
// { 
//   foo: 'bar',
//   abc: ['xyz', '123']
// }
{% endhighlight %}

##### Produce a URL query string

{% highlight javascript %}
const { stringify } = require('querystring');

stringify({ foo: 'bar', baz: ['qux', 'quux'], corge: '' });

// returns 'foo=bar&baz=qux&baz=quux&corge='
{% endhighlight %}

##### Perform URL percent-encoding on the given string

{% highlight javascript %}
const { escape } = require('querystring');

escape('str1 str2');

// returns 'str1%20str2'
{% endhighlight %}

##### Perform decoding of URL percent-encoded characters on the given string

{% highlight javascript %}
const { unescape } = require('querystring');

escape('str1%20str2');

// returns 'str1 str2'
{% endhighlight %}

### URLSearchParams

What's the actual difference between `querystring` vs. `URLSearchParams`?

<blockquote class="blockquote">
  <p>"The WHATWG <em>URLSearchParams</em> interface and the <em>querystring</em> module have similar purpose, but the purpose of the <em>querystring</em> module is more general, as it allows the customization of delimiter characters."</p>
  <footer class="blockquote-footer"><cite title="Node.js 14.x documentation">Node.js 14.x documentation</cite></footer>
</blockquote>

Working with URIs/URLs in JavaScript was always confusing. [WHATWG URL specification](https://url.spec.whatwg.org/) finally
standardizes the term URL and provides a standardized way how we work with URLs. *URLSearchParams* is part of this specification.
Now let's migrate our previous *querystring* examples to *URLSearchParams* API.

##### Parse a URL query string

{% highlight javascript %}
const params = new URLSearchParams('foo=bar&abc=xyz&abc=123');
// URLSearchParams { 'foo' => 'bar', 'abc' => 'xyz', 'abc' => '123' }

params.get('foo'); // 'bar'
params.getAll('abc') // ['xyz', '123']
{% endhighlight %}

##### Produce a URL query string

{% highlight javascript %}
new URLSearchParams({ foo: 'bar', baz: ['qux', 'quux'], corge: '' }).toString();
// returns 'foo=bar&baz=qux%2Cquux&corge='
{% endhighlight %}

As we can see, what we got is a different result compared to *querystring*.

<div class="table-responsive">
  <table class="table">
    <thead>
      <tr>
        <th scope="col">API</th>
        <th scope="col">Result</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td>querystring</td>
        <td>foo=bar&baz=qux&baz=quux&corge=</td>
      </tr>
      <tr>
        <td>URLSearchParams</td>
        <td>foo=bar&baz=qux%2Cquux&corge=</td>
      </tr>
    </tbody>
  </table>
</div>

<blockquote class="blockquote">
  <p>"Unlike <em>querystring</em> module, duplicate keys in the form of array values are not allowed. Arrays are stringified using array.toString(), which simply joins all array elements with commas."</p>
  <footer class="blockquote-footer"><cite title="Node.js 14.x documentation">Node.js 14.x documentation</cite></footer>
</blockquote>

To get the result compatible with *querystring*, we have to initialize *URLSearchParams* with a list of tuples 
instead of object:

{% highlight javascript %}
new URLSearchParams([
  ['foo', 'bar'],
  ['baz', 'qux'],
  ['baz', 'quux'],
  ['corge', ''],
]).toString();
// returns 'foo=bar&baz=qux&baz=quux&corge='
{% endhighlight %}


##### Perform URL percent-encoding on the given string

There is no low-level API for encoding simple strings in *URLSearchParams*.
We have to be a little creative to achieve URL encoding. Below
are 2 different yet functionally equivalent ways how to achieve the same result.

{% highlight javascript %}
new URLSearchParams([['', 'str1 str2']]).toString().slice(1);
new URLSearchParams({'': 'str1 str2'}).toString().slice(1);
// returns 'str1+str2'
{% endhighlight %}

Again, we can see what we got is a different result compared to *querystring*. WHATWG URL specification
is now precise about how various segments of URL gets encoded. Spaces in query parameters are now plus-sign (`+`) encoded, 
compared to percent-encoded spaces in the path segment of the URL. This currently accepted behavior is **backward incompatible** with *querystring*,
and there is nothing you can do about it except to take it.

<table class="table">
  <thead>
    <tr>
      <th scope="col">API</th>
      <th scope="col">Result</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>querystring</td>
      <td>str1%20str2</td>
    </tr>
    <tr>
      <td>URLSearchParams</td>
      <td>str1+str2</td>
    </tr>
  </tbody>
</table>

##### Perform decoding of URL percent-encoded characters on the given string

There is no low-level API for decoding a simple string in *URLSearchParams*.
Again we have to be creative to achieve URL decoding on the given string. 

{% highlight javascript %}
const urlPlusEncodedString = 'str1+str2';

new URLSearchParams(`=${urlPlusEncodedString}`).get('');
// returns 'str1 str2'
{% endhighlight %}

URLSearchParams will handle percent-encoded spaces as well:

{% highlight javascript %}
const urlPercentEncodedString = 'str1%20str2';

new URLSearchParams(`=${urlPercentEncodedString}`).get('');
// returns 'str1 str2'
{% endhighlight %}

### Other differences

**Handling of question mark character**

*URLSearchParams* removes question mark character from the start of the URL query string. *querystring*
keeps it and makes it part of the key name.

{% highlight javascript %}
const { parse } = require('querystring');

parse('?foo=bar'); // returns { '?foo': 'bar' }
new URLSearchParams('?foo=bar'); // returns URLSearchParams { 'foo' => 'bar' }
{% endhighlight %}

*Have you found other differences? I'd be happy to add them to this list.*

### Closing words

*querystring* is a legacy Node.js API that shouldn't be used anymore. New code should only
use *URLSearchParams*, which is part of **WHATWG URL specification**. Old code using *querystring*
can easily be migrated to *URLSearchParams* using this guide. The only **backward incompatibility**
between the two APIs is that *URLSearchParams* use plus-encoded spaces instead of percent-encoded
spaces in *querystring*.