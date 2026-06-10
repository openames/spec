# AMES Technical Specification: Version 0.9 (RFC)

**06/10/2026**

> **Status:** v0.9 RFC. Stable enough for implementation and feedback. Breaking changes before 1.0 will be clearly marked.

---

## 1. Introduction

The Agent and Machine Exchange Standard (AMES) defines a format for agentic exchange documents: permission-aware companion representations of canonical web content, expressed as Agent Exchange (`.ax`) files. This version defines the Markdown serialization, `.ax.md`. Other serializations are reserved for future specification.

AMES is intended to make web content more legible to automated systems without reducing that content to undifferentiated extracted text. Conventional extraction methods often flatten structural hierarchy, omit visual and contextual relationships, separate content from attribution, and discard rights or usage signals that are necessary for faithful interpretation. AMES provides a standardized translation format that preserves content structure, publisher intent, attribution context, and machine-readable permissions for automated use.

AMES is a translation and signaling format. It expresses content structure, semantic context, attribution information, and publisher-defined usage permissions. It does not prescribe crawler architecture, require any specific retrieval system, dictate agent behavior, arbitrate content quality, or determine the legal effect that any permission statement may have under applicable law.

AMES does not replace or override `robots.txt`. `robots.txt` governs crawl access. AMES permissions address what an automated system is permitted to do with content once it has access to that content. The two mechanisms operate at different layers and are designed to coexist.

This specification defines the AMES document model, the `.ax.md` Markdown serialization, required and optional metadata fields, structural conventions, permission vocabulary, and reserved extension points.

---

## 2. The `.ax` Document Format

The `.ax` document has two primary components: an AMES manifest and a translated body. The manifest declares the document's identity, permissions, and content signals; the body is a structure-preserving translation of the canonical source content.

In the `.ax.md` serialization defined by this version, the manifest is a YAML block at the beginning of the file, using conventional YAML frontmatter delimiters, and the body is rendered as zoned Markdown. The manifest contains six defined blocks: Page Identity, Permissions, and Content, which are required, and Topic Signals, Visual Register, and Key Media, which are optional. Each block is defined in Section 3.

A publisher may also use the AMES permissions infrastructure for pages that do not provide a translated Markdown body. This is a valid AMES use case when the publisher wishes to express machine usage permissions or discovery signals without publishing a full translation of the page.

## 3. Manifest

The AMES manifest is organized into six defined blocks. Blocks 1 through 3 are required. Blocks 4 through 6 are optional. The ordering of blocks and any character limits are normative. The blocks and fields defined in this section are the complete set of valid AMES manifest fields for the `.ax.md` document for this version. Implementations must not introduce additional fields.

### 3.1 Block 1: Page Identity (Required)

`ames_version` (String): Always `"0.9"` for this version.

`canonical_url` (String): Absolute URL of the human-facing source page.

`publisher` (String): The organization or publication responsible for this page.

`title` (String): The page's title; must match the source page's HTML `<title>`.

`locale` (String, Optional): IETF BCP 47 language tag, such as `"en-US"`. Maps from the source page's HTML `<html lang="">`.

`date_published` (String, Optional): ISO 8601 publication date.

`date_modified` (String, Optional): ISO 8601 last-modified date.

### 3.2 Block 2: Permissions (Required)

A permissions declaration is the publisher's machine-readable statement of which uses of its content by automated systems are permitted or reserved. Publishers must state the permissions under which the page may be used by a consuming system. This may be done by using the AMES native permissions fields described below, or by using the designation `custom` and pointing to the governing permissions regime.

A page participating in AMES carries a permissions block with exactly one of two valid states:

1. AMES native permissions, consisting of four required fields: `index`, `ephemeral`, `store`, and `train`. AMES permissions govern, as defined in Section 3.2.1. All four positions must be declared; partial declarations are invalid.

2. `custom`. The publisher's own terms govern, and AMES makes no native permission grant. See Section 3.2.2.

#### 3.2.1 AMES Permissions

AMES expresses permissions natively through four independent positions: `index`, `ephemeral`, `store`, and `train`. These positions cover search indexing, ephemeral retrieval, persistent storage and synthesis, and model development.

