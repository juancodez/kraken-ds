# Glossary — Design System vocabulary

Personal learning glossary. Terms in plain language, with what they mean in
**this repo** specifically, not just the textbook definition.

Terms are grouped by theme. Each entry: what it is, why it matters here.

---

## Introduction — the whole ecosystem

### The problem this system solves

Most design systems get built backwards: someone draws a Button, someone else
codes it, and then everyone argues about what it should have been. The result
is a library full of components that work but disagree with each other —
different prop names for the same idea, different token names for the same
colour, no record of why anything is the way it is.

Cristian's system runs the other way. **Nothing gets built until a decision
exists. Nothing gets decided until evidence exists.** The sequence is strict:

> **Explore → Report → Decide → Build**

This is not a philosophy. It is enforced by the tools: the build fails if a
contract contradicts the code, CI fails if the prop glossary drifts, and the
ADR index fails if a decision isn't recorded. You cannot skip steps and have
a green build.

---

### The five layers of the ecosystem

```
┌─────────────────────────────────────────────────────────────────┐
│  1. BRAND & TOKENS                                              │
│  Brand decisions → Primitives → Semantic roles → Component slots│
│  Files: tokens/*.json → Style Dictionary → build/*.css + *.ts  │
└───────────────────────────┬─────────────────────────────────────┘
                            │ CSS custom properties (--ds-color-*)
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│  2. CONTRACTS (the product)                                     │
│  One JSON file per component. Framework-agnostic. States:       │
│  purpose, behavior, anatomy, axes, a11y, token policy per part. │
│  Files: contracts/components/<Name>/<Name>.contract.json        │
│         contracts/components/<Name>/CHANGELOG.md               │
└───────────────────────────┬─────────────────────────────────────┘
                            │ contract is consumed by
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│  3. INHERITED CONVENTIONS & NAMESPACE                           │
│  The shared vocabulary every component must speak.              │
│  Props Map validates that no component coins a synonym.         │
│  Files: contracts/prop-canon.json (the canon, hand-maintained)  │
│         .ai/maps/prop-map.{json,md} (measured reality, generated)│
└───────────────────────────┬─────────────────────────────────────┘
                            │ contract + conventions feed
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│  4. COMPONENT (logic + styles)                                  │
│  The contract builds two things in parallel:                    │
│  - Component Logic (.tsx) — CVA variant axes, state tracking    │
│  - Component Styles (.module.css) — token variables applied     │
│  Together they generate the Component API (the public surface). │
│  React binding: react/bindings/<Name>.react.json               │
│  Files: packages/react/src/<Name>.tsx                           │
│         packages/react/src/<Name>.module.css                    │
└───────────────────────────┬─────────────────────────────────────┘
                            │ component API produces artifacts
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│  5. ARTIFACTS                                                   │
│  Three outputs — all generated FROM the contract, never manually│
│  - Coded component (packages/react/dist/)                       │
│  - Figma component (wired via .figma/ manifest)                 │
│  - Component docs (Storybook or custom)                         │
└─────────────────────────────────────────────────────────────────┘
```

**Design decisions are not a one-time input — they loop back.** When a design
decision changes, it modifies the contract. The contract modifies the
component logic and styles. The artifacts regenerate. The loop is:

> Design decisions → Contract → Logic + Styles → API → Artifacts
> ↑                                                           │
> └─────────── Explore to modify the contract ────────────────┘

---

### What goes in each layer and why

**Layer 1 — Brand & Tokens**
The brand decision (which colours, which scale, which radius) lives as DTCG
JSON in `packages/tokens/tokens/`. Style Dictionary compiles that into CSS
custom properties. Everything downstream uses variables (`var(--ds-color-fill-loud)`),
never raw values (`#5146e6`). Changing a token re-themes every component that
uses it — for free, without touching any component code.

