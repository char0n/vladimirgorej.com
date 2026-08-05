---
layout: post
title: "From packets to pixels: how the internet actually works"
description: "Everything that happens between typing a URL and seeing a page — packets, IP, TCP, ports, HTTP, HTTPS, domain names, hosting, DNS, and the browser rendering pipeline. One article, end to end, with hands-on exercises that need nothing but a terminal and a browser."
date: 2026-08-05 10:00:00 +0200
image:
  path: assets/img/blog/from-packets-to-pixels.webp
  width: 1280
  height: 520
  caption: The path from a keystroke to a rendered page
---

<style>
  /* page-specific styles for this article */
  .article-toc {
    background: #f8f9fa;
    border: 1px solid #dee2e6;
    border-radius: .5rem;
    padding: 1.25rem 1.5rem;
    margin: 1.5rem 0 2.5rem;
  }
  .article-toc__title {
    font-size: .85rem;
    text-transform: uppercase;
    letter-spacing: .05em;
    color: #5c636a; /* darker than $gray-600 to clear 4.5:1 on the #f8f9fa panel */
    margin-bottom: .75rem;
  }
  .article-toc ol {
    margin: 0;
    padding-left: 1.25rem;
    columns: 2;
    column-gap: 2rem;
  }
  @media (max-width: 575px) {
    .article-toc ol { columns: 1; }
  }
  .article-toc li { margin-bottom: .35rem; break-inside: avoid; }
  .article-toc a { text-decoration: none; color: #0a58ca; } /* $blue-600: 4.5:1 on the panel */
  .article-toc a:hover { text-decoration: underline; }

  details.article-exercise {
    border: 1px solid #dee2e6;
    border-radius: .5rem;
    background: #f8f9fa;
    padding: .6rem 1rem;
    margin: 1.5rem 0;
  }
  details.article-exercise > summary {
    cursor: pointer;
    font-weight: 600;
    color: #343a40;
    list-style: none;
  }
  details.article-exercise > summary::-webkit-details-marker { display: none; }
  details.article-exercise > summary::before {
    content: "\25B8";
    display: inline-block;
    margin-right: .5rem;
    color: #fd7e14;
    transition: transform .15s ease;
  }
  details.article-exercise[open] > summary::before { transform: rotate(90deg); }
  details.article-exercise[open] > summary { margin-bottom: 1rem; }
  details.article-exercise .highlight { margin-bottom: 0; }
  details.article-exercise > *:last-child { margin-bottom: .4rem; }

  h2.article-section { margin-top: 3rem; scroll-margin-top: 1rem; }
</style>

<p class="lead">
  I spent five years maintaining Swagger tools, I now maintain
  <a href="https://speclynx.com" target="_blank" rel="noopener noreferrer">SpecLynx</a>,
  <a href="https://usearazzo.com" target="_blank" rel="noopener noreferrer">UseArazzo</a> and
  <a href="https://github.com/swaggerexpert" target="_blank" rel="noopener noreferrer">SwaggerExpert</a>, and I
  spend my days building agents that talk to APIs. HTTP isn't a topic I visit occasionally; it's where I live.
  Which made it mildly uncomfortable to notice how much of my mental model of the internet I had never
  actually verified.
</p>

Most of what an experienced engineer knows about the layers beneath their own code is inherited rather than
learned: absorbed from documentation skimmed at 2 a.m., from a colleague's offhand explanation, from a bug
that got fixed without ever being fully understood. That model works. It's mostly right. But "mostly right"
stays invisible until the day a DNS change stubbornly refuses to take effect, or a retried request duplicates
an order — and you find out the gap had been sitting there the whole time.

So I went back and checked. Not the exotic corners: the basics. What a packet actually is. Why a bare domain
traditionally can't be a `CNAME`. What "DNS propagation" really means (nothing propagates anywhere). Where a
browser spends its time between receiving bytes and showing you pixels. Most of it confirmed what I already
believed. A few things corrected me.

This article is the write-up of that pass — the explanation I wish I'd been handed years ago, and the one I'd
now give to anyone who builds on the web without having looked underneath it. It walks the whole path once,
end to end: each section builds on the previous one, and each ends with hands-on exercises that need nothing
more than a terminal and a browser.

Run them. That part isn't decoration. Verifying something yourself is the difference between knowing it and
having heard it — a distinction worth defending now that a confident, plausible explanation is always one
prompt away. A language model will happily tell you how DNS works. `dig +trace` will *show* you.

<nav class="article-toc" markdown="0">
  <div class="article-toc__title">What's ahead</div>
  <ol>
    <li><a href="#a-network-of-networks">A network of networks: packets, IPs, and ports</a></li>
    <li><a href="#speaking-http">Speaking HTTP: requests, responses, and status codes</a></li>
    <li><a href="#domain-names">Domain names: leasing your corner of the namespace</a></li>
    <li><a href="#hosting">Hosting: where websites actually live</a></li>
    <li><a href="#dns">DNS: the lookup chain behind every click</a></li>
    <li><a href="#the-browser">The browser: from response bytes to rendered pixels</a></li>
  </ol>
</nav>

## A network of networks: packets, IPs, and ports
{: #a-network-of-networks .article-section }

The internet is not a single thing you connect to. It is a *network of networks*: your home network, your
ISP's network, university networks, data-center networks — all agreeing to pass traffic between each other
using the same set of rules (protocols). Nobody owns or runs the whole thing; the networks physically meet and
swap traffic at **Internet Exchange Points (IXPs)** — buildings full of routers where dozens of providers plug
into each other. Cooperation through shared protocols, not central control, is what holds it together.

One distinction worth fixing early: **the internet is not the web.** The internet is the plumbing — cables,
routers, addresses, and the protocols that move data between machines. The **web** (websites, browsers, HTML,
links) is just one application running on top of that plumbing, alongside email, video calls, online games,
and messaging apps. This article covers the plumbing first, then climbs up to the web.

When you open a website, two machines are involved:

- **Client** — the machine that asks for something (your laptop, your phone, a script).
- **Server** — the machine that answers (a computer somewhere running software that listens for requests).

"Client" and "server" describe *roles*, not hardware. The same machine can be a client in one exchange and a
server in another.

### IP addresses: how machines find each other

Every device directly reachable on the internet has an **IP address** — a numeric label, like a postal address
for computers.

- **IPv4**: four numbers 0–255, e.g. `142.250.185.78`. Only ~4.3 billion possible addresses — we've
  effectively run out.
- **IPv6**: much longer hex addresses, e.g. `2a00:1450:4001:82f::200e`. Designed to never run out.

Because IPv4 addresses are scarce, your home devices usually share one *public* IP address. Your router hands
out *private* addresses (like `192.168.1.x`) internally and translates between the two — a trick called
**NAT** (Network Address Translation).

### Packets: how data actually travels

Data is never sent as one continuous stream. It's chopped into small chunks called **packets** (typically
~1,500 bytes). Each packet carries the destination IP, the source IP, a sequence number, and a slice of the
actual data.

Packets travel independently. Two packets from the same message may take *different routes* through the
network and arrive out of order — the receiving machine reassembles them. This design makes the internet
resilient: if one route fails, packets simply flow around it. Between your machine and a server, packets hop
through a chain of **routers**, each forwarding the packet closer to its destination.

### Protocols: the shared rules

None of this works unless every machine speaks the same language. That's what protocols are — agreed-upon
formats and procedures, stacked in layers:

<div class="table-responsive">
  <table class="table">
    <thead>
      <tr><th scope="col">Layer</th><th scope="col">Protocol</th><th scope="col">What it does</th></tr>
    </thead>
    <tbody>
      <tr><td>Addressing &amp; routing</td><td><strong>IP</strong></td><td>Gets packets from one machine to another</td></tr>
      <tr><td>Transport</td><td><strong>TCP</strong></td><td>Reliable, ordered delivery; retransmits lost packets</td></tr>
      <tr><td>Transport</td><td><strong>UDP</strong></td><td>Fast, no guarantees; good for video calls, games</td></tr>
      <tr><td>Application</td><td><strong>HTTP, SMTP, DNS…</strong></td><td>Meaning of the data (web pages, email, name lookups)</td></tr>
    </tbody>
  </table>
</div>

**TCP** is worth understanding: before sending data, client and server perform a *handshake*
(SYN → SYN-ACK → ACK) to establish a connection. TCP then numbers every byte, acknowledges receipt, and
re-sends anything lost. The web runs mostly on TCP (though HTTP/3 now uses QUIC, built on UDP).

### Ports: many conversations, one machine

A server has one IP address but runs many services. **Ports** (numbers 0–65535) distinguish them — like
apartment numbers within one building:

<div class="table-responsive">
  <table class="table">
    <thead>
      <tr><th scope="col">Port</th><th scope="col">Service</th></tr>
    </thead>
    <tbody>
      <tr><td>80</td><td>HTTP (web, unencrypted)</td></tr>
      <tr><td>443</td><td>HTTPS (web, encrypted)</td></tr>
      <tr><td>22</td><td>SSH (remote shell)</td></tr>
      <tr><td>25 / 587</td><td>SMTP (email)</td></tr>
      <tr><td>53</td><td>DNS</td></tr>
    </tbody>
  </table>
</div>

So a full "address" for a conversation is really **IP + port**: `142.250.185.78:443` means "the service
listening on port 443 at that machine."

You already use this daily as a developer without leaving your desk. **`localhost`** is a reserved name
(resolving to `127.0.0.1`) that means "this very machine" — client and server both on your own computer. When
you run a dev server and open `http://localhost:3000`, your browser is the client, your project is the server,
and `3000` picks it out among everything else running locally. Dev tools grab high, unclaimed ports by
convention — `3000`, `5173`, `8000`, `8080` — because the low, well-known ones require admin rights and are
spoken for.

### The whole trip, previewed

When you visit a website, roughly this happens — and the rest of this article unpacks each step:

1. Your machine looks up the site's IP address ([DNS](#dns)).
2. It opens a TCP connection to that IP on port 443.
3. Client and server negotiate encryption (TLS).
4. Your browser sends an [HTTP request](#speaking-http).
5. The server — wherever the site is [hosted](#hosting) — answers, split into packets, routed hop by hop.
6. Your [browser](#the-browser) reassembles and renders it.

<details class="article-exercise" markdown="1">
<summary>🧪 Try it yourself</summary>

{% highlight bash %}
# What's my machine's local IP?
ip addr        # Linux
ipconfig       # Windows

# What IP does a domain resolve to? How long is the round trip?
ping example.com

# Watch your packets hop router by router
traceroute example.com     # macOS / Linux
tracert example.com        # Windows
{% endhighlight %}

Then find your *public* IP (search "what is my IP") and compare it to your local one. They differ — that's
NAT at work.

</details>

<details class="article-exercise" markdown="1">
<summary>✅ Check yourself</summary>

- Why can two packets from the same download arrive out of order?
- What problem does NAT solve, and what created that problem?
- Your browser connects to `93.184.216.34:443`. What do the two parts mean?
- Why does a video call prefer UDP while a file download prefers TCP?

</details>

## Speaking HTTP: requests, responses, and status codes
{: #speaking-http .article-section }

**HTTP (HyperText Transfer Protocol)** is the application-level protocol browsers and servers use to exchange
web content. It's a simple request–response conversation in (mostly) plain text: the client asks, the server
answers, done.

HTTP is **stateless** — each request stands alone. The server doesn't inherently remember you between
requests. Anything that *feels* stateful (being logged in, a shopping cart) is built on top, usually with
cookies or tokens sent along with every request.

### Anatomy of a request

{% highlight http %}
GET /articles/42 HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0
Accept: text/html
Cookie: session=abc123
{% endhighlight %}

Four parts: a **method** (`GET`), a **path** (`/articles/42`), **headers** (`Name: value` metadata lines), and
an optional **body** — data you're sending, e.g. form contents or JSON. `GET` requests carry no body;
`POST`/`PUT` usually do.

<div class="table-responsive">
  <table class="table">
    <thead>
      <tr><th scope="col">Method</th><th scope="col">Meaning</th><th scope="col">Safe?</th><th scope="col">Idempotent?</th></tr>
    </thead>
    <tbody>
      <tr><td><code>GET</code></td><td>Read a resource</td><td>Yes</td><td>Yes</td></tr>
      <tr><td><code>POST</code></td><td>Create / submit data</td><td>No</td><td>No</td></tr>
      <tr><td><code>PUT</code></td><td>Replace a resource</td><td>No</td><td>Yes</td></tr>
      <tr><td><code>PATCH</code></td><td>Partially update</td><td>No</td><td>Not guaranteed</td></tr>
      <tr><td><code>DELETE</code></td><td>Remove a resource</td><td>No</td><td>Yes</td></tr>
    </tbody>
  </table>
</div>

*Safe* = doesn't change anything on the server. *Idempotent* = doing it twice has the same effect as doing it
once — which is why retrying a `POST` risks a duplicate order, while retrying a `PUT` doesn't.

Keep in mind these meanings are conventions, not enforcement: the server runs whatever code it wants for any
method. The conventions still matter, because everything around you — caches, browsers, proxies, retry logic —
assumes you follow them.

### Paths and query parameters

The **path** is how the server decides *which* resource or piece of code handles the request. On a static
site, `/about` might map straight to an `about.html` file. In an API, paths typically name data and its
hierarchy:

{% highlight text %}
/users            → the collection of users
/users/42         → the user with id 42
/users/42/posts   → that user's posts
{% endhighlight %}

**Query parameters** — the `?key=value&key2=value2` tail of a URL — refine the request without changing which
resource it targets:

{% highlight text %}
/products?category=books&sort=price&page=2
{% endhighlight %}

Same resource (`/products`), but filtered, sorted, and paginated — the classic jobs of query parameters:
filtering, searching, sorting, pagination. The server reads them as ordinary key–value inputs; nothing about
them is magic.

### Anatomy of a response

{% highlight http %}
HTTP/1.1 200 OK
Content-Type: text/html; charset=utf-8
Content-Length: 5320
Cache-Control: max-age=3600
Set-Cookie: session=abc123; HttpOnly

<!DOCTYPE html>
<html>...
{% endhighlight %}

A **status line**, then **headers**, then the **body** — the actual content.

The first digit of the status code tells you the category. Memorize the shape, not the full list:
**2 good, 3 look elsewhere, 4 your fault, 5 my fault.**

<div class="table-responsive">
  <table class="table">
    <thead>
      <tr><th scope="col">Range</th><th scope="col">Meaning</th><th scope="col">Common examples</th></tr>
    </thead>
    <tbody>
      <tr><td>1xx</td><td>Informational</td><td><code>101 Switching Protocols</code></td></tr>
      <tr><td>2xx</td><td>Success</td><td><code>200 OK</code>, <code>201 Created</code>, <code>204 No Content</code></td></tr>
      <tr><td>3xx</td><td>Redirection</td><td><code>301 Moved Permanently</code>, <code>302 Found</code>, <code>304 Not Modified</code></td></tr>
      <tr><td>4xx</td><td>Client made a mistake</td><td><code>400 Bad Request</code>, <code>401 Unauthorized</code>, <code>403 Forbidden</code>, <code>404 Not Found</code>, <code>429 Too Many Requests</code></td></tr>
      <tr><td>5xx</td><td>Server failed</td><td><code>500 Internal Server Error</code>, <code>502 Bad Gateway</code>, <code>503 Service Unavailable</code></td></tr>
    </tbody>
  </table>
</div>

<details class="article-exercise" markdown="1">
<summary>📋 Headers worth knowing</summary>

<div class="table-responsive">
  <table class="table">
    <thead>
      <tr><th scope="col">Header</th><th scope="col">Direction</th><th scope="col">Purpose</th></tr>
    </thead>
    <tbody>
      <tr><td><code>Host</code></td><td>request</td><td>Which site on this server you want (one IP can host many sites)</td></tr>
      <tr><td><code>Content-Type</code></td><td>both</td><td>Format of the body (<code>text/html</code>, <code>application/json</code>…)</td></tr>
      <tr><td><code>Authorization</code></td><td>request</td><td>Credentials, e.g. <code>Bearer &lt;token&gt;</code></td></tr>
      <tr><td><code>Cookie</code> / <code>Set-Cookie</code></td><td>req / resp</td><td>Client-side state, sessions</td></tr>
      <tr><td><code>Cache-Control</code></td><td>both</td><td>Whether/how long the response may be cached</td></tr>
      <tr><td><code>Location</code></td><td>response</td><td>Where a redirect points</td></tr>
      <tr><td><code>Accept</code></td><td>request</td><td>Formats the client can handle</td></tr>
    </tbody>
  </table>
</div>

</details>

### Not just pages: HTTP is the API protocol

Everything above applies far beyond loading web pages. When one program talks to another over the web — a
mobile app syncing messages, JavaScript refreshing a feed, one backend calling another — it's almost always
the same HTTP machinery carrying **JSON** instead of HTML:

{% highlight text %}
GET /api/me            →    200 OK
                            Content-Type: application/json

                            { "id": 7, "name": "Sam", "role": "admin" }
{% endhighlight %}

Nothing new is happening: same methods, same status codes, same headers — only the body format and the
consumer change. The response isn't shown to a human; code reads it and updates the page or acts on it. This
is why learning HTTP once pays off twice: it's both how websites load and how virtually every web API works.

### HTTPS: HTTP + encryption

Plain HTTP is readable by anyone on the path — your Wi-Fi neighbors, your ISP, anyone in between. **HTTPS**
wraps HTTP inside **TLS** (Transport Layer Security), which adds three guarantees:

- **Encryption** — nobody in the middle can read the traffic.
- **Integrity** — nobody can modify it undetected.
- **Authentication** — a *certificate*, signed by a trusted Certificate Authority, proves you're talking to
  the real `example.com`, not an impostor.

The padlock in your browser means the TLS handshake succeeded and the certificate checks out. It does **not**
mean the site itself is trustworthy — phishing sites use HTTPS too. Certificates are free today
([Let's Encrypt](https://letsencrypt.org/)), so there is essentially no excuse for a site to be HTTP-only.

### HTTP versions in one paragraph

**HTTP/1.1** (1997) — one request at a time per connection; browsers open several connections in parallel to
compensate. **HTTP/2** (2015) — multiplexes many requests over one connection, binary framing, header
compression. **HTTP/3** (2022) — runs on QUIC/UDP instead of TCP, avoiding a stalled packet blocking
everything behind it. As a developer you rarely change your code between them — the semantics (methods,
statuses, headers) stay the same.

<details class="article-exercise" markdown="1">
<summary>🧪 Try it yourself</summary>

{% highlight bash %}
# See a raw response, headers included
curl -i https://example.com

# Headers only
curl -I https://example.com

# Follow redirects and watch them happen
curl -iL http://github.com

# Send JSON
curl -X POST https://httpbin.org/post \
  -H "Content-Type: application/json" \
  -d '{"hello": "world"}'
{% endhighlight %}

Then open DevTools → **Network**, reload any page, and click a request. Everything in this section is right
there: method, status, headers, body.

</details>

<details class="article-exercise" markdown="1">
<summary>✅ Check yourself</summary>

- Why does a retry of a `POST` risk creating a duplicate order, while retrying a `PUT` doesn't?
- What's the difference between `401` and `403`?
- What three guarantees does TLS add on top of HTTP?
- The server sends `Cache-Control: max-age=3600`. What happens if you reload the page after ten minutes?

</details>

## Domain names: leasing your corner of the namespace
{: #domain-names .article-section }

Machines find each other by IP address, but `142.250.185.78` is hostile to humans — and a site's IP can change
when it moves servers. **Domain names** are stable, memorable labels (`example.com`) that get translated to IP
addresses on demand (that translation is [DNS](#dns)). This section is about the *names themselves*: who owns
them, who manages them, and how the pieces fit together.

### Anatomy of a domain

Read a domain **right to left** — it's a hierarchy:

{% highlight text %}
 blog.example.com.
  │      │     │ └── root (implicit trailing dot)
  │      │     └──── top-level domain (TLD)
  │      └────────── second-level domain (the part you register)
  └───────────────── subdomain (you create these freely)
{% endhighlight %}

- **Root** — the invisible top of the hierarchy, managed globally.
- **TLD** — `.com`, `.org`, `.dev`, plus country codes like `.cz`, `.de`, `.uk`. Each TLD is operated by a
  **registry**.
- **Second-level domain** — `example` in `example.com`. This is the unit you actually register and pay for.
- **Subdomains** — `blog.`, `shop.`, `api.` — once you own `example.com`, you can create unlimited subdomains
  at no cost, just by adding DNS records.

`www` is nothing special — just a conventional subdomain. `www.example.com` and `example.com` are technically
two different names, which is why sites configure one to redirect to the other.

A **URL** contains more than the domain:

{% highlight text %}
https://blog.example.com:443/posts/hello?ref=home#comments
└─┬──┘  └───────┬───────┘└┬┘└────┬─────┘└───┬───┘└───┬───┘
scheme       domain      port   path      query   fragment
{% endhighlight %}

### The players

Three roles, often confused:

<div class="table-responsive">
  <table class="table">
    <thead>
      <tr><th scope="col">Role</th><th scope="col">Who they are</th><th scope="col">Example</th></tr>
    </thead>
    <tbody>
      <tr><td><strong>Registry</strong></td><td>Operates an entire TLD, keeps its master database</td><td>Verisign runs <code>.com</code>; CZ.NIC runs <code>.cz</code></td></tr>
      <tr><td><strong>Registrar</strong></td><td>Retail company that sells registrations to the public</td><td>Namecheap, Cloudflare, Porkbun</td></tr>
      <tr><td><strong>Registrant</strong></td><td>You — the person/company holding the domain</td><td>—</td></tr>
    </tbody>
  </table>
</div>

Above the registries sits **ICANN**, the nonprofit that coordinates the global naming system and accredits
registrars.

Important mental model: **you never buy a domain — you lease it.** Registration is for 1–10 years and must be
renewed. Miss the renewal and the domain eventually returns to the open market (after a grace/redemption
period), where domain squatters are happy to grab anything with traffic.

### What registration actually gets you

When you register `example.com` at a registrar:

1. The registrar records you as registrant in the `.com` registry database.
2. You provide contact details (**WHOIS** data — usually hidden behind free privacy protection nowadays).
3. You tell the registry which **nameservers** answer questions about your domain — the hook that connects
   your domain to [DNS](#dns). By default these are your registrar's nameservers, but you can point them
   anywhere (e.g. Cloudflare's).

That's all a domain is: an entry in a registry database plus a delegation saying "ask *these* nameservers
about this name."

<details class="article-exercise" markdown="1">
<summary>💡 Practical notes on choosing and managing domains</summary>

- **Pricing games:** first-year prices are often loss-leaders (`$0.99`!) with steep renewal prices. Always
  check the *renewal* cost.
- **TLD choice:** `.com` is still what people type on autopilot. Country TLDs are fine for local audiences.
  Some TLDs (`.dev`, `.app`) force HTTPS at the browser level — a nice property.
- **Transfers:** you can move a domain between registrars. It requires unlocking the domain and an
  authorization code — a mild hassle by design, to prevent theft.
- **Domain ≠ hosting:** registering a domain gives you a name, not a place to put a website — even if
  registrars love to bundle them.
- **Security:** protect the registrar account with 2FA. Whoever controls the account controls the
  nameservers — and therefore your email, website, everything.

</details>

<details class="article-exercise" markdown="1">
<summary>🧪 Try it yourself</summary>

1. Look up any domain's public record: `whois example.com` in a terminal (or a web WHOIS tool). Find the
   registrar, creation date, expiry date, and nameservers.
2. Price a domain you like at two different registrars — compare first-year vs. renewal pricing.
3. Find the nameservers of a big site: `dig NS github.com +short`.

</details>

<details class="article-exercise" markdown="1">
<summary>✅ Check yourself</summary>

- In `docs.api.example.org`, which part did someone pay for, and which parts are free?
- What's the difference between a registry and a registrar?
- Why is "buying a domain" technically the wrong phrase?
- You registered a domain but haven't set up hosting. What does a visitor see, and why?

</details>

## Hosting: where websites actually live
{: #hosting .article-section }

**Hosting** means keeping your site's files and code on a computer that is switched on, connected to the
internet, and reachable 24/7 — so that when someone's browser asks for your site, something answers.

You *could* host a site from your laptop. In practice you don't, because your laptop sleeps, your home IP
changes, your upload bandwidth is small, and one viral moment would melt it. So you rent capacity from
companies whose entire job is keeping servers online.

### Static vs. dynamic — the distinction that matters

**Static hosting:** your site is just **files** — HTML, CSS, JavaScript, images — prepared in advance. The
server's only job is to hand them out, unchanged, to whoever asks. Every visitor gets the same files. It's
extremely cheap (often free), extremely fast, and there's almost nothing to break or hack. Perfect for
portfolios, blogs, documentation, and any app whose logic runs entirely in the browser. Typical providers:
GitHub Pages, Netlify, Vercel, Cloudflare Pages.

Note: a static *site* can still feel dynamic — JavaScript in the browser can fetch data from APIs. "Static"
describes the hosting, not the user experience.

**Dynamic hosting:** your site is a **program** that runs on the server and builds responses on the fly —
reading databases, checking who's logged in, processing payments. Different visitors get different responses.
It needs a runtime (Node.js, Python, PHP, Go…) executing server-side. This is what anything with accounts,
user-generated content, or a database requires.

### Flavors of dynamic hosting

<div class="table-responsive">
  <table class="table">
    <thead>
      <tr><th scope="col">Model</th><th scope="col">What you manage</th><th scope="col">Examples</th></tr>
    </thead>
    <tbody>
      <tr><td><strong>Shared hosting</strong></td><td>Your files; the host runs everything</td><td>Classic cPanel hosts</td></tr>
      <tr><td><strong>VPS (virtual private server)</strong></td><td>The whole OS — you're the admin</td><td>DigitalOcean, Hetzner, Linode</td></tr>
      <tr><td><strong>PaaS (platform as a service)</strong></td><td>Just your code; platform handles servers</td><td>Heroku, Render, Railway, Fly.io</td></tr>
      <tr><td><strong>Serverless / functions</strong></td><td>Individual functions that run per-request</td><td>AWS Lambda, Cloudflare Workers</td></tr>
      <tr><td><strong>Cloud (IaaS)</strong></td><td>Everything, at any scale</td><td>AWS, Google Cloud, Azure</td></tr>
    </tbody>
  </table>
</div>

The trade-off runs one direction: more control ↔ more responsibility. A VPS gives you full freedom and full
ownership of every security patch; a PaaS takes both away.

### CDNs: hosting's speed layer

A **CDN (Content Delivery Network)** is a worldwide fleet of servers that keep **cached copies** of your
content close to users. A visitor in Tokyo gets your files from a Tokyo edge server instead of crossing the
planet to your origin machine.

What CDNs buy you: **speed** (physics is real; shorter distance = lower latency), **resilience** (traffic
spikes hit the edge cache, not your origin), and **protection** (most CDNs absorb DDoS attacks and terminate
TLS for you).

Static assets are ideal CDN material. Modern static hosts (Netlify, Vercel, Cloudflare Pages) *are*
effectively CDNs — your whole site lives at the edge. Common standalone CDNs: Cloudflare, Fastly, CloudFront.

### Deployment: getting code onto hosting

**Deployment** is the process of moving your site from your machine to the host:

1. **The old way:** drag files onto the server over FTP. Error-prone, no history, no undo.
2. **The current way — git-based:** push to a Git repository; the platform detects the push, builds the site,
   and publishes it automatically (**CI/CD**).

{% highlight text %}
git push → platform builds → tests run → deploy to edge → live in ~1 minute
{% endhighlight %}

Good platforms add **preview deployments** (every branch gets its own URL), **instant rollback** (bad release?
one click back), and **environment variables** (secrets like API keys configured on the platform, never
committed to the repo).

### Custom domains: connecting the name to the host

Hosting providers give you a working default address like `yourproject.netlify.app`. To serve the site at
**your own domain**:

1. Register the domain ([previous section](#domain-names)).
2. In the hosting dashboard, add the custom domain — the host now expects traffic for that name.
3. In your DNS settings, point the domain at the host — an `A` record to an IP, or a `CNAME` to the host's
   address (exact records in the [next section](#dns)).
4. The host provisions a free TLS certificate automatically, so HTTPS just works.

The domain, the DNS, and the hosting can live at three different companies — they cooperate through exactly
this mechanism.

<details class="article-exercise" markdown="1">
<summary>🧪 Try it yourself</summary>

1. Create a file `index.html` with anything in it, push it to a GitHub repository, and enable
   **GitHub Pages** in the repo settings. You now have a hosted website — total cost: zero.
2. Open DevTools → Network on a big site and look at response headers like `cf-cache-status`, `x-cache`,
   `server`, or `age` — evidence of a CDN answering instead of the origin.

</details>

<details class="article-exercise" markdown="1">
<summary>✅ Check yourself</summary>

- Your blog is pure HTML/CSS. Which hosting model fits, and roughly what should it cost?
- Why can't a login system run on purely static hosting?
- What problem does a CDN solve that faster servers cannot?
- What are the moving parts between `git push` and your change being live?

</details>

## DNS: the lookup chain behind every click
{: #dns .article-section }

**DNS (Domain Name System)** is the internet's phone book: a global, distributed database that translates
names (`example.com`) into IP addresses (`93.184.216.34`) and other information. Every website visit, email
delivery, and API call starts with a DNS lookup.

No single server holds all the answers. DNS is a **hierarchy of delegation**: the root knows who runs each
TLD, the TLD knows who runs each domain, and the domain's own nameservers hold the actual records.

### A lookup, step by step

You type `blog.example.com`. Here's the full journey (the "cold" path — caching usually shortcuts most of it):

1. **Browser & OS cache** — have we looked this up recently? If yes, done.
2. **Resolver** — your machine asks a **recursive resolver** (run by your ISP, or a public one like `1.1.1.1`
   or `8.8.8.8`). The resolver does the legwork:
3. **Root nameservers** → "Who handles `.com`?" → referral to the `.com` TLD servers.
4. **TLD nameservers** → "Who handles `example.com`?" → referral to that domain's **authoritative
   nameservers** (the ones set at the registrar).
5. **Authoritative nameservers** → "What's the address of `blog.example.com`?" → the actual answer:
   `93.184.216.34`.
6. The resolver returns the answer — and **caches it** for next time.

Total cost: a few milliseconds when cached, maybe 20–120 ms cold.

Two roles worth keeping straight: the **recursive resolver** is the *asker* that chases referrals on your
behalf; an **authoritative nameserver** is the *answerer* that holds the real records for a domain.

### DNS records

A domain's zone is a set of typed records. The ones you'll actually touch:

<div class="table-responsive">
  <table class="table">
    <thead>
      <tr><th scope="col">Record</th><th scope="col">Maps</th><th scope="col">Example use</th></tr>
    </thead>
    <tbody>
      <tr><td><strong>A</strong></td><td>name → IPv4 address</td><td><code>example.com → 93.184.216.34</code></td></tr>
      <tr><td><strong>AAAA</strong></td><td>name → IPv6 address</td><td>same, for IPv6</td></tr>
      <tr><td><strong>CNAME</strong></td><td>name → another name</td><td><code>www → example.com</code>, or <code>blog → myapp.netlify.app</code></td></tr>
      <tr><td><strong>MX</strong></td><td>domain → mail servers (with priority)</td><td>route email to Google Workspace / Fastmail</td></tr>
      <tr><td><strong>TXT</strong></td><td>name → arbitrary text</td><td>ownership proofs; email security (SPF, DKIM, DMARC)</td></tr>
      <tr><td><strong>NS</strong></td><td>domain → its authoritative nameservers</td><td>the delegation itself</td></tr>
    </tbody>
  </table>
</div>

Rules of thumb:

- **A/AAAA** when you have an IP; **CNAME** when you have a hostname. A CNAME says "same answers as that other
  name" — so it can't coexist with other records on the same name, which is why the bare domain traditionally
  needs an A record, not a CNAME.
- **MX** is why your website and your email can live at completely different companies under one domain.
- **TXT** is the duct tape of DNS — every service that says "add this record to verify ownership" is using TXT.

### Caching, TTL, and the myth of "propagation"

Every record carries a **TTL (time to live)** in seconds — how long any cache may keep the answer before
asking again.

{% highlight text %}
example.com.   3600   IN   A   93.184.216.34
                └── cache me for up to 1 hour
{% endhighlight %}

High TTL (hours/days): fewer lookups, faster for users — but changes take longer to be seen everywhere. Low
TTL (60–300 s): changes spread fast, at the cost of more lookup traffic. **Pro move:** before a planned
migration, lower the TTL to 300 a day in advance; make the change; raise it back.

When people say a DNS change is "propagating," nothing is actually being pushed anywhere. Your authoritative
server answers with the new value **immediately**. What you're waiting for is the world's caches to let their
old copies **expire** (per the old TTL). Different resolvers cached at different moments, so some users see
the new value while others still see the old one. This is normal and resolves itself within the old TTL.

### Connecting a domain to hosting, in records

The custom-domain workflow from the [hosting section](#hosting), expressed concretely:

<div class="table-responsive">
  <table class="table">
    <thead>
      <tr><th scope="col">Goal</th><th scope="col">Record you create</th></tr>
    </thead>
    <tbody>
      <tr><td>Bare domain → your host's IP</td><td><code>A  example.com → 76.76.21.21</code></td></tr>
      <tr><td><code>www</code> → same site</td><td><code>CNAME  www → example.com</code></td></tr>
      <tr><td>Subdomain → a platform</td><td><code>CNAME  blog → myapp.netlify.app</code></td></tr>
      <tr><td>Email at your domain</td><td><code>MX  example.com → mail provider</code></td></tr>
      <tr><td>Prove ownership to a service</td><td><code>TXT  example.com → "verification=…"</code></td></tr>
    </tbody>
  </table>
</div>

And remember the top of the chain: the **NS records set at your registrar** decide *whose* DNS answers at all.
Many people register at one company but point NS to Cloudflare and manage records there.

<details class="article-exercise" markdown="1">
<summary>🧪 Try it yourself</summary>

{% highlight bash %}
# Basic lookup
dig example.com +short

# Full detail — find the TTL in the ANSWER section
dig example.com

# Specific record types
dig example.com MX +short
dig example.com TXT +short
dig example.com NS +short

# Watch the whole delegation chain (root → TLD → authoritative)
dig example.com +trace

# Ask a specific resolver
dig @1.1.1.1 example.com +short
{% endhighlight %}

Run `dig` on the same domain twice and compare the TTLs — the second answer comes from cache, counting down.

</details>

<details class="article-exercise" markdown="1">
<summary>✅ Check yourself</summary>

- What's the difference between a recursive resolver and an authoritative nameserver?
- Why can't the bare `example.com` be a CNAME (in classic DNS)?
- You changed an A record with TTL 86400 and some users still see the old site five hours later. Is anything
  broken?
- Which record type controls where your email goes?

</details>

## The browser: from response bytes to rendered pixels
{: #the-browser .article-section }

A browser is a program that turns a URL into an interactive page. That involves four distinct jobs, in a
pipeline:

{% highlight text %}
fetch → parse & render → execute JavaScript → store data / stay interactive
{% endhighlight %}

Under the hood, each browser is built on an **engine**: Chrome and Edge use Blink (with the V8 JavaScript
engine), Firefox uses Gecko (SpiderMonkey), Safari uses WebKit (JavaScriptCore). Engines differ slightly,
which is why "works in Chrome" isn't a guarantee.

### Fetching

You press Enter. The browser resolves the domain via [DNS](#dns), opens a TCP + TLS connection, sends an
[HTTP request](#speaking-http), and receives an HTML response.

But one HTML file is never the whole page. As the browser reads the HTML, it discovers *more* things to
fetch — stylesheets, scripts, images, fonts, API calls — and fires off dozens of additional requests, largely
in parallel. A typical page today triggers 50–100 requests. The browser also decides what it can skip:
resources cached from a previous visit (per `Cache-Control`) are reused straight from disk.

### Rendering: the critical path

1. **Parse HTML → DOM.** The HTML text becomes the **DOM** (Document Object Model) — a live tree of elements.
   Broken HTML doesn't stop it; browsers repair as they go.
2. **Parse CSS → CSSOM.** All stylesheets become a second structure describing the rules.
3. **DOM + CSSOM → render tree.** Only visible elements, each with its final computed style.
4. **Layout (reflow).** Compute exact geometry — where every box sits and how big it is.
5. **Paint.** Fill in pixels: text, colors, borders, images.
6. **Composite.** Layers (e.g. things with animations or `position: fixed`) are assembled on the GPU into the
   final frame.

Two practical consequences:

- **CSS blocks rendering** — the browser won't paint until it knows the styles (otherwise you'd see ugly
  unstyled flashes).
- **Classic `<script>` tags block parsing** — the parser stops, runs the script, then continues. That's why
  scripts go at the end of `<body>` or carry `defer`/`async` attributes.

When JavaScript later changes the page, the browser redoes the necessary parts of this pipeline — which is why
heavy DOM manipulation can make pages feel sluggish.

### Running JavaScript

JavaScript is what makes pages *do* things. The browser's JS engine executes code that can read and modify the
DOM, react to **events** (clicks, typing, scrolling), fetch more data without reloading (`fetch()` — the basis
of every single-page app), and — with permission — draw, animate, play audio, or access camera and location.

One crucial constraint: JavaScript on a page runs on a **single main thread**, shared with rendering.
Long-running JS = frozen page. And for safety, all of it runs in a **sandbox**: page code cannot touch your
files or other tabs, and the **same-origin policy** stops a page from reading data belonging to another site.

### Storing site data

Browsers give sites several places to keep data on your machine:

<div class="table-responsive">
  <table class="table">
    <thead>
      <tr><th scope="col">Storage</th><th scope="col">Size</th><th scope="col">Lifetime</th><th scope="col">Typical use</th></tr>
    </thead>
    <tbody>
      <tr><td><strong>Cookies</strong></td><td>~4 KB each</td><td>set by expiry; sent to the server <strong>with every request</strong></td><td>sessions, "keep me logged in"</td></tr>
      <tr><td><strong>localStorage</strong></td><td>~5–10 MB</td><td>until cleared</td><td>preferences, e.g. dark mode</td></tr>
      <tr><td><strong>sessionStorage</strong></td><td>~5 MB</td><td>until the tab closes</td><td>temporary state</td></tr>
      <tr><td><strong>IndexedDB</strong></td><td>large</td><td>until cleared</td><td>offline apps, big structured data</td></tr>
      <tr><td><strong>Cache Storage</strong></td><td>large</td><td>managed by service workers</td><td>offline pages, PWAs</td></tr>
    </tbody>
  </table>
</div>

The key difference: **cookies travel to the server automatically; the rest stay in the browser** unless code
sends them. That's why login sessions use cookies — and also why cookies are the mechanism behind cross-site
tracking, and why browsers increasingly restrict third-party cookies.

### DevTools: X-ray vision

Every desktop browser ships professional-grade inspection tools — press **F12** or right-click → *Inspect*.
The tabs map exactly onto this article:

<div class="table-responsive">
  <table class="table">
    <thead>
      <tr><th scope="col">Tab</th><th scope="col">What it shows</th></tr>
    </thead>
    <tbody>
      <tr><td><strong>Elements</strong></td><td>The live DOM + applied CSS; edit both in place</td></tr>
      <tr><td><strong>Console</strong></td><td>JS errors and logs; run any JavaScript against the page</td></tr>
      <tr><td><strong>Network</strong></td><td>Every request: method, status, headers, timing, size</td></tr>
      <tr><td><strong>Sources</strong></td><td>The page's code; set breakpoints, step through JS</td></tr>
      <tr><td><strong>Application</strong></td><td>Cookies and all storage types, inspectable and editable</td></tr>
      <tr><td><strong>Performance / Lighthouse</strong></td><td>Where rendering time goes; automated audits</td></tr>
    </tbody>
  </table>
</div>

<details class="article-exercise" markdown="1">
<summary>🧪 Try it yourself</summary>

1. Open DevTools → **Network**, tick "Disable cache," reload a news site. Count the requests; sort by size;
   find the slowest one.
2. In **Elements**, double-click any headline on any website and rewrite it. (Only your local copy changes —
   refresh to restore.)
3. In **Console**, run `document.title = "I control this page"` and watch the tab.
4. In **Application**, inspect the cookies of a site you're logged into — find the session cookie, note its
   `HttpOnly` and `Secure` flags.
5. Run a **Lighthouse** audit on your favorite site — most of its suggestions will now make sense.

</details>

<details class="article-exercise" markdown="1">
<summary>✅ Check yourself</summary>

- Why does one page load involve dozens of HTTP requests?
- In the rendering pipeline, what's the difference between layout and paint?
- Why does a `while(true) {}` loop freeze the whole page, including scrolling?
- Your login persists across browser restarts. Which storage mechanism is involved, and why that one?

</details>

## The whole story, in six lines
{: .article-section }

1. Machines find each other by **IP** and exchange **packets** under shared **protocols**.
2. On top of that, **HTTP** is the request/response language of the web; **HTTPS** encrypts it.
3. **Domain names** give humans stable names, leased through registrars.
4. **Hosting** is where the site's files or code actually live and answer requests.
5. **DNS** connects the name to the host — records, caches, and TTLs in between.
6. The **browser** ties it all together: fetch, render, run, store, inspect.

Next time a page loads in 300 ms, you'll know exactly how much machinery just cooperated to make that happen —
and next time something breaks, you'll know which layer to interrogate first.
