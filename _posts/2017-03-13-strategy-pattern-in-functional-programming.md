---
layout: post
title: "Strategy pattern in functional programming"
description: "The Strategy Pattern is a well-known OOP design pattern. But what does it look like once you port it to functional programming, where functions are first-class objects? A worked example: parsing a USER model from two different external sources."
date: 2017-03-13 10:00:00 +0200
canonical_url: https://www.linkedin.com/pulse/strategy-pattern-functional-programming-vladim%C3%ADr-gorej/
image:
  path: assets/img/blog/strategy-pattern-in-functional-programming.webp
  width: 1004
  height: 450
  caption: Strategy pattern in functional programming
  object-fit: cover
---

<p class="lead">
  During the past couple of weeks I was involved in architecting and implementing a solution for complex model transformations.
  It was part of a wider <strong>MDD</strong> (Model Driven Development) approach I was trying to comprehend and apply to the use case of the project I am currently working on.
  During this endeavor I was confronted with a number of challenges. Eventually it all came down to one common denominator: the <a href="https://en.wikipedia.org/wiki/Strategy_pattern" target="_blank" rel="noopener noreferrer">Strategy Pattern</a>.
</p>

For demonstration purposes, let's reduce my use case to one simple problem. I have a <strong>USER</strong> model that comes from two different sources: a database and a web service.
Each source returns it in a different form, though the forms basically contain the same data. I want to parse these external data forms
into a transitional data form that my application understands. This transitional data form is called the <strong>Correspondence</strong> model (in MDD theory). Once I have both external
models in the Correspondence model, I want to merge them and map the result to the <strong>API</strong> model. The API model is exposed from the application through the JSON REST API.
The merging and mapping is out of scope for this article, so let's stick with the parsing part.

I will now demonstrate how to approach this problem from two perspectives: <strong>OOP</strong> (Object-Oriented Programming) and <strong>FP</strong> (Functional Programming). I have a long history of
involvement with OOP languages and have extensively studied the work of <a href="https://martinfowler.com/" target="_blank" rel="noopener noreferrer">Martin Fowler</a> and the <a href="http://www.blackwasp.co.uk/gofpatterns.aspx" target="_blank" rel="noopener noreferrer">Gang of Four</a>. Since I have longer experience in OOP than in FP, I designed
my first prototype in OOP. Then I moved forward and ported the OOP code into FP, using the fact that in JavaScript all functions are first-class objects.

## OOP version

{% highlight javascript linenos %}
class Parser {
  constructor(parseStrategy) {
    this.parseStrategy = parseStrategy;
  }

  parse(externalModel) {
    return this.parseStrategy.parse(externalModel);
  }
}

class UserDbParser {
  parse(dbUser) {
    return ...correspondenceUser; // This would contain the parsing algorithm.
  }
}

class UserWebServiceParser {
  parse(webServiceUser) {
    return ...correspondenceUser; // This would contain the parsing algorithm.
  }
}

const userDbParser = new Parser(new UserDbParser());
const userWebServiceParser = new Parser(new UserWebServiceParser());

DB.user.findById(1).then(userDbParser.parse.bind(userDbParser));
WebService.user.fetchOne(1).then(userWebServiceParser.parse.bind(userWebServiceParser));
{% endhighlight %}

This is how USER model parsing looks in OOP using the Strategy Pattern. I would probably use some kind of <a href="https://en.wikipedia.org/wiki/Inversion_of_control" target="_blank" rel="noopener noreferrer">IoC container</a> to manage the instances of my classes.

## FP version

{% highlight javascript linenos %}
const { curry } = require('ramda');

const parseDbUser = (dbUser) => { return ...correspondenceUser };
const parseWebServiceUser = (webServiceUser) => { return ...correspondenceUser };

const toCorrespondence = curry(
  (parseStrategy, dbOrWebServiceUser) => parseStrategy(dbOrWebServiceUser)
);

const fromDb = toCorrespondence(parseDbUser);
const fromWebService = toCorrespondence(parseWebServiceUser);

DB.user.findById(1).then(fromDb);
WebService.user.fetchOne(1).then(fromWebService);
{% endhighlight %}

This is how USER model parsing looks in FP using the Strategy Pattern, or at least my version of implementing the Strategy Pattern in FP. I could not find any coherent
articles on the issue to validate my approach. Anyway, the FP version is less verbose, has less overhead, and doesn't need any IoC. To tell you the truth, I immediately
liked it better.

So there it is: the Strategy Pattern in FP. I hope this article saves somebody some time when researching a similar issue. And finally, one wonders how other OOP design
patterns look in FP, right? ;]

## Update (12.08.2017)

<a href="https://www.linkedin.com/in/alex-hart-a33aa011" target="_blank" rel="noopener noreferrer">Alex Hart</a> was kind enough to provide us with <a href="https://gist.github.com/exallium/6e9dfd662b410fd0304338277e12ff0b" target="_blank" rel="noopener noreferrer">an approach to implementing the Strategy Pattern in Haskell</a>.

{% highlight haskell linenos %}
data Parser = WebParser | DbParser

class ParseStrategy p where
  parse :: p -> String -> String

instance ParseStrategy Parser where
  parse WebParser s = "code for parsing web stuff goes here"
  parse DbParser s = "code for parsing db stuff goes here"

fromDB = parse DbParser
fromWeb = parse WebParser

main :: IO ()
main = do
  -- decide which parser to utilize
  let s = fromDB "asdf"
  putStrLn s
{% endhighlight %}

---

<div class="alert alert-info" role="alert">
  All code snippets in this article are licensed under <a href="https://www.apache.org/licenses/LICENSE-2.0" target="_blank" rel="noopener noreferrer">Apache 2.0 license</a> using the following copyright notice: <strong>Copyright 2017 Vladimír Gorej</strong>.
</div>
