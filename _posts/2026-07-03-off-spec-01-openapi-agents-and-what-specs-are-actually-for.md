---
layout: post
title: "'Off Spec' № 01: On OpenAPI, agents, and what specs are actually for"
description: "In the first edition of APIwiz's 'Off Spec' series, Shane O'Connor put fifteen questions to Vladimír Gorej about how APIs actually get built, maintained, and consumed at scale — from what killed API Blueprint, to why OpenAPI was written for machines but not for agents, to what specs are really for once machines start reading them instead of humans."
date: 2026-07-03 10:00:00 +0200
canonical_url: https://www.linkedin.com/pulse/off-spec-01-vladim%C3%ADr-gorej-char0n-openapi-agents-what-shane-o-connor-yw5ge/
image:
  path: assets/img/blog/off-spec-01.webp
  width: 1280
  height: 720
  caption: APIwiz
---

<style>
  /* page-specific styles for this interview */
  .interview-toc {
    background: #f8f9fa;
    border: 1px solid #dee2e6;
    border-radius: .5rem;
    padding: 1.25rem 1.5rem;
    margin: 1.5rem 0 2.5rem;
  }
  .interview-toc__title {
    font-size: .85rem;
    text-transform: uppercase;
    letter-spacing: .05em;
    color: #6c757d;
    margin-bottom: .75rem;
  }
  .interview-toc ol {
    margin: 0;
    padding-left: 1.25rem;
    columns: 2;
    column-gap: 2rem;
  }
  @media (max-width: 575px) {
    .interview-toc ol { columns: 1; }
  }
  .interview-toc li { margin-bottom: .35rem; break-inside: avoid; }
  .interview-toc a { text-decoration: none; }
  .interview-toc a:hover { text-decoration: underline; }

  .interview-q {
    margin-top: 2.75rem;
    margin-bottom: 1rem;
    scroll-margin-top: 1rem;
  }
  .interview-q__num {
    color: #fd7e14;
    font-weight: 700;
    margin-right: .15rem;
  }

  .interview-pullquote {
    margin: 1.75rem 0;
    padding: .25rem 0 .25rem 1.25rem;
    border-left: 4px solid #fd7e14;
    font-size: 1.2rem;
    font-style: italic;
    line-height: 1.5;
    color: #495057;
  }

  .interview-lightning { margin: 1rem 0 0; }
  .interview-lightning dt { font-weight: 600; color: #212529; }
  .interview-lightning dd { margin-bottom: 1rem; }
</style>

<p class="lead">
  <em>'Off Spec'</em>, brought to you by APIwiz, is a conversation series about how APIs actually get built,
  maintained, governed, and consumed at scale — what the specs don't capture. For № 01,
  <a href="https://www.linkedin.com/in/shane-o-connor-apiwiz/" target="_blank" rel="noopener noreferrer">Shane O'Connor</a>
  put fifteen questions to me. This is a re-post of the
  <a href="https://www.linkedin.com/pulse/off-spec-01-vladim%C3%ADr-gorej-char0n-openapi-agents-what-shane-o-connor-yw5ge/" target="_blank" rel="noopener noreferrer">original interview</a>.
</p>

I spent five years as a core maintainer of Swagger tools. Before that I was principal engineer at Apiary, home of
API Blueprint. I'm a maintainer of the AsyncAPI specification. And today I co-found SpecLynx and work as an AI
engineer at Jentic, where the two ends of my career — specifications and agents — are finally colliding.

<nav class="interview-toc" markdown="0">
  <div class="interview-toc__title">Jump to a question</div>
  <ol>
    <li><a href="#q1">What no spec will ever capture</a></li>
    <li><a href="#q2">What actually killed API Blueprint</a></li>
    <li><a href="#q3">The decision almost nobody noticed</a></li>
    <li><a href="#q4">Was OpenAPI 3.1 worth the pain?</a></li>
    <li><a href="#q5">API wisdom that's just wrong</a></li>
    <li><a href="#q6">What teams miss about specs</a></li>
    <li><a href="#q7">One feature I'd delete</a></li>
    <li><a href="#q8">Why event-driven feels second-class</a></li>
    <li><a href="#q9">Was OpenAPI written for agents?</a></li>
    <li><a href="#q10">Will hand-crafted specs get bypassed?</a></li>
    <li><a href="#q11">MCP: reinventing OpenAPI?</a></li>
    <li><a href="#q12">What I built that the world didn't want</a></li>
    <li><a href="#q13">The unglamorous plumbing that matters</a></li>
    <li><a href="#q14">Who actually pays for open source</a></li>
    <li><a href="#q15">Lightning round</a></li>
  </ol>
</nav>

<h2 id="q1" class="interview-q h4"><span class="interview-q__num">1.</span> The series is called Off Spec. So let's start there: what's something about how APIs actually get built and consumed that no specification will ever capture?</h2>

No specification fully captures the original author's intent.

A spec can describe paths, schemas, status codes, and examples, but it rarely explains why the API was designed
that way: the trade-offs, constraints, assumptions, and consumer needs behind it.

That intent matters. Without it, people can follow the contract and still miss the design.

<p class="interview-pullquote">The real API is not just the spec. It is the spec plus the documentation, SDKs, examples, error messages, and production behavior people come to rely on.</p>

<h2 id="q2" class="interview-q h4"><span class="interview-q__num">2.</span> You were principal engineer at Apiary, home of API Blueprint, a spec that genuinely competed with Swagger. What actually killed Blueprint, and what did OpenAPI get right that it didn't?</h2>

Blueprint was not killed by one thing. It was a mix of tooling economics, industry convergence, and probably
corporate gravity after Apiary.

API Blueprint had a beautiful authoring experience. It was readable, elegant, and very human-friendly. But
OpenAPI was more explicit and easier for a broad tooling ecosystem to build around: generators, validators,
gateways, documentation tools, and enterprise platforms.

Once the industry started converging on OpenAPI, even Apiary had to support it. At that point Blueprint was no
longer just competing on specification quality; it was competing against a network effect.

So I would not say Blueprint lost because it was bad. It lost because OpenAPI became the safer ecosystem bet, and
after the Apiary/Oracle chapter there was not enough independent momentum left to reverse that.

<h2 id="q3" class="interview-q h4"><span class="interview-q__num">3.</span> Five years maintaining Swagger tools that millions of developers depend on. What's the most consequential decision you made that almost nobody noticed?</h2>

Probably Swagger ApiDOM.

We built Swagger ApiDOM as a semantic parser for API specifications, a unified way to parse, process, transform,
and reason about specs across the Swagger ecosystem. The important part is that we integrated it in a completely
backward-compatible way.

For most users, the only visible effect was that OpenAPI 3.1 support arrived. But underneath, it changed the
foundation: Swagger tooling no longer had to treat every spec version and format as a separate world. It also
opened the door to supporting AsyncAPI, Arazzo, and Overlays through the same underlying model.

<p class="interview-pullquote">Good infrastructure becomes invisible when it works well.</p>

<h2 id="q4" class="interview-q h4"><span class="interview-q__num">4.</span> OpenAPI 3.1 finally aligned with JSON Schema 2020-12 after years of divergence, and you built the renderer for it. Was that alignment worth the pain, or did it fix a problem most users never actually had?</h2>

Yes, it was worth the pain.

Most real-world specifications were still OpenAPI 3.0.x, but there was clear interest in OpenAPI 3.1 support. I do
not think most users cared about JSON Schema 2020-12 alignment as a standards debate. They cared that newer
specifications worked, validation was more consistent, and tooling could support OpenAPI 3.1 properly.

For tooling authors, the old divergence between OpenAPI Schema Objects and JSON Schema created years of special
cases across renderers, validators, parsers, and generators. OpenAPI 3.1 did not remove all complexity, but it did
move the ecosystem closer to a shared foundation.

So yes, it fixed a real problem, just not always at the layer where most users described it.

<h2 id="q5" class="interview-q h4"><span class="interview-q__num">5.</span> What's a piece of received wisdom in the API world that you think is just wrong? "Design-first always," "REST is dead," "spec-driven everything." Get as spicy as you want.</h2>

The received wisdom I think is wrong is that specs are mainly for documentation.

That view is too small. In today's world, an API specification should be treated more like infrastructure-as-code:
a living description of the system that is versioned, tested, checked against reality, and kept up to date.

A spec should participate in the lifecycle of the API: design, review, validation, testing, governance, SDK
generation, agent consumption, and runtime checks.

<p class="interview-pullquote">The future is not "write a spec to produce nice documentation." The future is "use the spec to prove the API, the tooling, and the consumers still agree."</p>

<h2 id="q6" class="interview-q h4"><span class="interview-q__num">6.</span> Most teams treat the OpenAPI spec as a documentation output, something you generate at the end. What are they fundamentally missing about what a spec is for?</h2>

They are missing the feedback loop.

If the spec is generated only at the end, it can tell consumers what the API does, but it cannot help the team
catch problems while the API is still being shaped.

A good spec should be part of the pull request, the CI pipeline, and the release process. It should catch breaking
changes, validate examples, drive contract tests, generate SDKs, and give consumers something stable to review and
build against as the API evolves.

Documentation is one output of a spec. The bigger job is keeping design, implementation, tooling, and consumers
aligned.

<h2 id="q7" class="interview-q h4"><span class="interview-q__num">7.</span> If you could delete one feature from the OpenAPI spec, and one thing from your own past code, what goes, and who'd be mad about it?</h2>

From OpenAPI 3.1, I would probably delete the jsonSchemaDialect fixed field.

Most people think of OpenAPI 3.1 as "the version that aligned with JSON Schema 2020-12." But technically, through
jsonSchemaDialect, an OpenAPI document can declare a different JSON Schema dialect, including previous drafts or
even a custom dialect.

In theory, that flexibility makes sense. In practice, I have not seen many OpenAPI documents that actually use it.
The complexity it brings to tooling implementation feels disproportionate. Parsers, validators, renderers, and
generators suddenly have to care not just about OpenAPI 3.1, but about which JSON Schema dialect is active and what
behavior that implies.

From my own past code, I would happily delete the code that tried to provide full support for jsonSchemaDialect.
It was the kind of implementation work where the theoretical correctness was real, but the practical value for most
users was hard to justify.

Who would be mad? Probably the very small number of people who really need custom dialect support. But I suspect
most users would happily trade that flexibility for simpler, more predictable tooling.

<h2 id="q8" class="interview-q h4"><span class="interview-q__num">8.</span> You also maintain AsyncAPI. Why does event-driven still feel like a second-class citizen in the API tooling world, and is that finally changing?</h2>

Event-driven still feels second-class because it is harder to see.

With HTTP APIs, the interaction model is familiar: request, response, status code, payload. It maps naturally to
documentation, testing, gateways, SDKs, and mental models. Event-driven systems are different. The important
behavior is spread across producers, consumers, brokers, topics, retries, ordering, delivery guarantees, and time.

That makes the tooling problem harder. You are not just describing an endpoint; you are describing a system of
conversations that may happen asynchronously and indirectly.

I do think it is changing. AsyncAPI gives the industry a shared language for describing those systems, and as
workflows, agents, and event-driven architectures become more common, that language becomes more important.

<h2 id="q9" class="interview-q h4"><span class="interview-q__num">9.</span> You're now an AI engineer at Jentic. LLMs and agents have quietly become the biggest consumers of API specs. Was OpenAPI ever written for machines, or for humans pretending to be machines?</h2>

OpenAPI was written for machines, but not for agents.

It was designed so tooling could parse an API contract and generate documentation, clients, servers, validators,
tests, and governance checks. In that sense, it was absolutely machine-readable by design.

But agents are a different kind of consumer. They do not just need a valid contract or a generated client. They
need to decide whether an operation is relevant, what the side effects might be, how to combine calls, how to
recover from errors, and whether the API is safe to use in a given context.

That is where work like Jentic API Scorecard comes in: measuring whether an OpenAPI document is actually aligned
with how agents use specs.

<p class="interview-pullquote">The question is no longer just "is this valid OpenAPI?" It is "does this spec make the API understandable, usable, and safe for an autonomous consumer?"</p>

<h2 id="q10" class="interview-q h4"><span class="interview-q__num">10.</span> Honest take: are we about to watch a generation of lovingly hand-crafted API specs get bypassed, because an agent would rather read the code or just hit the endpoint and see what happens?</h2>

I think agents will bypass bad specs, not good ones.

If a specification is outdated, incomplete, or just a thin documentation artifact, then yes, an agent may get more
value from reading code, examples, traffic, or trying the endpoint directly. In that world, the lovingly
hand-crafted spec risks becoming decorative.

But a good OpenAPI document is still one of the best ways to give an agent a structured, intentional view of an
API. Code tells you what is implemented. Runtime behavior tells you what happens. A spec should tell you what is
intended, supported, stable, and safe to rely on.

So I do not think agents make specs irrelevant. I think they raise the bar.

<p class="interview-pullquote">Specs that are stale will be ignored faster. Specs that are accurate, rich, and aligned with reality become even more important.</p>

<h2 id="q11" class="interview-q h4"><span class="interview-q__num">11.</span> MCP and tool schemas are suddenly everywhere. Is that the API world reinventing OpenAPI badly, or solving something OpenAPI genuinely couldn't?</h2>

I do not think MCP and tool schemas are just OpenAPI reinvented badly. They are solving a different problem.

OpenAPI describes an API surface in detail: operations, parameters, schemas, responses, auth, errors, and
documentation. MCP is more focused on exposing capabilities to agents as callable tools.

That narrower focus is useful. An agent does not always need the full API description; it needs to know what action
it can take, what inputs are required, what result to expect, and whether the action is safe in context.

But I do think there is a risk of relearning old API lessons. Versioning, authentication, error handling,
pagination, compatibility, and governance all come back eventually.

So my take is: MCP is not a replacement for OpenAPI. It is an agent-facing runtime layer. The interesting future is
connecting them: using OpenAPI for rich API context, and MCP/tool schemas for the operational interface agents
actually call.

<h2 id="q12" class="interview-q h4"><span class="interview-q__num">12.</span> You founded SpecLynx and re-built ApiDOM. Serious bets. What's something you poured yourself into that the world just didn't want, and were they right not to want it?</h2>

SpecLynx is probably the clearest example.

We built a VS Code extension and editor around API specifications, and I still believe the problem was real. At the
time, a better human editing experience for specs made sense.

But the world has evolved. The more important surfaces now are MCP, LSP, CLIs, CI pipelines, and agent workflows,
places where specifications are not just written and viewed, but actively used, checked, transformed, and connected
to other systems.

Were people right not to want it? In that form, probably yes. Not because the idea was wrong, but because the
interface the world needs has changed. The future is less about people lovingly editing specs by hand, and more
about tools and agents continuously working with them.

<h2 id="q13" class="interview-q h4"><span class="interview-q__num">13.</span> What's the least glamorous part of API tooling, the unsexy plumbing, that actually matters more than all the design-philosophy debates people love to have on Twitter?</h2>

Proper parsing and reference resolution.

They are not glamorous, but they are where API tooling either becomes trustworthy or quietly falls apart. People
often think parsing means "read some JSON or YAML," and reference resolution means "follow a `$ref`." In reality,
both are much harder.

Good tooling has to understand the specification version, preserve semantic meaning, track source locations, handle
malformed input, and resolve references across files, URLs, anchors, bundles, dialects, and all the edge cases real
users create.

Everything else depends on that foundation: validation, rendering, editing, linting, code generation, governance,
testing, and now agent consumption.

<p class="interview-pullquote">If parsing or resolution is wrong, every layer above it becomes unreliable.</p>

<h2 id="q14" class="interview-q h4"><span class="interview-q__num">14.</span> Open source at Swagger's scale: what's the honest story about who actually pays for the tooling the entire API industry quietly runs on?</h2>

The honest story is that open source at this scale is often paid for indirectly, but it is paid for.

In the Swagger ecosystem, SmartBear played a central role. It owned the Swagger brand, built commercial products
around it, and funded engineering work on open source tooling that mattered both to those products and to the wider
API community.

That model has real benefits. The ecosystem gets dedicated engineering time, and the company has a strategic reason
to invest. But it also creates tension: the tooling is used far beyond one company, while the responsibility for
maintaining it is not shared equally.

So the honest answer is that some of the work is funded by companies with a reason to invest, and some of it is
still carried by maintainers because the ecosystem depends on it.

<h2 id="q15" class="interview-q h4"><span class="interview-q__num">15.</span> Lightning round. One word or one line each.</h2>

<dl class="interview-lightning row" markdown="0">
  <dt class="col-sm-4">Design-first or code-first?</dt>
  <dd class="col-sm-8">Contract-first.</dd>

  <dt class="col-sm-4">OpenAPI or GraphQL schema?</dt>
  <dd class="col-sm-8">OpenAPI for action-oriented APIs; GraphQL schema when clients need to query related data flexibly.</dd>

  <dt class="col-sm-4">Most overrated API tool?</dt>
  <dd class="col-sm-8">Static API portals.</dd>

  <dt class="col-sm-4">Most underrated?</dt>
  <dd class="col-sm-8">A good parser.</dd>

  <dt class="col-sm-4">Will REST still dominate in 2030?</dt>
  <dd class="col-sm-8">Yes, but agents will care more about clear operations, good descriptions, predictable errors, and safety than whether the API is perfectly RESTful.</dd>
</dl>

---

<p class="text-muted">
  This interview originally appeared on LinkedIn as part of APIwiz's <em>'Off Spec'</em> series. Read the
  <a href="https://www.linkedin.com/pulse/off-spec-01-vladim%C3%ADr-gorej-char0n-openapi-agents-what-shane-o-connor-yw5ge/" target="_blank" rel="noopener noreferrer">original article</a>.
</p>