Three tiers, strict direction of reference:
- Primitive (`color.indigo.500 = #5146e6`) — raw palette, no meaning
- Semantic role (`color.fill.loud → color.indigo.500`) — meaning, no raw value
- Component slot — only used when one component needs a token nobody else shares

Components style against **semantic roles only**. Never against brand families
(`--ds-color-brand-*`), because a component pinned to a brand colour cannot
be re-themed.

**Layer 2 — Contracts**
The contract is what the component IS. Not how it looks — what it does, what
states it can be in, what the consumer controls vs what the user controls,
which parts make up its anatomy, and which CSS channel each part exposes for
theming. One JSON file per component. No framework inside it — it can compile
to React, React Native, or a web component.

**Layer 3 — Inherited conventions (Props Map)**
Every component must speak the same vocabulary. `size` always uses
`xs|s|m|l|xl`, never `small|medium|large`. `variant` names an intent, never
a colour. The Props Map is generated from actual component source and compared
against the canon — divergences are flagged. This is what stops two engineers
independently coining `emphasis` and `hierarchy` for the same concept.

**Layer 4 — Component (logic + styles)**
The contract builds two things:
- **Logic** — the CVA variant map (which class names apply for which axis
  values), state tracking, event handling, accessibility attributes
- **Styles** — the CSS that references token variables

Neither is complete without the other, and neither is the source of truth.
The contract is. The gate (`verify:contract`) asserts the two agree with it.

**Layer 5 — Artifacts**
Everything generated. The coded component (what a consumer installs), the
Figma component (what a designer uses), the docs (what everyone reads). None
of these are authored by hand — they are outputs of the layers above.

---

### The AI layer

This is what makes the system "AI-powered." Three tools:

**MCP (Model Context Protocol)**
The protocol that lets an AI agent call external tools. In this system the
main MCP connection is to Figma — the agent can read the design file, inspect
variables, get component names and structure, without a human copy-pasting
screenshots.

**Agent**
A model (Claude) that uses MCP tools to take actions: read the Figma file,
write a research doc, propose a contract, run the verify gate and report what
broke. The agent does not make decisions — it surfaces evidence and executes
recorded decisions.

**Skill (procedure)**
A reusable prompt + instruction set that tells the agent how to do a specific
job. This repo ships five skills (`.claude/skills/`):
- `ds-decide` — turns a research report's open questions into an ADR
- `ds-component` — retired flow; describes how a component was written by hand
- `ds-figma-explain` — explains what is in a Figma file in plain language
- `ds-figma-document` — writes a research doc from Figma inspection
- `ds-figma-component` — bridges Figma component structure into the contract

**Context window & progressive disclosure**
The agent has a limited working memory (context window). The system is
designed so nothing is loaded up front — only the doc that governs the
specific thing being touched. This is why every README is small, every ADR is
narrow, and CLAUDE.md says "pull the right context at the right time."

---

### The five traps (what goes wrong in every DS)

Cristian names these explicitly. Knowing them is knowing what the system's
gates are guarding against.

**Drift**
When the source of truth and the actual code disagree. A Figma component says
one thing; the coded component does something different. A contract says one
axis; the TSX implements another. The gates (`verify:contract`, `prop-map:check`)
exist specifically to detect drift automatically.

**Single source of truth**
Every fact must live in exactly one place. Tokens in `tokens/`, prop names in
`prop-canon.json`, component spec in the contract. Copying a fact into two
places guarantees they eventually disagree, and one of them is wrong.

**Variant is not a prop**
A visual variant (`size`, `hierarchy`, `variant`) is an axis in the contract.
A state (`disabled`, `loading`, `checked`) is a state. They cannot be mixed.
A state in an axis produces broken accessibility trees because the platform
already knows about `disabled` — giving it a custom prop fights the browser.

**Measure, don't eyeball**
A research doc records numbers, not adjectives. "The Figma file has 41
variables across 3 collections" is evidence. "The file has quite a lot of
variables" is noise. Decisions made on adjectives cannot be re-evaluated
later.

