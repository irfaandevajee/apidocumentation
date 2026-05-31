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

<!-- TODO block:1 -->

{% comment %}block:2{% endcomment %}
## Today you will leave with

<!-- TODO block:2 -->

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
<!-- TODO block:5 -->

{% comment %}block:6{% endcomment %}
<!-- TODO block:6 -->

{% comment %}block:7{% endcomment %}
## Words you can drop in standups now

<!-- TODO block:7 -->

{% include ad-slot.html slot="lesson-mid-2" format="auto" %}

{% comment %}block:8{% endcomment %}
## AI co pilot tip

<!-- TODO block:8 -->

{% comment %}block:9{% endcomment %}
## Before you go

<!-- TODO block:9 -->

{% comment %}block:10{% endcomment %}
## Next week at Greenfield

<!-- TODO block:10 -->

{% include signoff.html %}
