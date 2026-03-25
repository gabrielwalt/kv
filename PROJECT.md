# Project reference — **ko** (Koffievoordeel → EDS Crosswalk)

Living inventory: environments, source pages, blocks, tokens, import tooling, and migration status.

## Environments

| Role | URL |
|------|-----|
| **GitHub** | [https://github.com/gabrielwalt/ko](https://github.com/gabrielwalt/ko) |
| **AEM Author** (content) | [https://author-p189820-e1977856.adobeaemcloud.com/content/ko/index.html](https://author-p189820-e1977856.adobeaemcloud.com/content/ko/index.html) |
| **EDS preview** | [https://main--ko--gabrielwalt.aem.page/](https://main--ko--gabrielwalt.aem.page/) |
| **EDS live** | `https://main--ko--gabrielwalt.aem.live/` |

Local preview: `aem up` → typically `http://localhost:3000` (see [AGENTS.md](./AGENTS.md)).

### `fstab.yaml` alignment

Mountpoints must use your org’s **Franklin delivery** URL (Code Sync), not the browser `/content/...` path. If preview or sync fails, confirm `mountpoints./.url` matches `https://<author-host>/bin/franklin.delivery/gabrielwalt/ko/<branch>` (or your team’s equivalent). The checked-in file may still reflect boilerplate; update it when wiring this repo to the Author instance above.

| Field | Value |
|--------|--------|
| **Source site** | [https://www.koffievoordeel.nl/](https://www.koffievoordeel.nl/) |
| **Import scope** | Six pages listed under [Migration status](#migration-status) |
| **Content path prefix** | As modeled under `/content/ko/` in AEM (confirm with your implementation) |
| **Local import output** | Optional `/content/` or `.plain.html` scratch — usually **not** Git-tracked if AEM is system of record |

---

## Source site — koffievoordeel.nl

Document DOM patterns here as you discover them (selectors, wrappers, grids, edge cases).

- **Platform**: _(to confirm — e.g. commerce / CMS stack, templates)_
- **Main content regions**: _(landmarks / main column selectors)_
- **Sections**: _(how visual bands map to HTML — wrappers, full-bleed vs contained)_
- **Images / CDN**: _(domains, query params, lazy-load attributes)_
- **Special cases**: _(iframes, third-party widgets, cookie banners to strip in import)_

### CDN / assets (fill in)

| Source | Domain / pattern | Notes |
|--------|------------------|--------|
| _(add rows)_ | | |

---

## Brand and design tokens

Boilerplate defaults live in `styles/styles.css` until replaced with Koffievoordeel brand values.

| Property | Current value | Notes |
|----------|-----------------|--------|
| Background | `white` (`--background-color`) | |
| Text | `#131313` | |
| Accent / links | `#3b63fb` | |
| Muted surfaces | `#f8f8f8` (`--light-color`) | Used by section styles below |
| Body font | Roboto (+ fallback) | `@font-face` fallbacks in `styles/styles.css` |
| Heading font | Roboto Condensed (+ fallback) | |

### Section styles

| Style (UE) | Class on section | CSS |
|--------------|------------------|-----|
| Highlight | `highlight` → `.section.highlight` | `styles/styles.css` (shared with `.section.light` background) |

The section model’s multiselect currently exposes **Highlight** (`highlight`). Styles also define `.section.light` if you add that option to the model later.

---

## Block reference

Blocks registered in the main section filter (`models/_section.json` → `section` filter): `text`, `image`, `button`, `title`, `hero`, `cards`, `columns`, `fragment`. Default content types come from `models/` spreads.

### `cards`

**Location**: `blocks/cards/` — `_cards.json`, `cards.js`, `cards.css`

### `columns`

**Location**: `blocks/columns/` — `_columns.json`, `columns.js`, `columns.css`

### `fragment`

**Location**: `blocks/fragment/` — `_fragment.json`, `fragment.js`, `fragment.css`

### `hero`

**Location**: `blocks/hero/` — `_hero.json`, `hero.css` (no `hero.js` in repo yet; decoration may be minimal or pending)

### `header` / `footer`

**Location**: `blocks/header/`, `blocks/footer/` — chrome blocks (`header.js` / `footer.js`, CSS). Not listed in the main section `components` filter; used as configured in your page / experience fragment setup.

### Default content (`models/`)

`_text.json`, `_title.json`, `_image.json`, `_button.json` — merged into component definition/models/filters via `npm run build:json`.

---

## Import infrastructure

**Status**: Not present yet — add under `tools/importer/` when you start bulk import.

When added:

- **Entry**: `tools/importer/import.js`
- **Bundle**: `tools/importer/import.bundle.js` (generated; do not hand-edit)
- **URL list**: e.g. `tools/importer/urls.txt` with the six source URLs below

Use selector-based parsers, document order, and transformer hooks (`beforeTransform` / `afterTransform`) in the tables below as you implement them.

### Parsers

| Parser | File | Source selector(s) |
|--------|------|---------------------|
| _(none yet)_ | | |

### Transformers

| Transformer | File | Hook | Purpose |
|-------------|------|------|---------|
| _(none yet)_ | | | |

### Bundle example

```bash
npx esbuild tools/importer/import.js --bundle --format=iife --global-name=CustomImportScript --outfile=tools/importer/import.bundle.js
```

---

## Migration status

### Pages (initial scope)

| Page | Source URL | AEM / EDS path | Status |
|------|------------|----------------|--------|
| Abonnement | [https://www.koffievoordeel.nl/abonnement](https://www.koffievoordeel.nl/abonnement) | | Not started |
| Illy Iperespresso recepten | [https://www.koffievoordeel.nl/drie-heerlijke-illy-iperespresso-recepten](https://www.koffievoordeel.nl/drie-heerlijke-illy-iperespresso-recepten) | | Not started |
| Abonnement wijzigen | [https://www.koffievoordeel.nl/abonnement-wijzigen](https://www.koffievoordeel.nl/abonnement-wijzigen) | | Not started |
| Terug in de tijd met Illy | [https://www.koffievoordeel.nl/terug-in-de-tijd-met-illy](https://www.koffievoordeel.nl/terug-in-de-tijd-met-illy) | | Not started |
| Gran Maestro Italiano | [https://www.koffievoordeel.nl/gran-maestro-italiano-experience](https://www.koffievoordeel.nl/gran-maestro-italiano-experience) | | Not started |
| Blog — wetenschap koffie | [https://www.koffievoordeel.nl/blog/de-wetenschap-achter-de-perfecte-kop-koffie](https://www.koffievoordeel.nl/blog/de-wetenschap-achter-de-perfecte-kop-koffie) | | Not started |

### Content verification (template)

| Content | Captured | Details |
|---------|----------|---------|
| Hero / above-the-fold | | |
| Body copy & headings | | |
| Images & alt text | | |
| CTAs / forms | | |
| Footer / legal | | |

---

## Fonts

| Font | Files / notes |
|------|----------------|
| Roboto | `fonts/` (see `styles/styles.css` `@font-face` and variables) |
| Roboto Condensed | `fonts/` |

---

## CSS custom properties (core)

Defined on `:root` in `styles/styles.css` — extend this table when you add brand tokens.

| Token | Usage |
|-------|--------|
| `--background-color`, `--text-color`, `--link-color`, `--link-hover-color` | Base page |
| `--body-font-family`, `--heading-font-family` | Typography |
| `--body-font-size-*`, `--heading-font-size-*` | Type scale |
| `--nav-height` | Header offset |
