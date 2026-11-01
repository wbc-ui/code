<p align="center">
  <strong>The code editor as data. Vue 3.</strong><br/>
  <em>One component. Editor, preview, markdown, samples and formatter — configured by a single object.</em>
</p>

Official documentation, demo, cases, and Interactive Lab: [code.wbc-ui.com](https://code.wbc-ui.com).

<p align="center">
<img src="https://img.shields.io/badge/npm-1.0.0--r01-1976D2" alt="version">
<img src="https://img.shields.io/badge/license-MIT-blue" alt="license">
<img src="https://img.shields.io/badge/vue-3.5%2B-42b883" alt="vue">
</p>

---

## Why?

### Write an editor in one component

```html
<WBCode :item="{ code: 'console.log(1)', language: 'javascript' }" />
```

No worker config, no toolbar markup. One tag, one object.

### The panels are data, not markup

```javascript
{
  code: '# Hello',
  language: 'markdown',
  display: ['code', 'preview', 'markdown'],   // which panels exist
  autoUpdate: ['preview']                     // which re-render as you type
}
```

Add `'html'`, `'upload'`, `'samples'` or `'formatter'` to `display` and the panel appears. Remove it and it's gone. There is no template to edit.

### Host-owned config, package-owned UI

Options passed to `app.use(WBCode, { … })` are stored on `$wbcCodeOptions` and readable by every instance — a host sets policy once and individual editors stay declarative.

---

## What is @wbc-ui/code?

A **Vue 3 plugin** that registers a single component, **`<WBCode>`**, built on `@wbc-ui/core`. It renders *through* the WBC engine rather than around it — the editor's output **is** a WBC tree, so anything WBC can render can appear inside it.

Syntax highlighting is **Prism**, bundled with nine grammars preloaded — markup, javascript, python, bash, json, css, markdown, latex, rust — plus Prism's default theme.

| Piece | Role |
|---|---|
| `<WBCode :item="…">` | The whole editor — panels, language, samples, formatting |
| `item.display` | Which panels exist |
| `item.autoUpdate` | Which panels re-render live as you type |
| `item.samplesss` | Per-language starter snippets |

---

## Usage Examples

### Level 1 — Hello, WBCode

```html
<WBCode :item="{ code: 'console.log(\'Hello World\');', language: 'javascript' }" />
```

### Level 2 — Markdown editor with live preview

```html
<WBCode :item="{
  code: '# Title\n\nSome **markdown**.',
  language: 'markdown',
  display: ['code', 'markdown'],
  autoUpdate: ['markdown']
}" />
```

### Level 3 — Full workbench with samples

```html
<WBCode :item="{
  code: 'print(\'hi\')',
  language: 'python',
  filename: 'main.py',
  display: ['code', 'preview', 'html', 'samples', 'formatter'],
  autoUpdate: ['preview'],
  samplesss: {
    python: ['print(\'hello world\')', 'for i in range(3): print(i)'],
    javascript: ['console.log(\"Hello\")']
  }
}" />
```

---

## The `item` prop

`item` is the whole configuration surface. Every key is optional.

| Key | Type | Purpose |
|---|---|---|
| `code` | String | The initial source. |
| `language` | String | Active language for highlighting. |
| `languages` | String | Comma-separated list offered in the language picker. |
| `display` | Array | Panels to render: `code`, `preview`, `html`, `upload`, `samples`, `markdown`, `formatter`. |
| `autoUpdate` | Array | Which of those panels re-render as you type. |
| `filename` | String | Name used by the download / upload panel. |
| `samplesss` | Object | Per-language array of starter snippets. |
| `codeDive` | Boolean | Enables the reactive dive mode inherited from WBC. |
| `global` | String | Global namespace accessor for evaluated code. |

> **The default `item` is not empty.** Rendering `<WBCode />` with no `item` gives a populated demo editor — `code: "# WBCode Vue 3"`, all seven panels enabled, sample snippets for five languages. Pass an explicit `item` for anything real.

---

## Props

| Prop | Type | Default | Description |
|---|---|---|---|
| `item` | Object | *(populated demo config — see above)* | The editor descriptor. |
| `loading` | Boolean | `false` | Loading state indicator. |
| `global` | String | `null` | Optional global namespace accessor. |

> **Not listed: `wbCode`.** The component also declares a `wbCode` prop that wraps the
> editor in a code-viewer decoration. It is a **developer affordance and not part of the
> supported API** — it is deliberately undocumented here, and application code should not
> depend on it. The prop is now **dev-gated with a fail-closed guard** — it resolves through
> store authorization and falls back to the plain editor without it. See
> `_meta/adr_011_wbcode_prop_gating.md`.

---

## Installation

### Prerequisites

- **Node.js** ≥ 18
- **Vue 3.5+** and **`@wbc-ui/core`** — both peer dependencies

### npm

```bash
npm install @wbc-ui/code

# Peer dependencies — install once per project
npm install @wbc-ui/core vue@^3.5.0
```

### Yarn / pnpm

```bash
yarn add @wbc-ui/code @wbc-ui/core vue@^3.5.0
pnpm add @wbc-ui/code @wbc-ui/core vue@^3.5.0
```

### Peer dependencies

| Package | Required? | Notes |
|---|---|---|
| `@wbc-ui/core` (`^1.0.0-r01`) | **required** | The engine this component renders through. |
| `vue` (`^3.5.0`) | **required** | Vue 3 runtime. |

Bundled, not peers: `prismjs`, `html2pdf.js`.

### Vue 3 plugin registration

```javascript
import { createApp } from 'vue';
import WBCCore from '@wbc-ui/core';
import WBCode from '@wbc-ui/code';
import App from './App.vue';

const app = createApp(App);
app.use(WBCCore);   // always first — WBCode inherits WBC's methods
app.use(WBCode);
app.mount('#app');
```

Registration is **eager** — `<WBCode>` is available immediately, which is what makes it usable inside markdown-driven pages where a lazily registered component renders as a blank placeholder. `install` is idempotent per app.

### Named imports

```javascript
import { WBCode } from '@wbc-ui/code';
```

### Troubleshooting common install errors

| Symptom | Cause | Fix |
|---|---|---|
| `<WBCode>` renders as an unknown element | The plugin wasn't installed on this app | `app.use(WBCode)` — and after `app.use(WBCCore)`. |
| Editor renders, nothing is highlighted | The language isn't among the nine preloaded Prism grammars | Import it yourself: `import 'prismjs/components/prism-go'`. |
| A panel is missing | `display` doesn't list it | Add the panel name to `item.display`. |
| WBC methods are undefined inside the editor | `@wbc-ui/core` was installed *after* this package | Install core first — method inheritance happens at module evaluation. |

---

## ⚡ The Component Under the Hood

- Registers exactly one component: **`WBCode`**
- **Inherits `WBC.methods`** at module-evaluation time, so the editor shares the engine's helper surface
- Renders its output **through `<WBC>`** — the editor is a WBC tree, not a wrapper around one
- Prism grammars and the default Prism theme are **bundled**, not fetched at runtime

---

## 🌐 Ecosystem

| Package | What it adds |
|---|---|
| `@wbc-ui/core` | The rendering engine — **required** |
| `@wbc-ui/chart` | Apache ECharts integration |
| `@wbc-ui/latex` | KaTeX math rendering |
| `@wbc-ui/mermaid` | Diagram-as-code rendering |
| `@wbc-ui/alert` | Floating debug / alert panel |
| `@wbc-ui/dataviewer` | JSON / data-table explorer |

---

## Build artifacts

| Field | Path |
|---|---|
| `module` (ESM) | `./dist/index.js` |
| `main` (CJS) | `./dist/index.cjs` |

Published files: `dist/`, `README.md`, `LICENSE`.

---

## Exports

| Export | Kind | Purpose |
|---|---|---|
| `install` | function | Vue 3 plugin installer — `install(app, options)`. |
| `WBCode` | component | The editor component. |

Default export: `{ install }`.

---

## 🛡️ B1 non-regression guard

A test in `src/tests/B1_guard.test.js` walks every source file and fails on any **bare `h('Capitalized')` call** — the Vue 3 inert-element trap:

```javascript
h('WBC', { ... })       // emit <wbc>, component never mounts — BUG
h(resolveDynamicComponent('WBC'), { ... })  // correct
```

The guard runs as part of `npm test` (vitest). The allow-list has 3 entries for commented-out dead code (2 in tests/wbCodeGating.test.js, 1 in WBCode.vue:2295) — each with its reason inline. If a future change needs a bare `h('Name')` call in production code, **add an entry to `ALLOW_LIST` in the test file with the reason inline** — an unexplained exemption is how wb-code's partially-swept state went unnoticed.

## 📄 License

MIT © [Wissem Boughamoura](https://github.com/wissemb11) · [wi-bg.com](https://www.wi-bg.com) · [wbc-ui.com](https://wbc-ui.com)