**Generated / validated / reviewed**
Three different things that look the same but are not:
- Generated — a script produced it. It is consistent with its inputs, but
  the inputs may be wrong.
- Validated — a gate checked it against a schema or another file. It is
  legal, but legal ≠ true.
- Reviewed — a person read it and agreed it describes reality. Only this
  last one catches a plausible-but-wrong purpose statement or an accessibility
  claim nobody tested.

The gates handle generated and validated. Reviewed is always on you.

---

---

## Core arc

**Explore → Report → Decide → Build**
The four steps Cristian teaches. You do not draw a Button first. You read the
design source, write down what is measurably there (a *report*), turn the open
questions into recorded *decisions* (ADRs), then build the component against
a decision that already exists.

**Design source**
The Figma file (or files) we read from. In our case: the "Kraken DS" file. It
is a **canvas to read**, not a pipeline stage — nothing in Figma is
auto-swept into code.

---

## Governance

**ADR — Architectural Decision Record**
A short markdown file that records a decision, why it was made, and what
alternatives were considered. Lives in `docs/ADR/`. Once accepted, other work
must respect it or update it — silently ignoring one is a defect.

**Decision vs contract**
An ADR is the *decision* (agnostic, high-level). A **contract** is the
mechanical realisation of that decision (schemas, JSON files, scripts) that
lives next to the code. The ADR *links* its contracts. This separation lets a
decision stay stable while its implementation moves.

**Draft vs Accepted (ADR status)**
Pre-v0 (before first release), most ADRs stay Draft. An ADR only becomes
Accepted once **the code that realises it exists**. After v0, you supersede
old ADRs instead of editing them.

---

## Component vocabulary

**Contract (component contract)**
A `<Name>.contract.json` file for each component. It states what the source
code cannot state on its own: which HTML element renders, where the forwarded
ref lands, what a slot accepts, accessibility commitments, and **which family
of token is allowed to paint which channel of which node**. The gate checks
that the code and the contract agree.

**Prop glossary**
Generated file (`.ai/maps/prop-map.md`) that lists every prop used across the
system, its type, and where it appears. Descriptive, not prescriptive.
Regenerated by `pnpm prop-map`. Hand-editing it fails CI.

**Slot**
A place inside a component where the *user* (of the component) passes in
content — typically `children`, or a named slot like `leadingIcon`.

**Part**
A named piece of a component that renders. Contracts name the parts so gates
can check nothing is claimed that does not actually render.

**Axis / variant**
An axis is a dimension of variation for a component (e.g. `size`, `tone`,
`intent`). Its values are the variants (`sm | md | lg`). Contracts declare
the axes; the source must implement exactly those values.

**CVA — Class Variance Authority**
A tiny library used to bind variant names to CSS classes in a type-safe way.
The library here uses CVA + CSS Modules.

**CSS Modules**
CSS files scoped to a component. Class names get hashed at build time so they
cannot collide with other components' classes.

---

## Tokens & the pipeline

**Token**
A named design value: a colour, a spacing step, a font size. Not a raw hex
code — a *reference* to a hex code that has a meaning. `--kraken-color-bg-primary`
is a token; `#0f172a` is a value.

**Primitive vs semantic token**
Primitive: raw palette (`color-neutral-900`). Semantic: what it means in
context (`color-bg-primary`). Semantic tokens reference primitives; components
consume semantic tokens.

**DTCG — Design Tokens Community Group**
The spec for the JSON format tokens are written in. `{ "$type": "color",
"$value": "#0f172a" }` is DTCG. The industry-standard shape.

**Style Dictionary**
The tool that reads DTCG JSON and writes it out as CSS custom properties, TS
constants, or any other target. It is the *pipeline* between token JSON and
what a component consumes.

**CSS custom property**
`--kraken-color-bg-primary: #0f172a;`. Native browser feature. The tokens
compile to these; components use `var(--kraken-color-bg-primary)`.