```yaml
permissions:
  index: 1
  ephemeral: 1
  store: 0
  train: 0
```

A value of `1` permits the specified use; a value of `0` reserves the corresponding right. Each position is independent: an allowed position does not imply permission for any reserved or omitted position, and a reserved or omitted position reserves its right regardless of what any other position allows, subject to applicable law.

These permissions do not transfer ownership, waive copyright, waive moral rights where applicable, or authorize any use beyond the specific conduct described in the granted position.

AMES granularizes permissions by observable use and persistence rather than by the class of automated system or company performing the act. Systems that primarily serve to route a user to the source rather than retaining or synthesizing content fall under the index position regardless of the underlying mechanism used.

A permission state is part of a minimum valid deployment. A participating page declares either AMES native permissions or the `custom` declaration. A permissions block that is present but blank is invalid. Content for which no AMES declaration exists is governed by applicable law and the publisher's other mechanisms, including `robots.txt`, terms of service, copyright notices, licenses, and other applicable rights statements.



| Position | Field       | Allow                                                        | Deny                                                         |
| -------- | ----------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **1**    | `index`     | Search indexing permitted. Permits transient reproduction of content, including the extraction or generation of snippets or equivalent navigational preview text or images, for the sole purpose of routing users to the source page, including temporary caching integral to that process. | Search indexing rights reserved. Transient reproduction and temporary caching of this content for search indexing purposes is not permitted. |
| **2**    | `ephemeral` | Ephemeral retrieval permitted. Permits automated systems to retrieve and use content solely to fulfill an immediate, specific user request, provided the content is not retained beyond the lifecycle of that request. Within that single request, content may be synthesized or excerpted to serve as context for the user's immediate response, with meaningful citation of the source. Content may not be stored, indexed, reused, or retained beyond that request; this permission does not authorize the generation of reusable derived records or persistent substitute displays. This permission does not extend to systematic, scheduled, speculative, or preemptive retrieval; corpus-building; or text and data mining for model-development purposes. | Ephemeral retrieval rights reserved. Automated retrieval of this content to fulfill user requests, including agentic browsing, tool-mediated retrieval, or similar automated retrieval processes, is not permitted. |
| **3**    | `store`     | Persistent storage and synthesis permitted. Permits automated systems to ingest, persistently retain, and process content in persistent databases or stores across multiple, separate user requests, to produce summaries, synthesized outputs, or excerpts, with meaningful citation of the source. This permission governs retention and reuse of content beyond any single request; it is the persistence that distinguishes it from ephemeral retrieval. | Persistent storage and synthesis rights reserved. The ingestion, retention, or automated processing of this content to produce persistent summaries, synthesized outputs, or excerpts across multiple requests, beyond the sole purpose of routing users to the source page, is not permitted. |
| **4**    | `train`     | Model development permitted. Permits extraction and use of content for text and data mining for model-development purposes, including as training, fine-tuning, validation, benchmark, or evaluation data for model or system development, for machine-based systems that infer from input to generate outputs. This permission is not to be construed as a waiver of copyright and does not authorize reproduction, distribution, or publication of content beyond what the model-development process itself requires. | Model development rights reserved. Extraction or use of this content for text and data mining for model-development purposes, including as training, fine-tuning, validation, benchmark, or evaluation data for model or system development, for machine-based systems that infer from input to generate outputs, is not permitted. |

Note: Positions 2 and 3 both permit synthesis with meaningful citation; they are distinguished by retention across requests. `ephemeral` is synthesize-and-discard: content is used within a single request and not retained beyond its lifecycle. `store` is synthesize-and-retain: content is ingested and held in persistent stores for reuse across multiple, separate requests. The line is stated in both definitions so neither position absorbs the other, and it is drawn by persistence, not by a system's internal model or the format of its output.

AMES does not constitute legal advice, confer legal rights, or provide enforcement. Publishers remain responsible for understanding applicable law and enforcing their rights. Nothing in this specification waives or affects any copyright or other right held by a publisher; the permissions expressed through AMES declare which automated uses a publisher permits or reserves, and do not transfer ownership, modify underlying rights, or substitute for any remedy under applicable law.

