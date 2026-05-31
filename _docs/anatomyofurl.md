---
title: "Anatomy of a URL (part 1)"
permalink: /anatomyofurl.html
module: 2
lesson: 3
slug: anatomyofurl
reading_time: 14
description: "Read Greenfield's URL left to right and design your first one. The five parts every API URL ships and what each one commits to."
previous_page:
  url: /typesofAPI.html
  title: "Types of APIs"
next_page:
  url: /anatomyofurltwo.html
  title: "Anatomy of a URL (part 2)"
---

{% comment %}block:1{% endcomment %}
## Friday's PR

Tuesday morning of story week 6. I open my laptop. Devon's PR is at the top of my review queue. He opened it Friday afternoon with one line: "advanced filter, take 4. no POST. yes query string. yes `branch_north` not `north`. yes I read RFC 3986."

The URL is in the OpenAPI diff: `https://api.greenfield.lib/v1/books?q=mystery&branch=branch_north&shelf=fiction&on_shelf=true`. It's 9:30. Devon is in standup. He won't be back for forty minutes. Today you start your own URL, the one Devon flagged the week we first talked about REST. You can't write a URL until you can read one. So we read his left to right.

{% comment %}block:2{% endcomment %}
## Today you will leave with

- How to read a URL left to right and name the five parts.
- What each part commits to and who reads it downstream.
- One URL you wrote yourself.

{% include ad-slot.html slot="lesson-mid-1" format="auto" %}

{% comment %}block:3{% endcomment %}
## Read it left to right

The URL in Devon's diff is `https://api.greenfield.lib/v1/books?q=mystery`. Left to right, it has five parts, and each one tells some piece of the stack what to do.

**{% include glossary-term.html term="scheme" %}.** `https`. This tells the client which protocol to speak. HTTPS means the connection is encrypted; the rest of the URL is unreadable to anything in the middle. Greenfield is HTTPS, like every public API a 2026 reader is calling.

**{% include glossary-term.html term="host" %}.** `api.greenfield.lib`. This is what DNS resolves to an IP address. The `api.` is a subdomain Greenfield uses to separate its API server from its public website at `greenfield.lib`. Two different machines, same registered domain. The subdomain is a Greenfield convention; some APIs use the bare domain and route by path, some put the API on a different domain entirely (`api.github.com`).

**Version.** `/v1`. Devon shipped Greenfield with a version segment from day one because he didn't want his first design to trap him. When Greenfield ships v2, `/v1` keeps working until Devon turns it off. Old clients keep working; new clients opt in. Some APIs hide the version in a header (`Accept: application/vnd.greenfield.v2+json`) instead. Devon doesn't, because the header form hides the version from URL readers (logs, bookmarks, the apprentice reviewing the PR).

**Base path.** Greenfield doesn't have one. The path starts at the version. Some APIs prefix every endpoint with `/api` or `/platform/v1` to keep the API routable behind a load balancer that also serves the marketing site. Greenfield uses a dedicated subdomain for that job, so no extra prefix.

**{% include glossary-term.html term="resource" %}.** `/books`. This is the noun. The handler that serves this URL is the books handler; its job is to know about books. The query string customizes what books come back, but the resource decides what the response is about. M2L1's argument was that the URL names a thing. This is the thing.

**{% include glossary-term.html term="query string" %}.** `?q=mystery`. The `?` opens the query string. After it, `name=value` pairs separated by `&`. `q` is the existing filter; `mystery` is its value. The query string parameterizes the handler. It doesn't change which handler runs. Same handler, different filter.

{% capture mermaid_src %}
sequenceDiagram
  participant Atlas
  participant DNS
  participant API as Greenfield API
  participant Handler as books handler
  Atlas->>DNS: resolve api.greenfield.lib
  DNS-->>Atlas: IP address
  Atlas->>API: GET /v1/books?q=mystery over HTTPS
  Note over API: /v1 routes to the v1 server tier
  API->>Handler: dispatch on /books
  Note over Handler: q=mystery filters the response
  Handler-->>Atlas: 200 with matching books
{% endcapture %}
{% include mermaid.html content=mermaid_src alt="A sequence diagram tracing one URL across the network stack. Atlas first asks DNS to resolve api.greenfield.lib and gets back an IP address. Atlas then sends GET /v1/books?q=mystery over HTTPS to Greenfield's API server. The /v1 prefix routes the request to the v1 server tier. The /books segment dispatches the request to the books handler. The query string q=mystery filters which books the handler returns. The handler responds 200 with the matching books back to Atlas. Each layer of the stack reads only the URL part it needs: DNS reads the host, the API server reads the version, the handler reads the resource and the query string." %}

Look at the diagram. Each layer reads only the part it needs.

{% comment %}block:4{% endcomment %}
## Now you write

Greenfield's advanced filter ships this week. The requirements are short:

- Filter by branch (a branch identifier like `branch_north`).
- Filter by shelf (Greenfield has fiction, biography, reference, periodicals).
- Filter by whether the book is on shelf right now or out on loan.

Write the URL.

Read the parts you just named, in order. Scheme: same. Host: same. Version: same. Resource: same. Three new filters means three new query parameters. Add them to the existing query string.

Here is what the parts give you:

```text
https://api.greenfield.lib/v1/books?q=mystery&branch=branch_north&shelf=fiction&on_shelf=true
```

Three new query parameters: `branch`, `shelf`, `on_shelf`. Each one a `name=value` pair, separated from its neighbor by `&`. Below is the URL with the new parts highlighted. Hover or tap each part to read what it commits to.