**Upstream / downstream**
Upstream = what your DS depends on (e.g. NativeWind, Phosphor icons).
Downstream = what depends on your DS (the apps that consume the components).
The DS lives in the middle: extends the upstream, feeds the downstream.

---

## Repo shape

**Monorepo**
One repository holding many packages. Ours holds `@kraken/tokens`,
`@kraken/react`, `@kraken/contracts`, plus `apps/sandbox` and `apps/storybook`.

**pnpm workspace**
The mechanism that lets packages inside a monorepo reference each other by
name (`@kraken/react` can import from `@kraken/tokens` without publishing).

**Scope (npm scope)**
The `@kraken` in `@kraken/react`. Namespaces the package. Set once by
`pnpm init-ds kraken` — cannot be safely renamed by hand later.

**Sandbox**
`apps/sandbox` — a fast Vite page that boots in ~1s and points directly at
component source. For rapid, single-component work.

**Storybook**
`apps/storybook` — the classic component gallery. Present on disk but
**deliberately not installed** by default (too heavy). One line to switch on.

---

## Gates & CI

**Gate**
An automated check that fails the build if broken. `pnpm verify` runs every
gate locally; CI runs them again. A gate that only runs locally is not a
gate — it is enforced on whichever machine happens to run verify.

**Report (vs gate)**
A check that *reports* findings without failing. Uncontracted components and
paint findings are reports, not gates — because a gate that fails on
everything on day one gets switched off, and a switched-off gate protects
nothing.

**CI — Continuous Integration**
The remote pipeline (GitHub Actions) that runs the gates on every push. If CI
is green, the change is safe to land.

---

## Tools around the repo

**MCP — Model Context Protocol**
The protocol that lets Claude (or other agents) call external tools like
"read this Figma file" or "generate a screenshot". Each capability is an MCP
tool.

**Figma Console MCP**
An MCP server that gives Claude access to Figma's REST API (read libraries,
comments, variables, etc.) — beyond what the raw Plugin API can do.

**Figma Desktop Bridge**
A companion plugin that lets Claude talk to the Figma **desktop app** live
(select nodes, get screenshots, run scripts inside the file).

**Personal Access Token (PAT)**
A per-user key generated in Figma settings that authenticates the MCP server
against Figma's REST API on your behalf.

---

## Package.json entries you will see

**pnpm init-ds &lt;name&gt;**
One-shot brand rename. Turns `@ds/*` → `@name/*`, `--ds-*` → `--name-*`,
`data-ds-*` → `data-name-*`. Run **before** any component is written.

**pnpm verify**
Runs the full local gate chain: format → typecheck → contract → prop-map →
adr-index → docs → figma → build → test.

**pnpm dev**
Boots the sandbox at `localhost:4300`.

**pnpm contract &lt;Name&gt;**
Merges the component source and the contract, tells you what the component
*is*. No build step.

---

## Things Cristian says that sound like jargon

**"Contracts vs libraries"**
His shorthand for: the contract layer is more important than the library that
implements it. Two libraries can implement the same contract; the contract is
what stays stable.

**"Derivation chain"**
The chain from brand decisions → primitive tokens → semantic tokens →
component styles → app usage. Every step is downstream of a decision made
one level up.

**"Axes registry"**
The central list of allowed axes (`size`, `tone`, `intent`) with allowed
values. Prevents each component from inventing its own vocabulary.

**"Variant ≠ status"**
A variant is a visual axis (a Button can be `size=lg`). A status is a runtime
state (`disabled`, `loading`). They must not be confused because they behave
differently in a11y trees.

**"Prop ownership"**
For any prop, exactly one place owns its definition. Copying a prop
definition into another component is a defect — reuse it or contract it.

---

---

## File formats

**`.md` — Markdown**
Plain text with lightweight formatting (`# Heading`, `**bold**`, `- list`).
Not code — just writing. Every README, ADR, GLOSSARY, and LEARNING file is
Markdown. Renders as formatted docs on GitHub.

