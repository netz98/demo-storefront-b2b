# AGENTS.md

## Scope

- This file applies to the entire repository.
- It is intended for coding agents working in `demo-storefront-b2b`.
- There was no existing repo-level `AGENTS.md` to extend.
- No Cursor rules were found in `.cursor/rules/` or `.cursorrules`.
- No Copilot instructions were found in `.github/copilot-instructions.md`.

## Repo Shape

- The main storefront lives at the repository root.
- End-to-end tests live in `cypress/`.
- A small helper tool lives in `tools/pdp-metadata/`.
- The storefront code is plain JavaScript, mostly browser-side ESM.
- Cypress config files use CommonJS (`require`, `module.exports`).
- `scripts/__dropins__/` contains built or vendored drop-in artifacts copied from dependencies.
- Prefer editing source files under `blocks/`, `scripts/`, and config files unless the task explicitly requires updating vendored drop-in output.

## Install And Startup

- Install root deps: `npm install`
- Start local storefront dev server: `npm start`
- Root postinstall copies drop-in bundles into `scripts/__dropins__/`.
- If drop-in package versions change, run: `npm run postinstall`
- Install Cypress deps separately when working on tests: `npm --prefix cypress install`

## Build Commands

- Rebuild merged JSON component metadata: `npm run build:json`
- Rebuild only models JSON: `npm run build:json:models`
- Rebuild only definitions JSON: `npm run build:json:definitions`
- Rebuild only filters JSON: `npm run build:json:filters`
- Run the helper metadata tool: `npm --prefix tools/pdp-metadata start`

## Lint Commands

- Lint all JS at repo root: `npm run lint:js`
- Lint all CSS at repo root: `npm run lint:css`
- Lint everything configured at repo root: `npm run lint`
- Auto-fix root JS and CSS issues when possible: `npm run lint:fix`
- Lint one JS file from repo root: `npx eslint path/to/file.js`
- Lint multiple JS files from repo root: `npx eslint blocks/foo/foo.js scripts/bar.js`
- Lint one CSS file from repo root: `npx stylelint blocks/foo/foo.css`
- Lint Cypress tests: `npm --prefix cypress run lint`
- Lint one Cypress file: `npx eslint cypress/src/tests/b2b/verifyB2BQuickOrder.spec.js --config cypress/eslint.config.js`

## Test Commands

- There is no repo-level unit test runner configured in `package.json`.
- The main automated test suite in this repo is Cypress E2E under `cypress/`.
- Cypress default B2C run: `npm --prefix cypress run cypress:run`
- Cypress SaaS run: `npm --prefix cypress run cypress:saas:run`
- Cypress B2B SaaS run: `npm --prefix cypress run cypress:b2b:saas:run`
- Cypress PaaS open mode: `npm --prefix cypress run cypress:open`
- Cypress SaaS open mode: `npm --prefix cypress run cypress:saas:open`
- Cypress B2B SaaS open mode: `npm --prefix cypress run cypress:b2b:saas:open`
- Percy snapshot run: `npm --prefix cypress run cypress:percy`

## Single-Test Commands

- Run one B2B spec: `npm --prefix cypress run cypress:b2b:saas:run -- --spec "src/tests/b2b/verifyB2BQuickOrder.spec.js"`
- Run one B2C spec on SaaS config: `npm --prefix cypress run cypress:saas:run -- --spec "src/tests/b2c/verifyProductSearch.spec.js"`
- Run one B2C spec on PaaS config: `npm --prefix cypress run cypress:run -- --spec "src/tests/b2c/verifyGuestUserCheckout.spec.js"`
- Run one spec in headed mode for debugging: `npm --prefix cypress exec cypress run --headed --browser chrome --config-file cypress.b2b.saas.config.js --spec "src/tests/b2b/verifyB2BQuickOrder.spec.js"`
- Open Cypress UI for interactive single-spec execution: `npm --prefix cypress run cypress:b2b:saas:open`
- The repo includes `@cypress/grep`; use grep filters when needed, for example: `npm --prefix cypress exec cypress run --config-file cypress.b2b.saas.config.js --env grepTags=@B2BSaas,grep="multiple SKU workflow"`

## Test Environment Notes

- Cypress base URL defaults to `http://localhost:3000/`.
- Start the local server before running Cypress against local pages.
- B2B tests also require Commerce B2B features and related backend configuration.
- `cypress/README.md` documents required env vars for Admin REST interactions.
- Environment-specific data such as GraphQL endpoints, gift cards, and test products live in the Cypress config files.

## Pre-Commit Behavior

