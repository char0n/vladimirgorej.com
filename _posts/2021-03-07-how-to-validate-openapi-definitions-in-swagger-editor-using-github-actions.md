---
layout: post
title: "How to validate OpenAPI definitions in Swagger Editor using GitHub Actions"
description: "How to validate OpenAPI definitions in Swagger Editor using GitHub Actions"
date: 2021-03-07 10:39:04 +0200
image:
  path: assets/img/blog/swagger-editor-validate.png
  width: 536
  height: 188
---

The OpenAPI Specification (OAS) defines a standard, language-agnostic interface to HTTP APIs which
allows both humans and computers to discover and understand the capabilities of the service without
access to source code, documentation, or through network traffic inspection. When properly defined, 
a consumer can understand and interact with the remote service with a minimal amount of implementation logic.

An OpenAPI definition can then be used by documentation generation tools to display the API, code generation
tools to generate servers and clients in various programming languages, testing tools, and many other use cases.

I took the above description of what OpenAPI is directly from [its specification](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.1.0.md).
If you're one of the people that writes OpenAPI definition by hand in [Swagger Editor](https://editor.swagger.io/) then this article
was written just for you.

Couple of years ago I worked with a company called [Ubiquiti Inc](https://www.ui.com/). We were producing great wireless hardware solutions
with embedded UIs. I was fortunate to have joined the company in times when cross hardware configuration system
called [UNMS](https://unms.com/) was being architected and developed. Backend part of the system consisted of huge [REST API](https://en.wikipedia.org/wiki/Representational_state_transfer) layer that 
Frontend layer was consuming. We used design-first approach of producing the API. The workflow of this 
[design-first approach](https://www.visual-paradigm.com/guide/development/code-first-vs-design-first) consisted of the following steps:

- create changes in [OpenApi 2.0](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/2.0.md) definition using Swagger Editor
- issuer: issue a [GitHub Pull Request](https://docs.github.com/en/github/collaborating-with-issues-and-pull-requests/about-pull-requests) for team to review how new API endpoints could look like
- reviewer: paste the OpenAPI definition from Pull Request to Swagger Editor to see if no errors were introduced
- merge Pull Request
- implement the actual REST API defined in OpenAPI definition

Today I'd immediately think about how to automate this workflow and make [Continuous Integration](https://en.wikipedia.org/wiki/Continuous_integration) do the work for us.
Unfortunately at that time my knowledge about CI/CD pipelines was limited and I wasn't aware of full benefits 
and values CI/CD provides.

Some time has passed since then, I did my homework and studied CI/CD topic thoroughly. I became huge fan 
of [GitHub Actions](https://github.com/features/actions) and automatization. Let's try to automate the workflow described above using GitHub Actions.

### Technical design

We need to produce a GitHub Action that uses Swagger Editor to validate OpenAPI definition provided
as a parameter to that action. Using a Swagger Editor in GitHub Action can be achieved in two ways:
running it in docker container using [swagger-api/swagger-editor image](https://hub.docker.com/r/swaggerapi/swagger-editor/) or using [https://editor.swagger.io/](https://editor.swagger.io/) directly.
Now that we have Swagger Editor running we use [puppeteer](https://pptr.dev/) to open a headless version of Chromium Browser,
paste OpenAPI definition into the Swagger Editor and wait for it to render errors
(if definition is valid no errors will be rendered). Read errors from the Swagger Editor using puppeteer 
and represent them via GitHub Actions API. GitHub Action will report failure on any error or success if
document is valid and contains no errors.

Why use Swagger Editor and not just JSON Schema validation? Well Swagger Editor does add it's own
additional layer of error recognition on top of JSON Schema validation. Unfortunately this additional
layer is tight coupled to Swagger Editor code and the easiest way to use it is to use it via running
the Swagger Editor in browser.

### Implementation

The implementation wasn't that straightforward as I expected, but still managed to implement this GitHub Action
against the technical design mentioned earlier with all it's requirements. I called it [swagger-editor-validate](https://github.com/char0n/swagger-editor-validate).

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

There are two major use-cases of how to use this GitHub Action, but both of them use fact that your definition
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
use the following workflow. The workflow uses swagger-editor docker image that runs as service of the workflow.

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

During implementations I identified other features that this action would benefit from. 
In order not the extend the implementation scope of first version I decided not to implement these features
but rather document them and implement them later. These features include:

- [Providing OpenAPI definition as string](https://github.com/char0n/swagger-editor-validate/issues/1)
- [Providing OpenAPI definition as URL](https://github.com/char0n/swagger-editor-validate/issues/2)
- [Expose a screenshot on failure](https://github.com/char0n/swagger-editor-validate/issues/4)
- [Mechanism for ignoring errors](https://github.com/char0n/swagger-editor-validate/issues/3)

I think the most interesting one is - **Mechanism for ignoring errors**. It is very common that OpenAPI definitions 
contains errors when validated in [https://editor.swagger.io/](https://editor.swagger.io/). There are cases when there is nothing authors can do 
with these errors and these errors are known and ignored. This will open a door for such a use-case,
which I'm sure is quite common.

Hope that [swagger-editor-validate](https://github.com/char0n/swagger-editor-validate) will become a handy tool for anybody using Swagger Editor to edit OpenAPI definitions.
I do know it will become handy for my old team in Ubiquiti Inc ;] Please leave any feedback in comments, 
I'd highly appreciate it!