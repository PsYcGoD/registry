# Contributing to server.json Schema

This document describes the process for making and releasing changes to the `server.json` schema.

## Making Changes

The schema has a single source of truth: [`schema.ts`](./schema.ts). The committed
artifact [`draft/server.schema.json`](./draft/server.schema.json) is generated from it
and must not be edited by hand.

1. **Edit the source of truth**: Modify [`schema.ts`](./schema.ts) with your schema changes.

2. **Regenerate the schema**: Run `make generate-schema` (or, from this directory,
   `npm run generate`) to update `draft/server.schema.json` from `schema.ts`.

3. **Keep the OpenAPI spec in sync**: The `ServerDetail` component in
   `docs/reference/api/openapi.yaml` documents the same shape for the REST API. CI
   regenerates the schema from `openapi.yaml` too and fails if it diverges from the
   committed `server.schema.json`, so update `openapi.yaml` to match. (Removing this
   duplication — e.g. having `openapi.yaml` reference the generated schema — is a
   reasonable future cleanup.)

4. **Add or update examples**: Example documents under [`examples/`](./examples) are
   validated against the generated schema. `examples/valid/` must validate cleanly and
   `examples/invalid/` must fail at least one constraint. Run `npm run validate` (or
   `make validate`) to check.

5. **Update the changelog**: Add your changes to the "Draft (Unreleased)" section in `CHANGELOG.md`.

6. **Open a PR**: Submit a pull request to this repository for review.

### Local tooling

From this directory (`docs/reference/server-json/`):

```bash
npm ci            # install dev dependencies (tsx, typescript, ajv)
npm run generate  # regenerate draft/server.schema.json from schema.ts
npm run check     # fail if the committed schema is out of date + type-check schema.ts
npm run validate  # validate examples/ against the generated schema
```

The generated JSON deliberately reproduces the previous Go (`encoding/json`) output
byte-for-byte, so adopting this pipeline is a no-op for the schema content itself.

## Releasing Changes

When the draft changes are ready for release:

1. **Update the changelog**: Move changes from "Draft (Unreleased)" to a new dated section (e.g., `## 2025-XX-XX`).

2. **Update the schema URL**: Change the `$id` in the schema and the example URL in `openapi.yaml` from `draft` to the release date (e.g., `2025-XX-XX`).

3. **Merge the PR**: Get approval and merge the changes to main.

4. **Publish to static hosting**: Open a PR on [modelcontextprotocol/static](https://github.com/modelcontextprotocol/static/tree/main/schemas) to add the new versioned schema file. This "locks in" the released schema at its versioned URL.

## Schema Versioning

- **Draft schema**: `https://raw.githubusercontent.com/modelcontextprotocol/registry/main/docs/reference/server-json/draft/server.schema.json` - For in-progress changes, may change without notice.
- **Released schemas**: `https://static.modelcontextprotocol.io/schemas/YYYY-MM-DD/server.schema.json` - Stable, versioned by release date.