AMES permissions apply to the content of the source page, whether declared in the HTML head alone or also represented in an `.ax.md` translation file. They do not grant usage permissions for separately embodied media assets, source code, executable code, scripts, stylesheets, embedded components, third-party services, stock media, embedded media, third-party media, or other separately licensed materials, unless those rights and permissions are independently held and granted outside AMES.

#### 3.2.2 Custom Permissions

Although all conforming AMES implementations must carry a `permissions` declaration, a publisher is not required to use the AMES-defined permissions fields. A publisher may instead point to its preferred permissions language or framework using the `custom` designation. A `custom` value carries its pointer inline, after the keyword:

```yaml
permissions: custom
```

```yaml
permissions: custom https://example.com/terms
```

The value is read as a single string split on the first whitespace: the first token is the keyword `custom`, and any remainder is an optional pointer to the governing terms, which must be an absolute URL. The pointer is the only permitted supplement; free text is not valid. AMES does not retrieve, interpret, validate, or standardize the resource the pointer identifies.

### 3.3 Block 3: Content (Required)

The Content block identifies the zone or zones that constitute the substantive content of the source page. This block distinguishes the primary body of the work from navigation, chrome, advertising, comments, recommendation modules, related-content regions, and other non-substantive page elements.

The Content block contains one required field, `content.zone`, and one optional field, `content.dynamic_dom`.

`content.zone` (String or Array of Strings): Declares the zone or zones that constitute the substantive content of the page.

String, such as `"article"`: All elements matching this zone name are selected from the source page and treated as the substantive content zone.

Array, such as `["hero", "practice-areas"]`: All elements matching each declared zone name are selected and treated as the substantive content zones. The order of zone names in the array does not control content order; selected elements are interpreted in source order.

The declared content zone or zones must correspond to the substantive content translated in the zoned Markdown body. The Content block identifies the source content region; it must not introduce, summarize, replace, expand, or editorially alter the source content.

`content.dynamic_dom` (Boolean, Optional): Defaults to `false`. Set to `true` when client-side scripts alter the structural container sequence of the declared content zone after the initial server response. This flag does not alter what the translated body represents; structural parity is evaluated per Section 4. The flag is an advisory signal that the manifested page may differ structurally from the static source.

#### 3.4 Block 4 : Topic Signals (Optional)

- `topic_signals.intended_audience` (String, max 160 chars): The intended reader or use case.
- `topic_signals.core_topics` (Array of Strings, max 8 items, max 40 chars each): Primary subject areas.

#### 3.5 Block 5: Visual Register (Optional)

- `visual_register.logo_url` (String): Absolute URL to primary logo.
- `visual_register.colors` (Array, max 3 hex values): Primary, secondary, and accent colors in order of prominence.
- `visual_register.typography` (String, max 150 chars): Type choices, weight, and typographic character.
- `visual_register.visual_direction` (String, max 300 chars): Imagery style and overall visual character.

#### 3.6 Block 6: Key Media (Optional)

Identifies media essential to a machine reader's understanding of the page, declared under the top-level `media` key as a list of items. Decorative imagery is omitted.

| Field            | Required | Description                                                  |
| ---------------- | -------- | ------------------------------------------------------------ |
| `id`             | Yes      | Unique identifier within this file. Referenced in the body via `ames-media:id`. |
| `url`            | Yes      | Absolute, publicly resolvable URL.                           |
| `type`           | Yes      | One of: `diagram`, `photo`, `chart`, `illustration`, `screenshot`, `video`, `audio`, `embed`. |
| `alt_text`       | Yes      | Human-facing accessibility description.                      |
| `transcript_url` | No       | Absolute URL to an existing transcript (for `video`/`audio`). |
| `ocr_text`       | No       | Extractable text visible within the image (image types only). |

Declared media is referenced in the body using `ames-media:id` syntax. A declared media item must not also be embedded or referenced by its ordinary source URL as standard Markdown image syntax or as a raw URL.

