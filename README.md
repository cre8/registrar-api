# Registrar API Proposal

This repository contains a proposal and working OpenAPI contract for a public, read-only Wallet-Relying Party Registrar API.

The design goal is transparency and monitoring based on certificate-derived public data.

## What is in this repository

- `registrar-public-read-api.openapi.yaml`
  - The generated OpenAPI 3.1 specification (main deliverable).
- `registrar-api-agent-instructions.md`
  - The detailed concept and requirements used to shape the OpenAPI contract.
- `sources/`
  - Reference source material used as inputs.
- `docker-compose.yml`
  - Local Swagger UI setup to preview the OpenAPI file.
- `docs/index.html`
  - Static Swagger UI entry page used for GitHub Pages publishing.
- `.github/workflows/pages-swagger.yml`
  - CI workflow that validates the OpenAPI spec and deploys Swagger UI to GitHub Pages.

## Core design choices

- Read-only public API surface.
- Certificate-derived monitor model.
- Intended uses as first-class resources.
- Claims are always handled in credential scope (not as standalone endpoint resources).
- Pagination and point-in-time reconstruction support.

## Quick start: preview in Swagger UI

From the repository root:

1. Start Swagger UI

   `docker compose up -d`

2. Open the UI

   http://localhost:8081

3. Stop the UI

   `docker compose down`

Swagger UI serves `registrar-public-read-api.openapi.yaml` via the volume mount in `docker-compose.yml`.

## GitHub Pages deployment

This repository includes a GitHub Actions workflow that publishes Swagger UI to GitHub Pages.

Workflow file: `.github/workflows/pages-swagger.yml`

What it does:

1. Validates `registrar-public-read-api.openapi.yaml` with `swagger-cli`.
2. Builds a static site artifact containing:
  - `docs/index.html`
  - `registrar-public-read-api.openapi.yaml` (as `openapi.yaml`)
3. Deploys the artifact to GitHub Pages on pushes to `main`.

Expected URL pattern after Pages is enabled:

`https://<owner>.github.io/<repo>/`

For this repository, that will be:

`https://cre8.github.io/registrar-api/`

## How to work with this repository

### 1) Update requirements first

Edit `registrar-api-agent-instructions.md` when requirements change.

Examples:

- endpoint additions/removals
- filter semantics
- lifecycle and evidence expectations

### 2) Keep the OpenAPI contract aligned

Apply matching updates to `registrar-public-read-api.openapi.yaml`.

When changing behavior, update:

- endpoint descriptions
- query parameters
- response schemas
- examples

### 3) Preview after each change

Use Swagger UI locally to verify:

- endpoint discoverability
- schema rendering
- examples and descriptions

### 4) Keep consistency rules intact

- No write operations in this version.
- Public data only (no internal draft/administrative fields).
- Claims remain nested under credential context.
- Derived monitor entities should link to evidence endpoints.

## Suggested review checklist

Before finalizing changes, confirm:

- OpenAPI file still parses cleanly.
- Endpoint set matches instructions.
- Filters listed in instructions exist in OpenAPI.
- Examples reflect current semantics.
- Docker Swagger preview opens without schema warnings.

## Notes

- Server URLs and some metadata values are placeholders in the proposal.
- The API can later be extended with a separate administrative/write API.