**`.json` — JSON**
Structured data: keys and values, no logic, no functions. Machines and humans
can both read it. In this repo JSON does a lot of work:
- `package.json` — what the project is, its scripts, its dependencies
- `ds.config.json` — brand identity (scope, prefix)
- `*.contract.json` — component contracts (the heart of the system)
- `*.react.json` — React-specific bindings per component
- `prop-canon.json` — canonical list of allowed prop names and values
- `pnpm-lock.yaml` — exact versions of every installed dependency (auto-generated, never hand-edit)

**`.ts` — TypeScript**
JavaScript with types. You declare what kind of data a variable holds and the
compiler checks that before the code runs, catching mistakes early. Used for
all library code in `packages/react/src/` and for config files like
`vite.config.ts`.

**`.tsx` — TypeScript + JSX**
TypeScript that also contains HTML-like syntax (JSX) for writing React
components. Every component file would be `.tsx`. The JSX part lets you write
`<Button size="m" />` directly in TypeScript.

**`.mjs` — ES Module JavaScript**
Plain JavaScript using modern `import/export` syntax. The `.mjs` extension
tells Node.js to treat it as a module explicitly. All the scripts in this repo
are `.mjs`: `verify-contract.mjs`, `build-prop-map.mjs`, `init-ds.mjs`, etc.
You run them with `node` or via a `pnpm` script.

**`.css` — CSS**
Styling rules for HTML: selectors and properties. Components use
`.module.css` (CSS Modules) — same syntax but class names get hashed at build
time so they cannot collide between components. A consumer targets components
via `data-*` attributes, not class names.

**`.html` — HTML**
The skeleton of a web page. Only `apps/sandbox/index.html` exists here — the
single entry page that Vite loads, then injects the React app into.

**`tsconfig.json` — TypeScript config**
Tells the TypeScript compiler how to behave: which files to include, how
strict to be, which JS version to target. Not code — configuration. The repo
has `tsconfig.base.json` (shared rules) and per-package overrides.

**`vite.config.ts` — Vite config**
Vite is the build tool and dev server. This config says where the entry point
is, whether to build a library or an app, and which plugins to use.

**`pnpm-workspace.yaml` — workspace definition**
A tiny YAML file that tells pnpm which folders are sub-packages of the
monorepo. YAML is like JSON but uses indentation instead of braces.

**`.prettierrc` / `.prettierignore` — Prettier config**
Prettier is a code formatter — it auto-reformats code to a consistent style.
`.prettierrc` sets the rules; `.prettierignore` lists files to skip.
`pnpm format:check` is the first gate in `pnpm verify`.

**`.npmrc` — package manager config**
Low-level pnpm/npm settings: which registry to use, hoisting rules, etc.

**`.nvmrc` — Node version**
A one-line file with a Node.js version number (`20`). Tools like nvm (Node
Version Manager) read it to switch to the right Node version automatically
when you enter the folder.

**`.gitignore` / `.gitattributes` — Git config**
`.gitignore` lists files Git should not track (`node_modules/`, build
output). `.gitattributes` tells Git how to handle line endings and diffs for
specific file types.

**`.github/workflows/verify.yml` — GitHub Actions CI**
YAML that defines what GitHub runs automatically on every push. In this repo
it runs `pnpm verify` — the same chain you run locally. This is what makes
the gates actually enforced rather than just advisory.

---

## pnpm

A **package manager** — the tool that downloads and organizes the external
code your project depends on.

Every JavaScript project uses libraries other people wrote. Instead of
downloading them manually, you list what you need in `package.json` and run
`pnpm install` — it downloads everything into `node_modules/` and locks the
exact versions in `pnpm-lock.yaml`.

**Why pnpm and not npm or yarn?** Same job, but faster and uses less disk
space. pnpm also handles monorepos better — packages inside the same repo can
reference each other by name without being published anywhere.