```
![Descriptive alt text](ames-media:exposure-diagram)
```



## 4. The Zoned Markdown Body

After the manifest, the body is rendered in Markdown with zone markers preserving the structural sequence and container boundaries of the source HTML. Translation is a structural DOM traversal, not a generic HTML-to-Markdown conversion. Implementations walk the source DOM and emit a zone marker at each structural container boundary. Content elements are rendered as Markdown within the surrounding zone.

**Zone markers** use CommonMark link-label syntax and are invisible when rendered:

```
[//]: # (ames:zone zone-name)
[//]: # (ames:zone zone-name | permission-tuple)
```

Zoned Markdown is flat. Zone markers identify structural boundaries in traversal order; they do not open nested scopes. A new marker terminates the preceding zone. Named zones do not nest. Nested semantic elements each generate their own marker in flat sequence: a `<main>` containing an `<article>` produces two sequential markers.

**Structural parity.** Every structural container element encountered during traversal produces a zone marker. The zone name is the `data-ames` attribute value if present; otherwise, it is the element tag name. Empty zones are valid. A structural container that contains no renderable Markdown content still emits a marker. Structural parity is evaluated against the static, server-delivered HTML source, prior to the execution of any client-side scripts; the translated body represents that source.

A zone name must consist only of lowercase letters, digits, and hyphens, matching `[a-z0-9-]+`. The `data-ames` attribute value must already conform. A derived tag name conforms by construction. Where a `data-ames` value does not conform, the consuming system normalizes it by lowercasing and replacing each run of disallowed characters with a single hyphen; this preserves the structural marker but does not guarantee the original label. Publishers carry the burden of supplying conformant `data-ames` values.

A page whose content is unbounded or non-deterministic, such as an infinite-scroll feed, falls outside the structural parity model and is not a suitable candidate for AMES translation. A page that relies on client-side scripting to manifest its content remains eligible; the publisher declares this with `content.dynamic_dom`.

Structural container elements: `<div>`, `<main>`, `<nav>`, `<header>`, `<footer>`, `<aside>`, `<article>`, `<section>`, `<form>`, `<search>`, `<figure>`, `<details>`, `<summary>`, `<dialog>`.

Content elements render as Markdown within the surrounding zone and do not produce markers: `<p>`, `<h1>`–`<h6>`, `<ul>`, `<ol>`, `<li>`, `<blockquote>`, `<pre>`, `<table>`.

The following excluded elements do not produce zone markers and do not render as Markdown: `<script>`, `<style>`, `<noscript>`, `<iframe>`, `<canvas>`, `<svg>`. An SVG that carries substantive informational content may be declared in Block 6 as Key Media.

**Zone-level overrides.** A zone whose permissions differ from the page default carries its tuple in two places: the zone marker in the `.ax.md` file, and a `data-ames-policy` attribute on the corresponding source element.

```html
<article data-ames="article" data-ames-policy="1/1/0/0">
```

Zone permissions do not inherit from ancestor containers in the source DOM; each marker carries its own tuple if it differs from the page default.

**Translation conventions.**

- **Absolute URLs.** All hyperlinks and image URLs are resolved to absolute form. Declared media uses `ames-media:id`; undeclared images render as standard Markdown image syntax.
- **Tables.** Tables are rendered as pipe-delimited Markdown when the source table contains a discernible header row. Implementations must not manufacture header labels that are not present in the source. A table without a discernible source header row may be rendered using the closest faithful Markdown representation available, but any generated structure must not introduce labels, relationships, or meanings absent from the source.

**Zone sequence examples.**

Semantic HTML5: `<nav><main><article></article></main><footer>` →

```
[//]: # (ames:zone nav)
[//]: # (ames:zone main)
[//]: # (ames:zone article)
[//]: # (ames:zone footer)
```

Non-semantic markup with `data-ames` attributes produces the same sequence by attribute value; enhanced markup (`data-ames="primary-nav"`, `data-ames="footer-nav"`) produces zone names reflecting functional role.



## 5. The Site-Level Declaration File and Cascade

