# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

**ComfyUI-CE-Nodes** — a collection of custom utility nodes for ComfyUI. All nodes live in `__init__.py`, use only Python stdlib, and are registered under category `Text/Parsing`.

Current nodes:
- **HTML Parse (sync)** (`HTMLParseSync`) — extracts article metadata from raw HTML
- **Song JSON Split** (`SongJSONSplit`) — splits song-metadata JSON into individual outputs

## Quick Test

No formal test suite exists. Manual verification from ComfyUI root:

```python
from custom_nodes.ComfyUI_CE_Nodes import HTMLParseSync, SongJSONSplit
html = '<html><head><title>X</title></head><body><article><h1>H</h1><p>Body</p></article></body></html>'
print(HTMLParseSync().run(html, "https://example.com/x")[0])
```

## Architecture

Everything is in `__init__.py`. Each node is a class with `INPUT_TYPES`, `RETURN_TYPES`, `RETURN_NAMES`, `FUNCTION`, `CATEGORY`, and a `run()` method. Shared helpers are file-local functions prefixed with `_`.

### HTMLParseSync pipeline:
1. **URL discovery** (`_discover_url`) — resolves canonical/og:url/base href
2. **Domain extraction** (`_extract_domain_site`) — strips www/suffix
3. **Block stripping** (`_strip_blocks`) — removes script/style/nav/footer/ads
4. **Section picking** (`_pick_section`) — finds main content via `<article>`, `<main>`, content-class divs
5. **Metadata extraction** (`_extract_title`, `_extract_author`) — heuristic regex cascade
6. **HTML-to-text** (`_html_to_text`) — converts to plain text
7. **Truncation** (`_cap_wordsafe` / `_cap_chars`) — word-safe or hard cap
8. **Prompt templating** — renders `{title}`, `{content}`, etc. placeholders

### SongJSONSplit pipeline:
1. Strip markdown code fences and surrounding text
2. Extract `{...}` JSON object
3. Parse and output each field as separate STRING

## Key Conventions

- **Stdlib only** — `re`, `json`, `datetime`, `html.unescape`, `urllib.parse`. No pip dependencies.
- **No exceptions from `run()`** — always fall back to `""` or `"unknown"`.
- **Regex heuristics preferred** over full HTML parsers.
- **Tunables as constants** — e.g. `DEFAULT_MAX_CHARS`, `_BLOCK_TAGS`, `_CANDIDATES`.
- **Pre-compile hot regexes** — `_CANDIDATES` list uses compiled patterns.
- **New fields must be added to**: `RETURN_TYPES`, `RETURN_NAMES`, JSON output, and README.
- **New nodes** — add class, register in `NODE_CLASS_MAPPINGS` and `NODE_DISPLAY_NAME_MAPPINGS`.

## Versioning

- `__version__` in `__init__.py` (currently `0.1.3`)
- Patch: refactor/docs/minor heuristic tune
- Minor: new optional input or extraction field
- Major: behavior changes altering existing outputs/defaults
- Keep README changelog in sync with version bumps
