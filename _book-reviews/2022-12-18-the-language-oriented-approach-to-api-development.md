---
layout: book-review
title: "The Language-Oriented Approach to API Development"
description: "The Language-Oriented Approach to API Development"
date: 2022-12-18 10:00:00 +0200
timestamp: 1671354000
image:
  path: assets/img/book-reviews/the-language-oriented-approach-to-api-development.webp
  width: 416
  height: 593
  caption: "The Language-Oriented Approach to API Development"
  object-fit: scale-down
book:
  name: "The Language-Oriented Approach to API Development"
  author: "https://smizell.com/"
  bookFormat: "https://schema.org/EBook"
  datePublished: "2021"
  abstract: "Shifting the focus away from the technical details of API design toward the conversational language."
  inLanguage: English
  isbn: ""
---

<p class="lead">
  Stephen Mizell, the author of this book, is one of my favorite authors.
  The Language-Oriented Approach to API Development is the second book of his
  I will be reviewing (the first is called <a href="{% link _book-reviews/2023-01-12-api-by-design.md %}">API By Design</a>). I was privileged to be able
  to read the draft of this book before it was published. I'm also honored that
  I was mentioned in the book's <a href="https://smizell.com/language-oriented-approach/acknowledgments.html">Acknowledgments</a> section and indirectly contributed
  to Stephen's thought process.
</p>

It all started when I got Stephen's email saying he was working on a book on a
language-oriented approach to API development. He mentioned that my work on [ApiDOM](https://github.com/swagger-api/apidom)
and [SwaggerEditor@5](https://github.com/swagger-api/swagger-editor/tree/next) might help him.
He introduced me to the term **Language Workbenches** and provided a link
to [Martin Fowler's article explaining Language Workbenches](https://github.com/swagger-api/swagger-editor/tree/next).
Long story short, Language Workbenches are tools for Domain Specific Languages. He also attached
the draft of his *The Language-Oriented Approach to API Development* book to the same email.

The book starts by explaining Domain Specific Language in the context of API Design.
The organization can create a tailored language for its API Design, which captures
the conversational language of the API design process. This tailored API design language
is a domain-specific language or DSL. Stephen proposes that instead of describing APIs directly
in OpenAPI, we can use tailored API Design language and generate the OpenAPI (or any other industry standard)
from it. The language-oriented approach doesn't replace the need for OpenAPI.
Instead, it shifts the API design process await from it.

He then describes how a language-oriented approach can remediate the challenges of directly
authoring OpenAPI. The language-oriented approach encourages organizations to build an API
program by defining their conversational language and creating a tailored DSL around it.
This DLS is used to design APIs. The organization then implements the tooling to translate the
DSL into industry-standard OpenAPI, embedding its API governance model as part of the translation process.
The tooling usually consists of the following components:

- **DSL** (tailored API Design Language)
- **editor** - helps API Designers to write the DSL
- **generator** - translates the DSL into a representation (OpenAPI)

Stephen dedicates the following two chapters of the book to enumerating the **benefits** and **tradeoffs** of
the language-oriented approach. Naturally, the benefits chapter is twice as long as the tradeoffs one.
Still, the author is very objective about all the tradeoffs. One tradeoff that stands apart
is the creation of a workflow that is counter to the industry - organizations build their tools
rather than purchase turnkey software.

Examples are worth gold, so the author dedicates a whole next chapter to the [real-world DSL
that he invented](https://github.com/smizell/language-oriented-example) and uses this DSL
to build an API to keep track of automobiles. Stephen demonstrates how writing nine lines in
the DSL generates 43 lines of OpenAPI definition with embedded rules for his associated API governance model.

I took it upon myself to add support for Stephens DSL to [SwaggerEditor@5](https://github.com/swagger-api/swagger-editor/tree/next) via a plugin.
You can see the result on this [URL](https://swagger-editor-dsl.surge.sh/). As you write the DSL, it is translated in real-time
by a generator component to the OpenAPI definition and translated OpenAPI renderers
in the right pane via SwaggerUI. You can find a code of the plugin that adds support for this
tailored DSL into SwaggerEditor@5 on [GitHub](https://github.com/swagger-api/swagger-editor/commit/8b9b8f48a3d1174b2c8e67e4b57b3504dc23b91f).

How to get started with a language-oriented approach, and when do you use it?
Stephen deals with those questions in the three following chapters.
Read these chapters carefully when considering adopting a language-oriented approach within
your organization.

At the end of the book, the author provides several case studies and examples of
language-oriented approaches. Examples include (among others) Amazons [Smithy](https://smithy.io/2.0/index.html),
MicroSofts [TypeSpec](https://microsoft.github.io/typespec/) + [Cadl](https://github.com/microsoft/cadl),
and Mike Amundsens [ALPS](http://alps.io/). This range of tailored DSLs shows that the
language-oriented approach is a hot topic, given that significant market players already
provide tailored DSL and tooling around it to solve the challenges Stephen talks about in his book.

I recommend reading this book to all people involved in any stage of the API design-first
approach process who feel set up against a cascade of challenges where a solution to one
problem leads to more.

You can read this book free of charge on [Stephens's website](https://smizell.com/language-oriented-approach/).
You can also easily print it into PDF and read it in your favorite reader.
