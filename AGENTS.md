# AGENTS.md

## This repository: **ko**

Crosswalk project for importing selected **[koffievoordeel.nl](https://www.koffievoordeel.nl/)** pages into AEM Universal Editor + Edge Delivery.

| | |
|--|--|
| **GitHub** | [gabrielwalt/ko](https://github.com/gabrielwalt/ko) |
| **AEM Author** | [content/ko index](https://author-p189820-e1977856.adobeaemcloud.com/content/ko/index.html) |
| **EDS preview** | [main--ko--gabrielwalt.aem.page](https://main--ko--gabrielwalt.aem.page/) |

**Scope**: source URLs, blocks, tokens, import parsers, and migration checklist live in [PROJECT.md](./PROJECT.md). Update that file when you add parsers, new blocks, or complete a page import.

---

**This repository targets AEM Edge Delivery Services with the Universal Editor (UE)** — often called **crosswalk** or **xwalk**. It is **not** a Document Authoring (Google Docs / SharePoint) project.

Typical starting point: [aem-boilerplate-xwalk](https://github.com/adobe-rnd/aem-boilerplate-xwalk) and the [UE tutorial](https://www.aem.live/developer/ue-tutorial).

**What crosswalk means in practice**

- Content is authored in AEM via the Universal Editor and stored in JCR — not in `.md` files as the system of record for published pages.
- Blocks ship **component models** (`_*.json` under `blocks/` and `models/`), compiled with `npm run build:json` — Document Authoring projects do not use this merge step.
- Block JavaScript **must** call `moveInstrumentation()` when you restructure the DOM so UE attributes stay on the right nodes.
- Variants use a `classes` field (or related patterns) in component models — not parenthetical block names in the document.
- Auto-blocking is off; structure is **modelled**.
- `fstab.yaml` uses `type: "markup"` and points at AEM Author — not at Google Drive or SharePoint.

If you catch yourself applying Document Authoring-only patterns (auto-blocking from doc structure, skipping `moveInstrumentation`, editing compiled JSON by hand), stop and align with crosswalk docs.

For performance, accessibility, and general EDS guidance, see [https://www.aem.live/docs/](https://www.aem.live/docs/) and [Keeping it 100](https://www.aem.live/developer/keeping-it-100).

---

## Document Authoring vs Universal Editor (quick reference)

| Aspect | Document Authoring | This stack (UE / xwalk) |
|--------|--------------------|-------------------------|
| Content source | Google Drive / SharePoint | AEM Author (JCR) |
| `fstab.yaml` | Mounts gdrive/sharepoint | `type: "markup"`, `suffix: ".html"`, AEM Author |
| Authoring | Google Docs / Word | Universal Editor |
| Component JSON | Not used this way | `component-definition.json`, `component-models.json`, `component-filters.json` (compiled from sources) |
| Block models | None per block | `_*.json` per block + shared fragments in `models/` |
| Build | Often none for models | `merge-json-cli` → `npm run build:json` |
| Auto-blocking | On | Off |
| DOM instrumentation | N/A | `data-aue-*`, `data-richtext-*` |
| `moveInstrumentation()` | Not needed | Required when moving/replacing nodes |

---

## Critical rules for agents and contributors

### Keep knowledge in the repo

- Prefer **Git-tracked** docs: `AGENTS.md` (how to work), `PROJECT.md` (what exists — blocks, tokens, import, migration).
- After substantive changes, update `PROJECT.md` (blocks, section styles, tokens, parsers, transformers, migration tables).
- Do not rely on editor-only or local-only notes for project conventions.

### Git and content (adjust if your policy differs)

- Many teams prefer **developers** own commits and pushes; align with your house rules.
- For crosswalk, **published content** normally lives in AEM. Local `/content/` or `.plain.html` may be import scratch space — confirm whether your team tracks any of it in Git.

### Import tooling stays in sync

If you change block markup, model fields, or section structure:

- Update **parsers and transformers** under `tools/importer/` when you use that pipeline.
- After model changes, run `npm run build:json` and re-test blocks in UE.
- Prefer **selector-based** parsers and a **single** import entry file unless you have a documented reason to split.

### Block implementation

1. Use `moveInstrumentation()` whenever you replace or reparent instrumented nodes.
2. Run `npm run build:json` after editing any `_*.json` model source.
3. Do not edit compiled root-level `component-*.json` by hand.
4. Reuse blocks and **variants** (`classes`) before inventing new block types.
5. Avoid changing `editor-support.js` unless fixing a concrete UE editing bug.
6. Do not modify vendor `aem.js` unless your fork policy explicitly allows it.
7. Test at `http://localhost:3000` (or your configured dev URL).

---

## Project layout (crosswalk)

```
models/                            # Shared model fragments, section filter, spreads
  _component-definition.json
  _component-models.json
  _component-filters.json
  _page.json, _section.json, _text.json, _title.json, _image.json, _button.json, ...

blocks/{blockname}/
  _{blockname}.json                # definitions, models, filters for this block
  {blockname}.js
  {blockname}.css

component-definition.json          # COMPILED
component-models.json              # COMPILED
component-filters.json             # COMPILED

scripts/
  editor-support.js
  editor-support-rte.js
  dompurify.min.js

tools/importer/                    # optional
  import.js
  import.bundle.js                 # COMPILED
  urls.txt
  parsers/
  transformers/

fstab.yaml
head.html                          # CSP / UE nonce requirements as per your setup
```

Spread syntax like `{ "...": "path#/jsonPointer" }` is resolved by `merge-json-cli` when you run `npm run build:json`.

---

## Component modelling (summary)

Official references: [Universal Editor blocks](https://www.aem.live/developer/universal-editor-blocks), [Component model definitions](https://www.aem.live/developer/component-model-definitions).

### The three compiled JSON artifacts

1. **component-definition.json** — components, resource types, templates.
2. **component-models.json** — fields in the properties panel.
3. **component-filters.json** — allowed children for containers.

### Definition sketch

```json
{
  "title": "My Block",
  "id": "my-block",
  "plugins": {
    "xwalk": {
      "page": {
        "resourceType": "core/franklin/components/block/v1/block",
        "template": {
          "name": "My Block",
          "model": "my-block"
        }
      }
    }
  }
}
```

Use `.../block/item` for repeated items inside container blocks, `.../section/v1/section` for sections, and the core text/title/image/button types for default content as documented by Adobe.

### Container blocks

- **Container** row: `resourceType` block + `template.filter` pointing at a filter.
- **Item** row: `resourceType` block/item + per-item `model`.

### Common field types

| `component` | Role |
|-------------|------|
| `text`, `richtext`, `reference`, `select`, `multiselect`, `aem-content`, `aem-tag`, `boolean`, `number`, `date-time`, `tab`, `container` | As in Adobe docs |

### Field collapse (naming)

- Image pairs: `image` + `imageAlt`.
- Link groups: `link`, `linkText`, `linkTitle`, `linkType` (per current conventions).
- Headings: `title` + `titleType`.

Suffixes are **case-sensitive**.

### Grouped properties

Prefix with `groupName_` to land multiple fields in one table cell.

### Variants via `classes`

```json
{
  "component": "select",
  "name": "classes",
  "label": "Style",
  "valueType": "string",
  "options": [
    { "name": "Default", "value": "" },
    { "name": "Featured", "value": "featured" }
  ]
}
```

Renders classes on the block root (e.g. `.my-block.featured`).

---

## `moveInstrumentation`

From `scripts.js`, `moveInstrumentation(from, to)` copies UE / richtext instrumentation attributes when you change element types or wrappers.

Use it when you:

- Turn row `div`s into `li`, `tr`, `details`, etc.
- Replace `img` with `picture` / optimized picture output.
- Introduce new wrapper elements around instrumented content.

```javascript
import { createOptimizedPicture } from '../../scripts/aem.js';
import { moveInstrumentation } from '../../scripts/scripts.js';

export default function decorate(block) {
  const ul = document.createElement('ul');
  [...block.children].forEach((row) => {
    const li = document.createElement('li');
    moveInstrumentation(row, li);
    while (row.firstElementChild) li.append(row.firstElementChild);
    ul.append(li);
  });
  ul.querySelectorAll('picture > img').forEach((img) => {
    const optimizedPic = createOptimizedPicture(img.src, img.alt, false, [{ width: '750' }]);
    moveInstrumentation(img, optimizedPic.querySelector('img'));
    img.closest('picture').replaceWith(optimizedPic);
  });
  block.replaceChildren(ul);
}
```

---

## New block checklist

1. Add `blocks/{name}/_{name}.json` (definitions, models, filters).
2. Add `{name}.js` / `{name}.css`; use `moveInstrumentation` where needed.
3. Register the block in `models/_section.json` (section filter `components`) if authors should insert it in main content.
4. `npm run build:json`
5. Add `tools/importer/parsers/{name}.js` and wire it in `import.js` **if** you import from a legacy HTML site.
6. Re-bundle the import script after importer changes.
7. Document the block in `PROJECT.md`.

### New variant

1. Extend `classes` (or your variant mechanism) in `_{name}.json`.
2. Style `.blockname.variant` in CSS.
3. Adjust JS only if the DOM shape changes.
4. `npm run build:json`

### Placeholder blocks (no author-editable fields)

1. `"fields": []` in the model.
2. JS renders fixed markup.
3. Importer may emit an empty block table via `WebImporter.Blocks.createBlock(doc, { name: 'Block Name', cells: [] })` if applicable.

---

## CSS conventions

- Scope rules under `.blockname` or `.blockname-*` **element** classes.
- Prefer block-prefixed class names in JS (`blockname-card`, not bare `.card`).
- Avoid `-container` / `-wrapper` on arbitrary block internals if they collide with section language in your project.
- Prefer variant classes from the model over `:nth-child()` / fragile sibling assumptions; authors reorder blocks freely in UE.

---

## Import pipeline — patterns that generalize

Typical **transformer order**:

1. `beforeTransform` — cleanup, synthetic wrappers, **section** `hr` + metadata (often must run **before** parsers destroy wrapper nodes).
2. Parsers — replace matched regions with block tables.
3. `afterTransform` — chrome removal, placeholder conversion, etc.
4. Built-in importer rules (metadata, URLs, backgrounds).

### Section metadata placement

A robust approach: insert section breaks first, then attach metadata to the correct section (often at the **end** of each section’s wrapper, before the next break). Adapt to **your** DOM.

### Heading markup

If cleanup turns colored inline spans into `<em>` (or similar), parsers that rebuild headings should preserve **`innerHTML`**, not `textContent`, so highlights survive.

### “Fake” wrappers

If only certain nodes define section boundaries, you may inject a neutral wrapper in `beforeTransform` so one sections transformer can stay simple. Document the pattern in `PROJECT.md`.

### Shadow DOM

If the source site puts copy inside shadow roots, serialized HTML may **lose** it. Options include a pre-serialization script that copies text/attributes to light DOM `data-*` attributes and parsers that read those attributes. Document the contract in `PROJECT.md`.

### Parser order

When two selectors could match the same node, **registration order** matters. Document ordering rules and false-positive risks in `PROJECT.md`.

### Buttons vs links

If the source uses `<button>` without navigation and your target format expects links, document how parsers normalize that (e.g. `href` placeholders).

### Bundle command (example)

```bash
npx esbuild tools/importer/import.js --bundle --format=iife --global-name=CustomImportScript --outfile=tools/importer/import.bundle.js
```

Run your project’s bulk-import or single-page import command against `import.bundle.js` and your URL list. **Do not** hand-edit `import.bundle.js`.

---

## Handy doc search (EDS)

```bash
curl -s https://www.aem.live/docpages-index.json | jq -r '.data[] | select(.content | test("KEYWORD"; "i")) | "\(.path): \(.title)"'
```

Replace `KEYWORD` with your topic.

---

## Key files

| Path | Role |
|------|------|
| `PROJECT.md` | Blocks, tokens, import, migration |
| `AGENTS.md` | Conventions for this repo |
| `models/_section.json` | Section filter — update when adding blocks |
| `blocks/*/_*.json` | Block UE models (source) |
| `scripts/scripts.js` | `moveInstrumentation`, helpers |
| `scripts/editor-support.js` | UE runtime — change rarely |
| `fstab.yaml` | Content mount |
| `tools/importer/import.js` | Importer entry (if used) |