- Husky is enabled through `npm prepare`.
- Pre-commit runs `.husky/check-block-readme.js`.
- If you change `blocks/<block-name>/` source files and that block has no `README.md`, commit is blocked.
- Pre-commit also runs `.husky/pre-commit.mjs`.
- If staged files include partial model JSON files matching `(^|/)_.*.json`, the hook runs `npm run build:json` and stages `component-models.json`, `component-definition.json`, and `component-filters.json` automatically.
- When touching `blocks/`, expect README expectations to matter.

## JavaScript Style

- Follow ESLint from `.eslintrc.js`, which extends `airbnb-base`.
- Use ESM imports/exports in storefront code.
- Include `.js` extensions in local imports. This is enforced.
- Prefer named exports when useful; default-only export style is not required.
- Relative package imports are allowed in this repo.
- Circular imports are tolerated where existing code already relies on them, but do not introduce them casually.
- Use Unix line endings.
- Prefer `const`; use `let` only when reassignment is necessary.
- Semicolons are standard and used consistently.
- Use single quotes.
- Keep trailing commas where the existing style uses them.
- Use async/await for asynchronous flows; attach `.catch(...)` when handling promises inline.
- Use optional chaining and nullish coalescing when they simplify defensive code.
- Use early returns to keep control flow flat.

## Imports

- Group imports by purpose when the file is large: drop-ins, local modules, initializers.
- In practice, repo files often separate groups with short comments; preserve that style when already present.
- Put third-party or `@dropins/*` imports before local relative imports.
- Keep side-effect imports such as initializer bootstraps near other imports, usually at the end of the import section.
- Do not omit `.js` from relative imports.

## Naming Conventions

- Use `camelCase` for variables, functions, and object properties.
- Use `UPPER_SNAKE_CASE` for module-level constants that act like configuration.
- Use descriptive exported function names such as `decorate`, `initializeRequisitionList`, or `fetchPlaceholders`.
- DOM element references often use a `$` prefix, for example `$pagination` or `$wishlistToggle`; follow the local file style.
- CSS classes generally use BEM-like naming with `__` and `--`.
- Event names are slash-delimited strings like `search/result` or `aem/cart-loaded`; match existing namespaces.

## Types And Data Shape

- There is no TypeScript in the main storefront app.
- Do not introduce TypeScript unless the task explicitly requires it.
- Be explicit about expected object shapes via naming, destructuring, defaults, and JSDoc when helpful.
- Preserve existing public payload shapes for events, GraphQL transforms, and block config.
- Validate required inputs before making network or rendering calls.

## Error Handling

- Throw `Error` objects for invalid required inputs or malformed API responses.
- Surface actionable messages, for example `Quote UID is required` or `Failed to transform quote data`.
- Use `console.warn`, `console.error`, `console.info`, or `console.debug` when logging is needed.
- `console.log` is disallowed by ESLint outside explicit exceptions.
- Prefer catching errors close to user-facing flows so the UI can degrade gracefully.
- When a failure is non-fatal, warn and continue instead of crashing initialization.
- When handling GraphQL responses, check `errors` arrays and missing `data` branches explicitly.

## DOM And UI Patterns

- Most storefront features decorate server-rendered blocks.
- `export default async function decorate(block)` is a common block entrypoint pattern.
- Construct DOM with `document.createElement`, `createContextualFragment`, and drop-in render helpers.
- Preserve accessibility-related attributes and button/link semantics when modifying UI code.
- Keep placeholder loading, event wiring, and drop-in rendering close to block setup code.

## CSS Style

- Root CSS linting uses `stylelint-config-standard`.
- Keep selectors scoped to the block where possible.
- Reuse the existing BEM-like class naming style.
- Prefer CSS custom properties already used in the repo, such as spacing, color, and typography tokens.
- Avoid broad global selectors unless the existing block pattern requires them.

## Cypress Test Style

- Cypress tests are plain JavaScript and use Mocha-style `describe`/`it`.
- Reuse selectors from shared `fields` modules when available.
- Keep test constants near the top of the spec.
- Use tags such as `@B2BSaas`, `@skipSaas`, and `@skipPaas` consistently.
- The Cypress ESLint config only warns on unsafe chaining, unnecessary waiting, and assigning return values; warnings still deserve cleanup when practical.
- Prefer stable selectors like `data-testid` over brittle visual selectors when both exist.

## Agent Guidance

- Before editing `scripts/__dropins__/`, confirm the change should not instead be made in source or by updating a dependency and rerunning `postinstall`.
- If you change block source files, check whether the corresponding block README should also change.
- If you touch partial model JSON files, expect generated JSON outputs to change as part of the same work.
- Favor minimal, local changes that match existing patterns over broad refactors.
