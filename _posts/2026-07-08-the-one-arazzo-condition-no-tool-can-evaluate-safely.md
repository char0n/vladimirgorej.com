---
layout: post
title: "The one Arazzo condition no tool can evaluate safely"
description: "Arazzo simple conditions look like plain comparisons, but they combine runtime expressions with their own comparison language. Most cases can be parsed faithfully. One case cannot. This is how @swaggerexpert/arazzo-criterion handles the boundary without pretending ambiguity away."
date: 2026-07-08 10:00:00 +0200
image:
  path: assets/img/blog/arazzo-criterion.webp
  width: 993
  height: 256
  caption: Arazzo logo, © OpenAPI Initiative
  object-fit: contain
---

<p class="lead">
  I'm building an Arazzo runner: a program that executes API workflows step by step. That means every step
  needs an answer to a deceptively simple question: did this step succeed?
</p>

In the case I was implementing, that answer came from a [Criterion Object](https://spec.openapis.org/arazzo/v1.1.0.html#criterion-object)
with a `simple` condition, such as `$statusCode == 200`. It looks obvious. It reads like plain English. But
if Arazzo workflows are meant to be executable and portable, then every runner has to read that condition
the same way.

So I wrote a quick evaluator. It handled the easy cases, then quietly fell apart on corner case after
corner case I hadn't anticipated. That was enough to make me stop, put the runner down, and go into
research mode.

This is the story of what that turned up, and of how the detour became a standalone library,
[@swaggerexpert/arazzo-criterion](https://github.com/swaggerexpert/arazzo-criterion): a small evaluator
that reads each condition as faithfully as the spec permits, and is honest about the one case where no
tool can know for sure.

## What a condition looks like

[Arazzo](https://spec.openapis.org/arazzo/v1.1.0.html) is the OpenAPI Initiative's specification for
describing sequences of API calls: workflows. After each step, you check that it worked by evaluating its
success criteria. Each criterion is a [Criterion Object](https://spec.openapis.org/arazzo/v1.1.0.html#criterion-object),
and one common form is a `simple` condition:

{% highlight yaml %}
condition: $statusCode == 200
{% endhighlight %}

Read it out loud and it's obvious: *did the server return a 200 status code?* That readability is the
whole appeal. You can write conditions that sound like what you mean:

{% highlight yaml %}
condition: $response.body.status == 'available' && $response.body.pets[0].id > 0
{% endhighlight %}

*Is the status "available", and does the first pet have a real id?* A person can read that. My job was to
write software that reads it the same way a person does, and, just as importantly, the same way every
*other* runner does.

That portability requirement is the part that matters. Arazzo isn't prose documentation for humans; it's a
machine-readable workflow description. If one runner treats a condition as true and another treats the same
condition as false, the workflow has stopped being portable. At that point the problem is no longer just
parsing. It's interoperability.

## My first instinct: just run it

My first idea was the lazy one: a condition looks like code. `$statusCode == 200` is almost a line you
could paste into a program and let the language run. Fill in the `$...` parts (like `$statusCode`) with
their real values, hand the rest to the language's `eval`, and read back true or false. An afternoon's work.

I'm glad I didn't ship that. The obvious objection is safety: feeding arbitrary text into `eval` is the
classic way to run code you never meant to run.

So I reached for the safer version: parse the text with a proper JavaScript parser first (a well-tested
component that turns code into a structured tree without executing anything), then walk that tree myself
and decide what each piece means. No blind execution, no injection hole. This felt right for about a day.

Then the deeper problem surfaced, and it's the reason this whole project exists. A *JavaScript* parser
reads *JavaScript's* rules. A Python runner would reach for a Python parser, a Go one for Go's, each with
its own subtly different semantics. I'd have traded "whatever `eval` does" for "whatever this one
language's parser does", still shackled to a single language, with no guarantee that two conformant runners
reach the same answer. Arazzo even layers on comparison rules that no language reproduces out of the box,
but I'll come back to those.

That was the click: I didn't need JavaScript semantics. I needed *Arazzo* semantics. A definition that
sits above any single language, that I could write down once and that every implementation, in any
language, could follow to the exact same result. That's what pushed me toward describing the condition
language with its own precise, portable rulebook. And that's where the real problem revealed itself.

## Two languages, one line

Writing that rulebook down sounds simple. It isn't, and here is why: that one innocent line is secretly
**two different mini-languages mixed together**.

The first is for *pointing at data*: `$statusCode`, `$response.body`, `$request.query.limit`. Arazzo calls
these [runtime expressions](https://spec.openapis.org/arazzo/v1.1.0.html#runtime-expressions), and each
one just points at a value somewhere in the request or the response, saying *where to look*. They already
have a precise definition of their own. I'd formalized that definition earlier in a separate library,
[@swaggerexpert/arazzo-runtime-expression](https://github.com/swaggerexpert/arazzo-runtime-expression),
and building it out carefully even fed back into the standard: the work helped tighten the
runtime-expression grammar in Arazzo 1.1.0 ([PR #454](https://github.com/OAI/Arazzo-Specification/pull/454)).
So my criterion library doesn't reinvent any of that. It just reuses that parser.

The second is for *comparing* things: `==`, `!=`, `<`, `>`, the word-joiners `&&` and `||`, and so on.
This is the part that says *what to check* once you've looked.

In the common case I cared about, a condition is a runtime expression plus a comparison: **`⟨where to look⟩ ⟨what to check⟩`**.
Simple enough, until you notice that the two languages **share some of the same symbols**.

That's the whole problem in one sentence. Let me show you why it bites.

## Why the obvious reading breaks

Runtime expressions in Arazzo are allowed to be surprisingly loose. A query-parameter name, for instance,
can legally contain almost any character, including `=`, `<`, `>`, even spaces. On its own that's fine; a
runtime expression is usually the *only* thing on the line, so there's nothing after it to get confused with.

Now put a runtime expression next to a comparison and watch it go wrong:

{% highlight yaml %}
condition: $request.query.limit == 10
{% endhighlight %}

You and I read that instantly: *the query parameter `limit` equals 10*. But the runtime-expression reader,
following its own loose rules, sees no reason to stop at `limit`. Spaces are allowed in a name. So is
`=`. So it happily swallows the **entire line**, `== 10` included, treating the whole thing as if the
parameter were literally named `"limit == 10"`. The comparison never gets its turn, and the reading
collapses.

<figure class="figure d-block text-center my-4">
  <svg viewBox="0 0 760 300" role="img" aria-labelledby="splitTitle splitDesc" style="max-width:100%;height:auto;font-family:'SFMono-Regular',Consolas,'Liberation Mono',Menlo,monospace;">
    <title id="splitTitle">Two ways to read one condition</title>
    <desc id="splitDesc">The condition dollar request dot query dot limit equals equals 10. The intended reading splits it into the runtime expression (dollar request dot query dot limit) and the comparison (equals equals 10). A literal reader instead swallows the whole line as a single runtime-expression name, losing the comparison.</desc>
    <text x="380" y="30" text-anchor="middle" font-size="15" fill="#6c757d" style="font-family:system-ui,-apple-system,'Segoe UI',Roboto,sans-serif">What you mean</text>
    <rect x="150" y="48" width="290" height="46" rx="8" fill="#e7f1ff" stroke="#0d6efd" stroke-width="1.5"/>
    <text x="295" y="77" text-anchor="middle" font-size="18" fill="#084298">$request.query.limit</text>
    <line x1="452" y1="42" x2="452" y2="100" stroke="#198754" stroke-width="2.5" stroke-dasharray="4 3"/>
    <rect x="464" y="48" width="146" height="46" rx="8" fill="#fff3e6" stroke="#fd7e14" stroke-width="1.5"/>
    <text x="537" y="77" text-anchor="middle" font-size="18" fill="#a5510a">== 10</text>
    <text x="380" y="164" text-anchor="middle" font-size="15" fill="#6c757d" style="font-family:system-ui,-apple-system,'Segoe UI',Roboto,sans-serif">What a literal reader does</text>
    <rect x="150" y="182" width="460" height="46" rx="8" fill="#e7f1ff" stroke="#0d6efd" stroke-width="1.5"/>
    <text x="380" y="211" text-anchor="middle" font-size="18" fill="#084298">$request.query.limit == 10</text>
    <text x="380" y="258" text-anchor="middle" font-size="14" fill="#dc3545" style="font-family:system-ui,-apple-system,'Segoe UI',Roboto,sans-serif">✗ swallowed as a single name, the comparison is lost</text>
    <rect x="238" y="278" width="13" height="13" rx="3" fill="#e7f1ff" stroke="#0d6efd"/>
    <text x="257" y="289" font-size="13" fill="#495057" style="font-family:system-ui,-apple-system,'Segoe UI',Roboto,sans-serif">where to look</text>
    <rect x="408" y="278" width="13" height="13" rx="3" fill="#fff3e6" stroke="#fd7e14"/>
    <text x="427" y="289" font-size="13" fill="#495057" style="font-family:system-ui,-apple-system,'Segoe UI',Roboto,sans-serif">what to check</text>
  </svg>
  <figcaption class="figure-caption">The same line, read two ways. The whole problem is deciding where the runtime expression stops.</figcaption>
</figure>

It's a bit like reading the sentence *"Call Mr. Green house painters today"* and not knowing whether
someone is named "Mr. Green" or "Mr. Green house." Without a clear signal for where the name ends, a
literal-minded reader keeps going and gets it wrong.

## The part I could fix, and the part I couldn't

Some of these collisions have a clean answer. Take the dot, `.`, which shows up both inside runtime
expressions and as a way to reach into data. It sounds ambiguous, but it usually isn't, because the
runtime-expression rulebook itself knows where a runtime expression like `$response.body` naturally ends. So I can simply *ask it*, and let the
rulebook draw the line rather than guessing. That collision, I could solve.

The unsolvable one is the shared `=`. Consider a query parameter that genuinely has an `=` in its name:

{% highlight yaml %}
condition: $request.query.a=b == 1
{% endhighlight %}

Is that `=` **part of the parameter's name** (`a=b`), or the **start of the comparison** `==`? Both
readings are legal. There is no rule, none exists in the Arazzo spec, that says "this `=` is part of the
name" versus "this `=` is an operator." A human can't reliably tell either. Nobody can. The string is
simply ambiguous, and no amount of clever programming invents an answer that isn't there.

This is the honest heart of it: **the ambiguity isn't a flaw in my code. It's a property of what was
written.** The best any implementation can do is be predictable about it and say so out loud.

## Reading a condition the way it was meant

So how do you read a condition the way its author meant it, while staying upfront about the cases you
can't? Here's the approach I shipped.

Instead of teaching my comparison-reader every rule of the runtime-expression language (copying someone
else's rulebook and letting it drift out of date), I do something simpler. When I hit a runtime expression,
I grab the whole suspicious chunk and hand it to the one component that already knows every rule:
[@swaggerexpert/arazzo-runtime-expression](https://github.com/swaggerexpert/arazzo-runtime-expression),
the parser from earlier, used unmodified. I ask it a single question: *how much of this is a valid runtime
expression?*

The parser reads as far as it legitimately can and hands back the leftover. Whatever's left is the
"what to check" part. So for `$response.body.data[0].id`, it reports that `$response.body` is the
runtime expression, and I treat the rest, `.data[0].id`, as the criterion language's navigation into the
resolved value: *reach into the first data item's id*. The boundary is drawn by the component that actually
knows the rules, not by me guessing.

Two things fall out of this:

- **The upside:** I never fork or modify the runtime-expression rulebook. It stays the single source of
  truth, and my library just *asks* it. When those rules improve upstream, I get the improvement for free.
- **The catch:** my library first does a rough pass ("this looks like a runtime expression, roughly") and
  only then does the parser confirm it for real. Passing the rough pass isn't enough; something can
  *look* like a runtime expression and still be nonsense. Think of a bouncer glancing at you versus someone
  actually checking your ID. That two-look approach, it turns out, is exactly how big specifications embed
  one language inside another: match the rough shape first, verify it for real second.

## The one case that stays broken (on purpose)

In this `simple` grammar, because I stop the runtime-expression operand at the comparison symbols, that
unfenced operand can never *contain* those symbols. That's the flip side of making the common case work:

{% highlight javascript %}
import { test } from '@swaggerexpert/arazzo-criterion';

test('$statusCode == 200');        // true:  reads exactly as intended
test('$request.query.a=b == 1');   // false: the '=' in the name can't be told apart
test('$response.body#/a=b == 1');  // true:  this style has its own clear fences
{% endhighlight %}

That middle line is the unavoidable casualty from earlier, and it fails *cleanly*. It doesn't silently
guess wrong, which is the important part. The last line works because the `#/…` pointer form gives the
operand a clear internal structure of its own: the `=` sits *inside* the pointer, so it never touches the
comparison layer.

<div class="table-responsive">
  <table class="table">
    <thead>
      <tr><th scope="col">Condition</th><th scope="col">How it reads</th><th scope="col">Result</th></tr>
    </thead>
    <tbody>
      <tr><td><code>$statusCode == 200</code></td><td>compare the status code to 200</td><td>works</td></tr>
      <tr><td><code>$request.query.limit == 10</code></td><td>compare query param <code>limit</code> to 10</td><td>works</td></tr>
      <tr><td><code>$request.query.a=b == 1</code></td><td>ambiguous: is <code>=</code> in the name, or the operator?</td><td>refused: ambiguous</td></tr>
      <tr><td><code>$response.body#/a=b == 1</code></td><td>fenced pointer, the <code>=</code> lives inside it</td><td>works</td></tr>
    </tbody>
  </table>
</div>

I didn't bury this. It lives as a documented limitation,
[issue #3](https://github.com/swaggerexpert/arazzo-criterion/issues/3), with the reasoning spelled out.
If a future version of the Arazzo spec adds a way to "escape" these symbols, the limitation lifts, so I've
[asked it to](https://github.com/OAI/Arazzo-Specification/issues/518). Until then, being honest about it
*is* the feature.

## Comparisons play by Arazzo's rules

So far I've shown `test`, which only answers "is this a valid condition?". The point, though, is to
`evaluate` one and get an actual true or false. The library never fetches live data itself, so you hand it
a small `resolve` function that turns a runtime expression into its current value:

{% highlight javascript %}
import { evaluate } from '@swaggerexpert/arazzo-criterion';

const context = {
  $statusCode: 200,
  '$response.body': { status: 'Available', data: [{ id: 42 }] },
};
const resolve = (expression) => context[expression];

evaluate('$statusCode == 200', { resolve });                    // true
evaluate("$response.body.status == 'available'", { resolve });  // true, even though the data says "Available"
evaluate('$response.body.data[0].id > 10', { resolve });        // true
{% endhighlight %}

That second line hides the last decision worth mentioning, and it's the loose end from the `eval` idea I
promised to come back to. When you compare things, the spec has its own gentle rules:
text comparisons ignore capitalization (`'Available'` equals `'available'`), the number `200` and the text
`'200'` are treated as equal, and a check passes on anything truthy and fails on `false`, nothing, or a
missing value. Those aren't any programming language's native rules, which is the *second* reason `eval`
would have quietly given wrong answers, even in a world with only one language. I implemented them
faithfully instead.

The tempting mistake is to import a rule from a *similar* feature next door. People often assume an empty
list should count as "false", but that instinct comes from *other* condition types (the ones built on
JSONPath or XPath, where an empty match means failure). `simple` has no such rule. The library follows the
plain reading of the spec: a condition passes on any truthy value and fails only on `false`, `null`, or a
missing value. An empty list is none of those, so this evaluator treats it as truthy and passes it.

And that word "truthy" hides a real gap. The spec never defines whether an empty array is truthy, so the
answer falls to the host language, and languages disagree: `[]` is truthy in JavaScript but falsy in
Python (`bool([])` is `False`). This library runs on JavaScript, so `[]` passes; a Python runner could
fail the very same condition. It's the ambiguous `=` all over again, just quieter: where the spec stays
silent, two conformant runners can still diverge. I've since opened
[an Arazzo spec issue](https://github.com/OAI/Arazzo-Specification/issues/517) to pin these truthiness
rules down.

## Why this matters beyond Arazzo

Strip away the specifics and you're left with something you'll recognize once you've seen it:

- A web address sitting inside a sentence: where does the link end and the prose resume?
- A file name that's allowed to contain spaces, listed next to other file names.
- Any place where you paste one kind of expression into the middle of another.

The general truth is this: **whenever you embed one little language inside another and they share the
same symbols, someone has to decide where one ends and the other begins, and if those symbols overlap
with no way to escape it, some inputs are simply undecidable.** That's not a bug you failed to
fix. It's a fact about the input. Good engineering isn't pretending the fact away; it's choosing *where*
to make the tradeoff, making the common case feel effortless, and naming the rare case that
can't work.

Arazzo's [`simple` condition](https://spec.openapis.org/arazzo/v1.1.0.html#criterion-object) is just a crisp, real example: a very permissive runtime-expression language, a
comparison language on top, a shared set of symbols, and a spec that offers no escape. So I made the
everyday conditions read exactly as intended, and I wrote down the one they can't.

## For the curious: the grammar

Feel free to skip this part. But I promised a definition that lives above any single language, so here it
is. The Arazzo spec describes `simple` only in prose and never gives it a formal grammar, so this is an
original formalization: an [ABNF](https://www.rfc-editor.org/rfc/rfc5234) grammar, the same portable
notation the internet's specs are written in, that any implementation in any language can follow to the
same result.

<style>
  details.abnf-grammar {
    border: 1px solid #dee2e6;
    border-radius: .5rem;
    background: #f8f9fa;
    padding: .6rem 1rem;
    margin: 1.5rem 0 2rem;
  }
  details.abnf-grammar > summary {
    cursor: pointer;
    font-weight: 600;
    color: #343a40;
    list-style: none;
  }
  details.abnf-grammar > summary::-webkit-details-marker { display: none; }
  details.abnf-grammar > summary::before {
    content: "\25B8";
    display: inline-block;
    margin-right: .5rem;
    color: #fd7e14;
    transition: transform .15s ease;
  }
  details.abnf-grammar[open] > summary::before { transform: rotate(90deg); }
  details.abnf-grammar[open] > summary { margin-bottom: 1rem; }
  details.abnf-grammar .highlight { margin-bottom: 0; }
</style>

<details class="abnf-grammar" markdown="1">
<summary>Show the full ABNF grammar</summary>

{% highlight text %}
condition        = S logical-expr S

logical-expr     = logical-or-expr
logical-or-expr  = logical-and-expr *( S "||" S logical-and-expr )
logical-and-expr = basic-expr *( S "&&" S basic-expr )

basic-expr       = paren-expr / comparison-expr / test-expr
paren-expr       = [ logical-not-op S ] "(" S logical-expr S ")"
test-expr        = [ logical-not-op S ] comparable
comparison-expr  = comparable S comparison-op S comparable

logical-not-op   = "!"
comparison-op    = "==" / "!=" / "<=" / ">=" / "<" / ">"

comparable                 = literal / runtime-expression-operand
runtime-expression-operand = "$" 1*operand-char
operand-char               = %x22-25 / %x27 / %x2A-3B / %x3F-5A / %x5B-5D / %x5E-7A / %x7E / %x80-10FFFF

runtime-expression-navigation = 1*( member-access / index-access )
member-access                 = "." member-name
index-access                  = "[" index "]"
member-name                   = 1*( ALPHA / DIGIT / "_" / "-" )
index                         = 1*DIGIT

literal          = number / string / boolean / null
boolean          = "true" / "false"
null             = "null"

number           = ( int / "-0" ) [ frac ] [ exp ]
int              = "0" / ( [ "-" ] DIGIT1 *DIGIT )
frac             = "." 1*DIGIT
exp              = ( "e" / "E" ) [ "-" / "+" ] 1*DIGIT
DIGIT1           = %x31-39

string           = squote *( escaped-quote / string-char ) squote
escaped-quote    = squote squote
string-char      = %x20-26 / %x28-10FFFF
squote           = %x27

S                = *B
B                = %x20 / %x09 / %x0A / %x0D

ALPHA          = %x41-5A / %x61-7A
DIGIT          = %x30-39
{% endhighlight %}

</details>

Three things in there tie straight back to the story:

- **The bounded token.** `runtime-expression-operand = "$" 1*operand-char` is the "grab the whole `$…`
  chunk" step. Look closely at `operand-char`: it is deliberately every character *except* whitespace and
  the operator symbols `! & ( ) < = > { | }`. That one exclusion is what makes a runtime expression stop
  at the comparison, and it is the very same reason this unfenced runtime-expression operand can't
  *contain* those symbols. Both faces of the problem, one line of grammar.
- **Runtime expressions aren't defined here.** Notice the grammar never says what a valid runtime
  expression actually *is*. It only captures the `$…` token, then delegates the real check to
  arazzo-runtime-expression afterwards. The grammar matches the rough shape; the second pass confirms it.
- **The two-layer spine.** The `logical-*` and `comparison-expr` rules are adapted from
  [RFC 9535](https://www.rfc-editor.org/rfc/rfc9535) (the JSONPath standard). A logical layer composes
  booleans over a *flat*, non-recursive comparison layer, which is why a chained comparison like
  `$a < 200 < 300` isn't merely rejected, it's ungrammatical: the rules give you no way to even write it.

## Try it

{% highlight shell %}
$ npm install @swaggerexpert/arazzo-criterion
{% endhighlight %}

<div class="list-group mb-3">
  <a href="https://github.com/swaggerexpert/arazzo-criterion" class="list-group-item list-group-item-action" target="_blank" rel="noopener noreferrer">
    <div class="d-flex w-100 justify-content-between">
      <h2 class="h5 mb-1"><i class="fa-brands fa-github"></i> @swaggerexpert/arazzo-criterion</h2>
    </div>
    <blockquote class="blockquote fs-6 mb-1">
      Reads, validates, and evaluates the <em>simple</em> conditions of the Arazzo Criterion Object.
    </blockquote>
    <script type="application/ld+json">
      {
        "@context": "https://schema.org",
        "@type": "SoftwareSourceCode",
        "author": { "@id": "{{ site.url }}" },
        "name": "@swaggerexpert/arazzo-criterion",
        "abstract": "Reads, validates, and evaluates the simple conditions of the Arazzo Criterion Object.",
        "codeRepository": "https://github.com/swaggerexpert/arazzo-criterion"
      }
    </script>
  </a>
</div>

It's part of [SwaggerExpert](https://github.com/swaggerexpert), a family of libraries that formalize the
expression languages around OpenAPI and Arazzo:

* <a href="https://github.com/swaggerexpert/arazzo-runtime-expression" target="_blank" rel="noopener noreferrer">arazzo-runtime-expression</a> (the runtime-expression parser this library asks)
* <a href="https://github.com/swaggerexpert/openapi-runtime-expression" target="_blank" rel="noopener noreferrer">openapi-runtime-expression</a> (its OpenAPI sibling)
* <a href="https://github.com/swaggerexpert/jsonpath" target="_blank" rel="noopener noreferrer">jsonpath</a> and <a href="https://github.com/swaggerexpert/json-pointer" target="_blank" rel="noopener noreferrer">json-pointer</a> (the pointer syntaxes that give an escape hatch)

## Closing words

I wanted one thing: software that reads a condition the way its author meant it. For everyday conditions,
it does. The trick is that it never guesses. It asks the runtime-expression parser where each expression
ends, and trusts the answer. One case stays out of reach: a name that contains a symbol like `=`, which
the spec gives no way to tell apart from the operator. No tool can evaluate that one safely, so rather than
guess, mine refuses it and says why. That is the condition in the title.

And it doesn't end here. `simple` still has no formal grammar in the spec, and its evaluation rules don't
say how a bare value like an empty array should be treated. So I've taken both questions back to the Arazzo
specification: one issue to [define the evaluation semantics](https://github.com/OAI/Arazzo-Specification/issues/517),
and one to [define a normative grammar and settle the operand boundary](https://github.com/OAI/Arazzo-Specification/issues/518).
Some of it may land in a future version of the spec. That is already how the earlier runtime-expression
fixes reached Arazzo 1.1.0, and it is where a definition meant to be shared belongs.