**Workspace** = pnpm's word for "these sub-projects can reference each other
by name." So `packages/react` imports from `@ds/tokens` and pnpm resolves
that to `packages/tokens/` without publishing anything.

---

## How components and tokens connect

This is the full chain from design source to a working component. Every arrow
goes one way.

```
Figma file  (the design source — canvas to READ, never auto-swept)
    │
    ▼  hand-measured by a person, written as a research doc
docs/research/<name>.md   (what is measurably there)
    │
    ▼  open questions become decisions
docs/ADR/<number>-<name>.md   (what we decided and why)
    │
    ├─────────────────────────────────────────────────┐
    ▼                                                 ▼
packages/tokens/tokens/*.json              packages/contracts/components/
(DTCG token source — primitives,           <Name>/<Name>.contract.json
 semantic roles, component slots)           (the component spec: purpose,
    │                                        anatomy, states, axes, a11y,
    ▼  Style Dictionary compiles             token policy per part)
packages/tokens/build/                          │
  *.css  → CSS custom properties               ▼  React binding
  *.ts   → typed TS constants        packages/react/bindings/
                                      <Name>.react.json
    │                                  (element, ref target, notes)
    │                                       │
    └──────────────────────────────────────▼
                              packages/react/src/
                               <Name>.tsx + <Name>.module.css
                               (the actual component code)
                                    │
                                    ▼  Vite builds it
                              packages/react/dist/
                               (ESM + CJS + .d.ts + one stylesheet)
                                    │
                                    ▼
                              apps/sandbox/  or a consumer app
                               (imports and uses the component)
```

**What each file in this chain does:**

`tokens/*.json` — the source of truth for every design value (colours,
spacing, radii, type scales). Written in DTCG format. Empty until measured
from Figma. Primitives reference nothing; semantics reference primitives;
component slots reference semantics.

`packages/tokens/build/` — the compiled output. Style Dictionary reads the
DTCG JSON and writes CSS custom properties (`--ds-color-fill-loud`) and
typed TS constants. Components never read the token JSON directly — they read
the compiled CSS variables.

`<Name>.contract.json` — what the component IS. Framework-agnostic. Declares
the anatomy (named parts), the states, the axes (variant dimensions), and
**which CSS channel each part paints and what token family may supply it**.
This is the token policy: a contract does not say *what* to use — it says
*which socket exists* for a consumer to wire.

`<Name>.react.json` — the React-specific binding on top of the contract.
Says which HTML element renders, where the forwarded ref lands, which part
absorbs `className`. Small file — everything not React-specific lives in the
contract.

`<Name>.tsx` — the component code. Uses CVA to map axis values to CSS
classes, and CSS Modules for the classes themselves. Named nodes carry both
`data-ds-part="x"` (public styling target) and `className={styles.x}` (the
actual style). The component never hard-codes a token value — it uses
`var(--ds-color-fill-loud)` from the compiled token output.

`<Name>.module.css` — the component's styles. References token CSS variables.
The hashed class names are the private implementation; the `data-*` attributes
are the public surface.

**The three levels of token:**

| Level | Example | References |
|-------|---------|------------|
| Primitive | `--ds-color-indigo-500: #5146e6` | nothing, it is a literal |
| Semantic role | `--ds-color-brand-primary` → `var(--ds-color-indigo-500)` | primitives only |
| Component slot | `--ds-color-fill-loud` → `var(--ds-color-brand-primary)` | semantic roles only |

A component styles against the **semantic role** level
(`--ds-color-fill-*`, `--ds-color-border-*`, `--ds-color-on-*`), never against
a branded family. This is what makes every variant "free": changing what
`--ds-color-fill-loud` points to re-themes every component that uses it.

---

*This file is personal notes. Update it whenever a new term shows up. Do not
edit generated files based on this — the generated files are the source of
truth for their own domain.*