{% include interactive-svg.html slug="anatomyofurl" alt="The advanced filter URL rendered as visible text wrapped to two lines: line 1 https://api.greenfield.lib/v1/books, line 2 ?q=mystery&branch=branch_north&shelf=fiction&on_shelf=true. The five parts of the URL are individually hoverable regions: scheme (https), host (api.greenfield.lib), version (/v1), resource (/books), and query string (line 2 in its entirety). The three new query parameters added in this PR (branch, shelf, on_shelf) are rendered in copper while the existing q parameter and the rest of the URL are rendered in ink, marking what the apprentice wrote versus what was already there. Hovering or tapping each part reveals what it commits to and who reads it downstream. The version part's tooltip notes the alternative header-based versioning form some APIs use. The query string part's tooltip names the four name=value pairs and their separators." %}

A few things you might have done differently and what the parts say about each:

**Did you write `/v1/books/filter`?** That makes the URL name an action (filter), not the thing being filtered (books). The block 3 rule was: the resource is the noun. Filter belongs in the query string. `/v1/books` is still the resource; the query string says how to look at it.

**Did you write `/v1/books/branch_north/fiction/true`?** Path segments are for resources and their children, not for filter values. Two reasons it goes badly: the order of segments has to be memorized (which one is the branch, which one is the shelf?), and adding a fourth filter next year means breaking every URL anyone has saved. Query string parameters are named and unordered. Adding `?author=connelly` next year is invisible to every existing caller.

**Did you write `?filters=branch:branch_north,shelf:fiction,on_shelf=true`?** That packs the filters into a single query parameter as a string. Greenfield's server would now have to parse that string itself, defining its own grammar (what does the comma mean, what does the colon mean, how do you escape values that contain commas). The URL standard already defines how `&` and `=` work; reusing them is free.

Devon shipped the same URL you just wrote. The PR comment you leave him: nothing. The URL design is right. The doc page that explains what each filter does, what `on_shelf=false` means versus omitting `on_shelf` entirely, what happens when `branch` is misspelled: that is Module 4. Today you wrote the URL.

{% comment %}block:5{% endcomment %}
{% capture body %}
Pick a public API doc you've used recently. Open one of its endpoint URLs (most reference pages show the full URL right next to the method). Read it left to right. Name the five parts. Then answer: where did the API put its version? Does it have a base path? Does the resource name a thing or an action? Does it use a query string or a body for filters?

Write five lines per URL: one per part, then a verdict in one line on whether the URL was easy to read. Repeat for two more URLs from two different APIs. Notice when one API is harder to read than another. The harder ones usually have a reason. Sometimes the reason is good (the API has been around for fifteen years and changing the URL would break the world). Sometimes the reason is that nobody ever sat down and asked these questions.
{% endcapture %}
{% include callout.html variant="exercise" body=body %}

{% comment %}block:6{% endcomment %}
{% capture body %}
Plenty of APIs have URLs like `POST /v1/users/123/activate`, `POST /v1/orders/cancel`, `GET /v1/books/get?id=bk_184932`. Verbs in the path. That's RPC in a REST jacket, the same trap from last week.

Devon's rule: if the URL has a verb in it, the API is RPC underneath. The doc page owes the reader an operation list, not a resource list. The right move is not to pretend otherwise. Document what the API does, with the words the API uses.

The trap matters for URL design because once an API ships `/v1/users/activate`, every URL after it tends to follow. Patterns set hard. The first URL you write is the URL every URL after it will copy. Pick a noun.
{% endcapture %}
{% include callout.html variant="warning" body=body %}

{% comment %}block:7{% endcomment %}
## Words you can drop in standups now

{% include glossary-term.html term="scheme" %}. Use this when you mean "the `https://` part of the URL." More specific than "protocol", less ambiguous to engineers.

{% include glossary-term.html term="host" %}. Use this when you mean "the part that DNS resolves to an IP." Catches all of subdomain, registered domain, and TLD without needing to name each one.

{% include glossary-term.html term="resource" %}. Use this when you mean "the noun in the path that names what the response will be about." The opposite of an action.

{% include glossary-term.html term="query string" %}. Use this when you mean "the part after the `?` that customizes the response without changing which handler runs."

{% include ad-slot.html slot="lesson-mid-2" format="auto" %}

{% comment %}block:8{% endcomment %}
## AI co pilot tip

Tool: Cursor (the code editor with AI built in).

**The situation.** You're reviewing a PR that adds a new endpoint, or you're drafting one yourself. The diff has a URL pattern in it (the OpenAPI YAML, the routes file, a comment block). You want a second opinion on whether the URL design follows the parts.

**The prompt** (highlight the URL or the OpenAPI path entry, open the Cursor chat panel):

```text
Here is a URL pattern: <paste>. Read it left to right and name the five parts in left-to-right order. For each part, say one sentence on whether the choice fits common REST conventions. If the URL names an action instead of a resource, say so. If a filter is in the path that should be in the query string, say so. Do not hedge.
```

**What to expect back.** Five short lines, one per part, and a verdict in one sentence on the URL as a whole. If Cursor finds a verb in the path, the verdict should say so.

**What to watch for.** Cursor sometimes tries to be diplomatic about a URL that names an action ("this is a stylistic choice"). Push back: "is this URL RPC or REST?" The second pass usually picks one. Cursor is at its best when it has the surrounding code as context, so ask the question with the routes file or the OpenAPI YAML open in the editor.

{% comment %}block:9{% endcomment %}
## Before you go

Look at the URL bar of any web page you have open. Read it left to right. Name the parts that are there and the parts that aren't.

{% comment %}block:10{% endcomment %}
## Next week at Greenfield

<!-- TODO block:10 -->

{% include signoff.html %}