### 5.1 Site-Level Declaration

An AMES deployment should include an `ames-ai.txt` file at `/.well-known/ames-ai.txt`. The file is recommended, not required. It is not required to be linked from an individual page; consuming systems discover it by convention at that fixed path.

Because the site-level declaration is discovered independently of any page-level context, it is self-identifying.

**Format.** The file uses flat `Key: Value` lines. This format avoids MIME-type and server-configuration friction on shared hosting and requires no YAML or Markdown parser. Lines beginning with `#` are comments. Comments carry human-readable notice and do not modify the keyed fields. The keyed fields are the machine-readable declaration. `AMES-Policy` is the operative site-level permission state, and its meaning is defined by the specification identified by `AMES-Spec`. Field keys use the `AMES-` prefix.

**Fields:**

- `AMES-Version` (Required): Always `0.9` for this version.
- `AMES-Publisher` (Required): The organization responsible for the domain.
- `AMES-Domain` (Required): The host governed by the declaration, such as `catchlightmag.com`. This field self-identifies the declaration when the file travels apart from its location.
- `AMES-Policy` (Required): Site-wide permission state, expressed as a four-position tuple or `custom`, per Section 3.
- `AMES-Positions` (Required in tuple state): The tuple legend; in this version, exactly `index/ephemeral/store/train`.
- `AMES-Values` (Required in tuple state): The value legend; `1=allow, 0=deny`.
- `AMES-Locale` (Optional): IETF BCP 47 primary language tag for the domain.
- `AMES-Spec` (Required): Absolute URL identifying the AMES specification version that defines the meaning of the declaration.

The fields defined in this section are the complete set of valid fields for the site-level declaration file. Implementations must not introduce additional site-level fields.

**Scope.** A site-level declaration governs only the host from which it is retrieved. A declaration retrieved from `https://catchlightmag.com/.well-known/ames-ai.txt` does not govern `https://blog.catchlightmag.com` or any other host, and a declaration on one host does not extend to its subdomains or to its parent registrable domain. A host requiring a site-level declaration serves its own file at its own `/.well-known/ames-ai.txt` path. A site-level declaration is valid only where its `AMES-Domain` value matches the host from which it was retrieved; a host matches where it is identical to the `AMES-Domain` value or differs from it only by a leading `www.` label.

**Example: `/.well-known/ames-ai.txt`:**

```text
# Notice
# This file provides a site-level AMES declaration for automated systems. Page-level AMES declarations control where present. Where no valid page-level AMES declaration is present, the AMES-Policy in this file states the publisher's site-level default.
#
# The meaning of the AMES-Policy is defined by the AMES specification identified by AMES-Spec. Except as expressly permitted by a valid AMES declaration, all copyright, related rights, neighboring rights, database rights, text-and-data-mining rights, and other applicable rights are reserved, including for purposes of Article 4 of Directive (EU) 2019/790 where applicable.
#
# This declaration applies only to content for which the publisher holds the relevant rights and does not grant rights in third-party or separately licensed material.

AMES-Version: 0.9
AMES-Policy: 1/1/0/0
AMES-Positions: index/ephemeral/store/train
AMES-Values: 1=allow, 0=deny
AMES-Publisher: Catchlight
AMES-Domain: catchlightmag.com
AMES-Spec: https://openames.org/v0.9
```

In the `custom` state, `AMES-Policy` carries the keyword alone, optionally followed by an absolute URL to the governing terms; `AMES-Positions` and `AMES-Values` are omitted, as there is no tuple to legend.

```text
AMES-Version: 0.9
AMES-Policy: custom
AMES-Publisher: Catchlight
AMES-Domain: catchlightmag.com
AMES-Spec: https://openames.org/v0.9
```

A publisher governing its domain entirely through a custom permissions regime does not need this file. Where a custom regime already carries the publisher's terms, the `ames-ai.txt` declaration may be omitted in favor of that regime.

### 5.2 Permissions Cascade

**Cascade.** AMES permissions resolve from the most specific valid declaration to the broadest applicable valid declaration: zone overrides page, and page overrides site. Each broader layer applies only where a more-specific valid declaration is absent.

