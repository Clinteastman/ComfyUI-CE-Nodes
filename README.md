# ComfyUI-CE-Nodes

A collection of custom utility nodes for ComfyUI. Stdlib only — no extra pip packages required.

---

## Installation

Clone this repo into your ComfyUI `custom_nodes/` directory:

```text
ComfyUI/
  custom_nodes/
    ComfyUI-CE-Nodes/
      __init__.py
      README.md
```

Restart ComfyUI. Nodes appear under **Text / Parsing**.

---

## Nodes

### HTML Parse (sync)

**Class:** `HTMLParseSync`

Extract clean article metadata from raw HTML **without making network requests**. Uses regex heuristics — no external parser dependencies.

#### Inputs

| Name | Type | Required | Description |
|------|------|----------|-------------|
| html | STRING | Yes | Raw HTML content of the page. |
| url | STRING | No | Original page URL (used for domain/site heuristics; can be blank). |
| max_chars | INT | No | Max content characters (default 8000, word-safe by default). |
| word_safe | BOOLEAN | No | If true, truncation ends at a word boundary. |
| prompt_template | STRING | No | Optional prompt template with placeholders (see below). |

If `url` is blank the node tries to discover a canonical/og/base URL from the HTML.

#### Outputs (all STRING)

| # | Name | Description |
|---|------|-------------|
| 1 | json | JSON string with all extracted fields. |
| 2 | title | Extracted title (may be empty). |
| 3 | content | Cleaned article plaintext (capped). |
| 4 | author | Author/byline (may be empty). |
| 5 | domain | e.g. `example.com`. |
| 6 | site_name | Heuristic site identifier (root minus public suffix). |
| 7 | word_count | Word count (stringified integer). |
| 8 | prompt | Rendered prompt if a template was provided (else empty). |

#### JSON Structure

```json
{
  "url": "https://example.com/post/123",
  "domain": "example.com",
  "site_name": "example",
  "title": "An Example Article",
  "content": "First paragraph...\n\nSecond paragraph...",
  "author": "Jane Doe",
  "word_count": 173,
  "version": "0.1.3",
  "extracted_at": "2025-08-31T12:34:56.789012Z"
}
```

#### Prompt Templating

Provide a `prompt_template` and the node renders it with these placeholders:

| Placeholder | Meaning |
|-------------|---------|
| `{title}` | Extracted title |
| `{author}` | Extracted author/byline |
| `{content}` | Full (possibly truncated) cleaned content |
| `{content_snip}` | ~1000 char word-safe snippet of content |
| `{domain}` | Domain (e.g. example.com) |
| `{site_name}` | Site name heuristic |
| `{url}` | Discovered or provided URL |
| `{word_count}` | Integer word count |
| `{version}` | Node version |
| `{extracted_at}` | ISO UTC timestamp |

Unknown placeholders are left unchanged. Escape literal braces with `{{` / `}}`.

#### Heuristics & Notes

- Main content detected via `<article>`, `<main>`, or content-class `<div>`; falls back to `<body>`.
- Strips scripts, styles, comments, nav/footer/header/aside/iframes and common ad/promo/cookie blocks.
- `<li>` converted to `•` bullets. Tables flattened to spaced text.
- No JavaScript execution — dynamic/lazy-loaded content will be absent.

---

### Song JSON Split

**Class:** `SongJSONSplit`

Split a song-metadata JSON string into individual typed outputs. Designed to receive JSON from an LLM and break it into separate connections for downstream nodes.

#### Input cleaning

The node automatically handles messy LLM output:
- Strips markdown code fences (`` ```json ... ``` ``)
- Discards any text outside the `{...}` JSON object
- Invalid or empty JSON returns all empty strings (no errors)

#### Inputs

| Name | Type | Required | Description |
|------|------|----------|-------------|
| json_string | STRING | Yes | JSON string with song metadata fields. |

#### Outputs (all STRING)

| # | Name | Description |
|---|------|-------------|
| 1 | song_title | Song title. |
| 2 | tags | Style/production description tags. |
| 3 | lyrics | Full lyrics with newlines preserved. |
| 4 | bpm | Beats per minute. |
| 5 | duration | Duration in seconds. |
| 6 | key | Musical key (e.g. `E minor`). |
| 7 | language | Language code (e.g. `en`). |
| 8 | time_signature | Time signature numerator (e.g. `4`). |
| 9 | seed | Seed value (empty string if null). |

#### Example Input

```json
{
  "song_title": "Midnight Hustle",
  "tags": "Hard-hitting trap production with thunderous 808 bass...",
  "lyrics": "[Intro]\n(dark pad fades in)\nYeah, yeah...",
  "bpm": 145,
  "duration": 150,
  "key": "E minor",
  "language": "en",
  "time_signature": 4,
  "seed": null
}
```

Special thanks to [iChrist](https://www.reddit.com/user/iChrist/) for the basis of the workflow and prompt.

---

## Changelog

### 0.2.0

- Added **Song JSON Split** node.
- Renamed repository to ComfyUI-CE-Nodes.

### 0.1.3

- Added prompt templating system with `prompt_template` input and `prompt` output.
- Added `content_snip` placeholder.

### 0.1.2

- Added `word_count` as separate output.
- Added semantic `version` field in JSON.
- Added word-safe truncation option (`word_safe`).
- Improved canonical URL attribute-order handling.

### 0.1.1

- Fixed use of non-existent Python `str.trim()`.

### 0.1.0

- Initial release with HTML Parse (sync).

---

## License

Currently unspecified. Add a license file (e.g. MIT) if distributing.
