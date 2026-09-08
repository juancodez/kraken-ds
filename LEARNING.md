# Learning roadmap — Kraken DS

Personal learning log for working with `weave-ds-template` on the Kraken DS
project. Updated every session with what we did, what we learned, what is
next.

Companion file: [`GLOSSARY.md`](./GLOSSARY.md) — vocabulary reference.

---

## How this file is structured

- **Sessions** — dated log entries. Newest at top.
- **Concepts encountered** — terms that showed up, added to the glossary.
- **Backlog** — things we said we would do but have not done yet.
- **Open questions** — things I do not understand yet, waiting for an answer.

---

## Session 1 — 2026-09-03

**Goal:** get oriented in the repo, understand what it is, set up the personal
learning scaffolding.

**What we did:**
- Cloned `weave-ds-template` from `cris-achiardi/weave-ds-template`.
- Made a working copy at `C:\Users\tn\Kraken Ds Test\` (still with git
  remote pointing to Cristian's repo — needs to be changed before pushing).
- Read `README.md` and `CLAUDE.md` to understand the arc:
  **explore → report → decide → build**.
- Looked at the design source: the "Kraken DS" Figma file. It contains **one
  page called "Research"** with 12 mobile screens (390×844) from the Kraken
  crypto app — Portfolio, Home, Explore, ETH detail, Pay.
- Created `GLOSSARY.md` and this file.

**Concepts encountered (see [glossary](./GLOSSARY.md)):**
- ADR — Architectural Decision Record
- Contract vs decision
- Token (primitive vs semantic)
- DTCG spec
- Style Dictionary
- Upstream / downstream
- Monorepo & pnpm workspace
- Sandbox vs Storybook
- Gate vs report
- MCP (Model Context Protocol), Figma Console MCP, Figma Desktop Bridge
- Scope (`@ds/*` → `@kraken/*`)

**Open questions:**
- What scope name do we brand with? Provisional: `kraken`.
- Should the folder be renamed to `kraken-ds-test` (no spaces) before we run
  pnpm scripts? Spaces in paths can break tooling on Windows.
- Do we push this to a new GitHub repo now, or keep it local until v0?

**Backlog (next session):**
1. Decide the scope name and run `pnpm init-ds <name> --dry` (inspect first).
2. If good, run `pnpm init-ds <name>` for real, then `pnpm install`.
3. Run `pnpm verify` — every gate should pass green on an empty repo. This
   is the template's acceptance test.
4. Run `pnpm dev` — see the empty sandbox at `localhost:4300`.
5. Then start the **explore** step: pick ONE Kraken screen (probably
   `Portfolio — Chart (small)`) and write down what is measurably there in
   `docs/research/`.

---

### Session 1 — addendum (repo tour)

Walked through every folder and file at the root, understanding purpose and
Cristian's rationale.

**Learned:**
- `ds.config.json` is the single source of truth for scope, token prefix,
  and data-attribute prefix. All four values must move together — that is
  why `pnpm init-ds` is a codemod and hand-renaming is banned.
- `packages/` holds four things: `contracts` (the agnostic product), `tokens`
  (DTCG JSON → compiled outputs), `react` (one framework binding), and
  `platform-web` (DOM-level conformance).
- `apps/` has `sandbox` (fast Vite harness, in the workspace) and `storybook`
  (deliberately NOT installed on day one — you pay its cost only when it
  earns it).
- `docs/` has three zones: `research/` (pre-decision), `ADR/` (post-decision),
  `documentation/` (plain-language for non-code-readers).
- `.figma/` maps design-source nodes to component names — explicit mapping,
  not auto-swept. Figma is a canvas to read, not a pipeline stage.
- `.ai/maps/` holds the generated prop glossary. Useful even on an empty repo.
- `.claude/skills/` ships five agent skills: `ds-decide`, `ds-component`,
  `ds-figma-explain`, `ds-figma-document`, `ds-figma-component`. They
  automate what is worth automating; the rest stays manual on purpose.
- The `~5.9.3` TypeScript pin in `package.json` is deliberate — TS 6.x
  silently breaks `react-docgen-typescript`'s enum extraction and every
  contract thins out with no warning.

**Mental model:** Figma (design source) → research (write it down) → ADR
(decide) → contracts (spec) → tokens + react (build) → sandbox (see it).
Arrows go one way. Nothing skips.

**New glossary entries added:** *(none this pass — all the terms we
encountered were already in the initial glossary)*

---

## Session log template (copy for each new session)

```
## Session N — YYYY-MM-DD

**Goal:** what we set out to do.

**What we did:**
- ...

**Concepts encountered:**
- ...

**Open questions:**
- ...

**Backlog (next session):**
- ...
```