A malformed page-level permission declaration falls back to the site-level declaration where one exists, rather than directly to the absence of AMES permissions. Where all layers are absent or invalid, no AMES declaration exists for the content.

Zone-level tuples require a page-level AMES native permission declaration to exist. A zone-level override, whether expressed in an `.ax.md` zone marker or in a `data-ames-policy` attribute, is invalid when no page-level `ames-policy` declaration is present. A zone-level override is defined relative to a declared page-level baseline. A malformed zone-level tuple is disregarded, and the zone is governed by the page-level declaration.



## 6. Source Header and Discovery

### 6.1 The HTML Source Header

A source page carries its page-level AMES declaration in the HTML `<head>`.

- **Usage reference (required).** `rel="automated-usage"` links to the resource that governs automated use of the page and carries the AMES version in its `href`. The relation is generic and not AMES-specific; the `href` value ties the page to AMES. There is no separate version meta tag.
- **Permissions (required).** `name="ames-policy"` declares the page-level permission state. The `content` attribute carries either a permission tuple or `custom`; `data-positions` carries the tuple legend when a tuple is used.
- **Exchange Document Link (required when a translation exists).** `rel="alternate" type="text/markdown"` links the source page to its `.ax.md` translation file.

```html
<link rel="automated-usage" href="https://openames.org/v0.9" />

<meta name="ames-policy"
      content="1/1/0/0"
      data-positions="index/ephemeral/store/train" />

<link rel="alternate" type="text/markdown"
      href="https://example.com/page.ax.md" />
```

The `ames-policy` tag has two valid states. In the **tuple state**, `content` is the four-position permission tuple, and `data-positions` is present and, in this version, exactly `index/ephemeral/store/train`.

In the **`custom` state**, `content` is `custom`, `data-positions` is omitted because there is no tuple to legend, and `data-notice` is optional. When present, `data-notice` carries an absolute URL pointing to the publisher's governing terms.

```html
<meta name="ames-policy" content="custom" />
<meta name="ames-policy" content="custom"
      data-notice="https://example.com/terms" />
```

`data-notice` is used only in the `custom` state, where it may carry an optional absolute URL pointing to the publisher's terms. It is not used in tuple state.

The site-level declaration is not linked from the page. It is found by convention at the fixed `/.well-known/ames-ai.txt` path. See Section 5.1.

Zone-level overrides travel on the source element as `data-ames-policy`, for example `data-ames-policy="1/0/0/0"`.

A `permissions` block in an `.ax.md` file and the `ames-policy` declaration in the source `<head>` express the same page-level permission state and must agree. Maintaining agreement between the canonical source page and the translation file is the publisher's burden. A consuming system may rely on whichever declaration it retrieves.

### 6.2 Other Discovery Mechanisms

**Sitemap injection (recommended).** `.ax.md` URLs are listed alongside their HTML counterparts in `sitemap.xml`, each with its own `<lastmod>`.

**LLM hinting (informational).** Publishers may list AMES URLs in `llms.txt` with explicit Markdown links.



## 7. Reserved Extensions

The following extensions are reserved in AMES 0.9. They are named but not specified, and are dormant in this version.

- **`.ax.json`**: JSON serialization of the agentic exchange document. AMES 0.9 specifies the Markdown serialization only. The `.ax.json` suffix and `application/ames+json` MIME type are reserved.
- **`content_integrity`** (manifest block): Reserved for cryptographic content verification, including fingerprinting, normalization, and permission verification status.



## 8. Worked Example

A complete HTML page and its `.ax.md` translation. Catchlight is fictional. The page permits indexing and ephemeral retrieval and reserves store and train (`1/1/0/0`); the comments zone, carrying content the publisher does not own, reserves ephemeral as well (`1/0/0/0`).

**Source HTML page:**

