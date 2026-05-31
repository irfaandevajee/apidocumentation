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

<!-- TODO block:4 -->

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
