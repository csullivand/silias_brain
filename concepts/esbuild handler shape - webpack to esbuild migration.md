---
tags: [concept, silia, infra, esbuild, lambda]
---

# esbuild handler shape — webpack→esbuild migration

## The trap
Silia is mid-migration from **webpack** to **esbuild** bundling for Lambdas (esbuild.config.js at repo root; `building` script; commits "migrate X to esbuild"). The two bundlers export the Lambda handler DIFFERENTLY, and SAM `Handler:` strings must match:

- **webpack** (`libraryTarget: commonjs2`, `library: <filename>`): a source using `export const handler` gets NESTED → `module.exports.<component>.handler`. So the template handler is **`xMin.x.handler`** (e.g. `listGrantsMin.listGrants.handler`, `indexMin.index.handler`).
- **esbuild** (`format: 'cjs'`, no globalName/footer): named exports land FLAT on `module.exports` → `module.exports.handler`. So the template handler must be **`xMin.handler`** (e.g. `listGrantsMin.handler`).

Sources using `exports.handler =` are flat under BOTH → `xMin.handler` either way.

## The symptom
After migrating a module's build to esbuild but leaving the webpack-era nested `Handler:` string, Lambda fails at cold start:
`Runtime.HandlerNotFound: xMin.x.handler is undefined or not exported`
(because `module.exports.x` is undefined under esbuild → `.handler` on undefined).

## How to confirm which shape a bundle uses
- Look at bundle tail: webpack → `__webpack_require__` + `module.exports.<name>=...`; esbuild → `__export(... {handler})` + `module.exports = __toCommonJS(...)`.
- Quick esbuild probe: `esbuild probe.ts --bundle --format=cjs` then `require(out).handler` (flat) vs `.probe` (undefined).

## The fix
Flatten nested handlers to `xMin.handler` once the module is on esbuild. Known stragglers (2026-08): `Access/infrastructure/aws.template.yml` (5 grant fns) + `DynamicTables/infrastructure/aws.template.yml` (5 `indexMin.index.handler`).

## Gotcha
`scripts/ci/probe-handler-shape.js` loads each bundle and looks for the handler, but its `findHandler()` searches ALL keys (flat OR nested), so it does NOT catch a nested-vs-flat mismatch between the template's exact `Handler:` string and the bundle. It catches load-fail / missing-bundle / callback-only, not this. Harden it to resolve the EXACT `Handler:` path.

Related: [[Claude Sessions/silia/SL-1432-detail-view-reconcile/2026-08-13]]