```html
<!DOCTYPE html>
<html lang="en-US">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>The Golden Hour Myth: Why the Best Light Is the Light You Understand</title>

  <link rel="automated-usage" href="https://openames.org/v0.9" />
  <meta name="ames-policy" content="1/1/0/0" data-positions="index/ephemeral/store/train" />
  <link rel="alternate" type="text/markdown" href="https://catchlightmag.com/articles/golden-hour-myth.ax.md" />

  <link rel="canonical" href="https://catchlightmag.com/articles/golden-hour-myth/" />
  <meta property="og:title" content="The Golden Hour Myth" />

  <link rel="stylesheet" href="/assets/css/typography.css" />
  <link rel="stylesheet" href="/assets/css/article.css" />
</head>
<body>

  <header data-ames="header">
    <p>Catchlight | For photographers who think</p>
  </header>

  <nav data-ames="primary-nav">
    <ul>
      <li><a href="/features">Features</a></li>
      <li><a href="/technique">Technique</a></li>
    </ul>
  </nav>

  <article data-ames="article">
    <h1>The Golden Hour Myth: Why the Best Light Is the Light You Understand</h1>
    <p><strong>By Maya Osei</strong> | Published March 18, 2026 | Technique</p>
    <p>Every photography tutorial says the same thing: shoot at golden hour. But chasing golden hour is a crutch &mdash; the photographers who transcend it are the ones who learn to see in any light.</p>
    <img src="/assets/articles/golden-hour-myth/midday-comparison.jpg"
         alt="Side-by-side portraits: golden hour at left, managed midday shade at right, producing equally compelling contrast." />
  </article>

  <aside data-ames="related">
    <h2>Related Articles</h2>
    <ul>
      <li><a href="/technique/reading-shadows">Reading Shadows: A Field Guide</a></li>
    </ul>
  </aside>

  <section data-ames="comments" data-ames-policy="1/0/0/0">
    <h2>Comments</h2>
  </section>

  <footer data-ames="footer">
    <p>&copy; 2026 Catchlight Magazine</p>
  </footer>
</body>
</html>
```

**Translation file `golden-hour-myth.ax.md`:**

```markdown
---
ames_version: "0.9"
canonical_url: "https://catchlightmag.com/articles/golden-hour-myth/"
publisher: "Catchlight"
title: "The Golden Hour Myth: Why the Best Light Is the Light You Understand"
locale: "en-US"
date_published: "2026-03-18T09:00:00Z"

permissions:
  index: 1
  ephemeral: 1
  store: 0
  train: 0

content:
  zone: "article"

topic_signals:
  intended_audience: "Intermediate to advanced photographers seeking to deepen craft rather than acquire gear."
  core_topics: ["photography", "natural light", "composition", "photographic technique", "visual literacy"]

visual_register:
  logo_url: "https://catchlightmag.com/assets/logo.svg"
  colors: ["#1a1a1a", "#f5f2ee", "#c8773a"]
  typography: "Clean serif headlines with generous leading; neutral sans-serif body."
  visual_direction: "Documentary and fine art photography. High contrast, minimal post-processing."

media:
  - id: "midday-portrait-comparison"
    url: "https://catchlightmag.com/assets/articles/golden-hour-myth/midday-comparison.jpg"
    type: "photo"
    alt_text: "Side-by-side portraits: golden hour at left, managed midday shade at right, producing equally compelling contrast."
---

[//]: # (ames:zone header)
Catchlight | For photographers who think

[//]: # (ames:zone primary-nav)
- [Features](https://catchlightmag.com/features)
- [Technique](https://catchlightmag.com/technique)

[//]: # (ames:zone article)
# The Golden Hour Myth: Why the Best Light Is the Light You Understand

**By Maya Osei** | Published March 18, 2026 | Technique

Every photography tutorial says the same thing: shoot at golden hour. But chasing golden hour is a crutch — the photographers who transcend it are the ones who learn to see in any light.

![Side-by-side portraits: golden hour vs. managed midday](ames-media:midday-portrait-comparison)

[//]: # (ames:zone related)
## Related Articles
- [Reading Shadows: A Field Guide](https://catchlightmag.com/technique/reading-shadows)

[//]: # (ames:zone comments | 1/0/0/0)
## Comments

[//]: # (ames:zone footer)
© 2026 Catchlight Magazine
```

