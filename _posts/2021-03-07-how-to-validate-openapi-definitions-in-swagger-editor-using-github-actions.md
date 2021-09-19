---
layout: post
title: "How to validate OpenAPI definitions in Swagger Editor using GitHub Actions"
description: "How to validate OpenAPI definitions in Swagger Editor using GitHub Actions"
date: 2021-03-07 10:39:04 +0200
image:
  path: assets/img/blog/swagger-editor-validate.png
  width: 536
  height: 188
  caption: Swagger Editor Validate - GitHub Action
---

<p class="lead">
  The OpenAPI Specification (OAS) defines a standard, language-agnostic interface to HTTP APIs, 
  allowing both humans and computers to discover and understand the service's capabilities without
  access to source code, documentation, or network traffic inspection. 
  When properly defined, a consumer can understand and interact with the remote service with minimal implementation logic.
</p>

An OpenAPI definition can then be used by documentation generation tools to display the API, code generation
tools to generate servers and clients in various programming languages, testing tools, and many other use cases.

I took the above description of what OpenAPI is directly from [its specification](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.1.0.md).
If you're one of the people who write OpenAPI definition by hand in [Swagger Editor](https://editor.swagger.io/), this article was written just for you.

A couple of years ago, I worked with [Ubiquiti Inc](https://www.ui.com/). We were producing great wireless hardware solutions
with embedded UIs. I was fortunate to have joined the company when a cross hardware configuration system
called [UNMS](https://unms.com/) was being architected and developed. The backend part of the system consisted of a vast [REST API](https://en.wikipedia.org/wiki/Representational_state_transfer) layer that 
the Frontend layer was consuming. We used a design-first approach to producing the API. The workflow of this 
[design-first approach](https://www.visual-paradigm.com/guide/development/code-first-vs-design-first) consisted of the following steps:

- create changes in [OpenApi 2.0](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/2.0.md) definition using Swagger Editor
- issuer: issue a [GitHub Pull Request](https://docs.github.com/en/github/collaborating-with-issues-and-pull-requests/about-pull-requests) for the team to review how new API endpoints could look like
- reviewer: paste the OpenAPI definition from Pull Request to Swagger Editor to see if no errors were introduced
- merge Pull Request
- implement the actual REST API defined in OpenAPI definition

Today I'd immediately think about how to automate this workflow and make [Continuous Integration](https://en.wikipedia.org/wiki/Continuous_integration) do the work for us.
Unfortunately, at that time, my knowledge about CI/CD pipelines was limited, and I wasn't aware of the full benefits 
and values CI/CD provides.

Some time has passed since then. I did my homework and studied CI/CD topic thoroughly. When [GitHub Actions](https://github.com/features/actions) were born
I immediately became a massive fan. Let's try to automate the workflow described above using GitHub Actions.

### Technical design

We need to produce a GitHub Action that uses Swagger Editor to validate the OpenAPI definition provided
as a parameter to that action. Using a Swagger Editor in GitHub Action can be achieved in two ways:
running it in a docker container using [swagger-api/swagger-editor image](https://hub.docker.com/r/swaggerapi/swagger-editor/) or using [https://editor.swagger.io/](https://editor.swagger.io/) directly.
Now that we have Swagger Editor running, we use [puppeteer](https://pptr.dev/) to open a headless version of Chromium Browser.
Then we paste the OpenAPI definition into the Swagger Editor and wait for it to render errors
(if the definition is valid, no errors will be generated). Read errors from the Swagger Editor using puppeteer 
and represent them via GitHub Actions API. GitHub Action will report failure on any error or success if
the document is valid and contains no errors.

Why use Swagger Editor and not just JSON Schema validation? Swagger Editor adds its own layer of error recognition on top of JSON Schema validation.
Unfortunately, this additional layer is tightly coupled to Swagger Editor code, and the easiest way to use it is to use it via running
the Swagger Editor in a browser.

### Implementation

The implementation wasn't that straightforward as I expected, but I still managed to implement this GitHub Action
against the technical design mentioned earlier with all its requirements. I called it [swagger-editor-validate](https://github.com/char0n/swagger-editor-validate).

<div class="list-group mb-3">
  <a href="https://github.com/char0n/swagger-editor-validate" class="list-group-item list-group-item-action">
    <div class="d-flex w-100 justify-content-between">
      <h3 class="h5 mb-1"><i class="fab fa-github"></i> swagger-editor-validate</h3>
    </div>
    <blockquote class="blockquote fs-6 mb-1">
      This GitHub Actions validates OpenAPI (OAS) definition file using Swagger Editor.
    </blockquote>
    <script type="application/ld+json">
      {
        "@context": "https://schema.org",
        "@type": "SoftwareSourceCode",
        "author": { "@id": "https://vladimirgorej.com" },
        "name": "swagger-editor-validate",
        "abstract": "This GitHub Actions validates OpenAPI (OAS) definition file using Swagger Editor.",
        "codeRepository": "https://github.com/char0n/swagger-editor-validate"
      }
    </script>
  </a>
</div>

### Usage

There are two significant use-cases of using this GitHub Action, but both of them use the fact that your definition
file lives in your GitHub repository. You just need to provide a file system path to it.

#### Public use-case

If you have access to the internet and don't mind that this GitHub Action sends your OpenAPI definition to 
[https://editor.swagger.io/](https://editor.swagger.io/) for validation, then use this workflow.

{% highlight yaml linenos %}
on: [push]

jobs:
  test_swagger_editor_validator_remote:
    runs-on: ubuntu-latest
    name: Swagger Editor Validator Remote

    steps:
      - uses: actions/checkout@v2
      - name: Validate OpenAPI definition
        uses: char0n/swagger-editor-validate@v1
        with:
          definition-file: examples/openapi-2-0.yaml
{% endhighlight %}

#### Private use-case

If you want to maintain complete privacy and your OpenAPI definition may contain sensitive information 
use the following workflow. The workflow uses a swagger-editor docker image that runs as a service of the workflow.

{% highlight yaml linenos %}
on: [push]


jobs:
  test_swagger_editor_validator_service:
    runs-on: ubuntu-latest
    name: Swagger Editor Validator Service

    # Service containers to run with `runner-job`
    services:
      # Label used to access the service container
      swagger-editor:
        # Docker Hub image
        image: swaggerapi/swagger-editor
        ports:
          # Maps port 8080 on service container to the host 80
          - 80:8080


    steps:
      - uses: actions/checkout@v2
      - name: Validate OpenAPI definition
        uses: char0n/swagger-editor-validate@v1
        with:
          swagger-editor-url: http://localhost/
          definition-file: examples/openapi-2-0.yaml
{% endhighlight %}

This is what you'll see if your OpenAPI definition validates **successfully**.

<figure class="figure">
  <img src="{{ '/assets/img/blog/swagger-editor-validate-success.png' | relative_url }}" width="1149" height="365" class="figure-img rounded mx-auto d-block img-fluid" alt="Successful validation" />
  <figcaption class="figure-caption text-center">Successful validation</figcaption>
</figure>

This is what you'll see if your OpenAPI definition contains **errors**.

<figure class="figure">
  <img src="{{ '/assets/img/blog/swagger-editor-validate-errors.png' | relative_url }}" width="1218" height="430" class="figure-img rounded mx-auto d-block img-fluid" alt="Failed validation" />
  <figcaption class="figure-caption text-center">Failed validation</figcaption>
</figure>

### Future

During implementations, I identified other features that this action would benefit from.
To not extend the implementation scope of the first version, I decided not to implement these features but instead document them and implement them later.
These features include:

- [Providing OpenAPI definition as a string](https://github.com/char0n/swagger-editor-validate/issues/1)
- [Providing OpenAPI definition as URL](https://github.com/char0n/swagger-editor-validate/issues/2)
- [Expose a screenshot on failure](https://github.com/char0n/swagger-editor-validate/issues/4)
- [The mechanism for ignoring errors](https://github.com/char0n/swagger-editor-validate/issues/3)

I think the most interesting one is - **Mechanism for ignoring errors**. Commonly, OpenAPI definitions 
contain errors when validated in [https://editor.swagger.io/](https://editor.swagger.io/). There are cases when there is nothing authors can do 
with these errors, which are known and ignored. This will open a door for such a use-case,
which I'm sure is quite common.

Hope that [swagger-editor-validate](https://github.com/char0n/swagger-editor-validate) will become a handy tool for anybody using Swagger Editor to edit OpenAPI definitions.
I do know it will become convenient for my old team in Ubiquiti Inc ;] Please leave any feedback in the comments. I'd highly appreciate it!