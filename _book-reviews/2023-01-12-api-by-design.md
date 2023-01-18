---
layout: book-review
title: "API by Design"
description: "API by Design"
date: 2023-01-12 10:00:00 +0200
timestamp: 1673514000
image:
  path: assets/img/book-reviews/api-by-design.webp
  width: 528
  height: 827
  caption: "API by Design"
  object-fit: scale-down
book:
  name: "API by Design"
  author: "https://smizell.com/"
  bookFormat: "http://schema.org/EBook"
  datePublished: "2021"
  abstract: "Measure complexity. Make informed decisions. Design and build better APIs."
  inLanguage: English
  isbn: ""
---

<p class="lead">
  I have known the author of this book, Stephen Mizell, for some time now.
  Stephen had already left when I joined the Apiary company, but his ideas and code were everpresent.
  His brilliant ideas are time-proven, and even today, I'm building tools related to APIs on his thoughts
  or libraries. We met again briefly at SmartBear company, where we collaborated on another API-related
  tool called <a href="https://github.com/swagger-api/apidom/">ApiDOM</a>.
</p>

At the beginning of his book, Stephen introduces the topic of API complexity,
how it accumulates in time and how beginners learn to deal with it.
He proposes a mechanism for measuring the complexity and a common language (nomenclature) to discuss it.

He describes the laws of thermodynamics to introduce the term **entropy**,
which is the measurement of less-usable energy within the system.
He invents the term **API entropy** and uses it as a measurement of the complexity of the API.

If API entropy is left unchecked, it spreads to other parts of the system, eventually
leading to **chaos** and affecting future decisions.

He then chooses [JSON Schema](https://json-schema.org/) (schema for short) to introduce the term **Schema entropy**,
which measures schema complexity. Schema entropy is part of overall API entropy and
is a sum of the following qualitative factors:

#### Semantic-schema entanglement

Semantic-schema entanglement happens when the semantics and schema are so
closely intertwined that a change in one requires a change in the other.
This explanation is abstract, but he explains clearly what semantic-schema entanglement
is by comparing how we work with HTML vs. JSON.

#### Schema variation

Schema variation is the total number of valid JSON structures for a schema.
Stephen shows us how many schema variations correlate to a higher schema entropy.
He then explains how schema variations are calculated.

#### Schema depth and width

Stephen explains that the depth concept is related to the schema nesting and that the schema is wider, the more properties it has.

#### Reference entanglement
Schemas can be defined in one place and referenced in another document,
and references can be inbound (references to this schema) and outbound (references which this schema references).
Stephen further explains how a change in one schema can change every schema where
it's referenced and defines this entanglement as reference entanglement.

Stephen has a great talent for explaining complex concepts like the qualitative
factors I've enumerated above and precisely defines formulas to calculate qualitative
factors into overall schema entropy measurement. By now, the reader of the book has
a complete understanding of how API entropy affects APIs and our decisions and how to measure it.

Stephen dedicates the rest of the book to API entropy **remediation practices**,
including various API versioning strategies, adopting a microservice architecture,
or narrowing semantic dependence by introducing a **representor pattern**.
He also briefly discusses API coupling and its two characteristics - **surface area** and **strength**, and their effect on API entropy.

At the end of the book, Stephen wrote a real-world case study of GitHub API, where he examined most of the concepts introduced by this book.

API by Design book gave me a mechanism to calculate API entropy and recipes on how to manage it.
It also inspired me to create a tool called ApiDOM Entropy which will eventually land in [ApiDOM monorepo](https://github.com/swagger-api/apidom/).

This book is **worth every cent**, it costs <strong><var><abbr title="USD">$</abbr>15</var></strong>, and you can buy it [here](https://smizell.com/books/api-by-design/).
