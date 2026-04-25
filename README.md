# GraphN OpenAPI Specification

[![Latest release](https://img.shields.io/github/v/release/voltagepark/graphn-openapi?label=spec)](https://github.com/voltagepark/graphn-openapi/releases)
[![Apache 2.0 licensed clients](https://img.shields.io/badge/clients-Apache%202.0-blue)](#license)

The official OpenAPI 3.1 specification for the [GraphN](https://graphn.ai)
platform. Use this spec to generate strongly-typed client SDKs in any language
that has an OpenAPI generator.

> **Source of truth:** this repo is automatically mirrored from a private
> monorepo on every spec release. Do **not** open PRs against this repo —
> file feature requests and bug reports as
> [issues](https://github.com/voltagepark/graphn-openapi/issues) instead.

## Contents

| Path | Purpose |
| --- | --- |
| `openapi.yaml` | The spec (YAML). Primary artifact. |
| `openapi.json` | Auto-converted JSON form (some generators prefer it). |
| `fixtures/` | Per-operation example request/response payloads. |
| `CHANGELOG.md` | Versioned changelog. |

## Quickstart — generate a client

### TypeScript / JavaScript

```bash
npx @hey-api/openapi-ts \
  --input https://raw.githubusercontent.com/voltagepark/graphn-openapi/main/openapi.yaml \
  --output src/graphn \
  --client fetch
```

### Go

```bash
go install github.com/oapi-codegen/oapi-codegen/v2/cmd/oapi-codegen@latest
oapi-codegen -package graphn \
  -generate types,client \
  https://raw.githubusercontent.com/voltagepark/graphn-openapi/main/openapi.yaml \
  > graphn.gen.go
```

### Java

```bash
docker run --rm -v "${PWD}:/local" openapitools/openapi-generator-cli generate \
  -i https://raw.githubusercontent.com/voltagepark/graphn-openapi/main/openapi.yaml \
  -g java \
  -o /local/graphn-java
```

### Python

You don't need to generate one — install the official SDK:

```bash
pip install graphn
```

Source: [voltagepark/graphn-sdk-python](https://github.com/voltagepark/graphn-sdk-python)

## Hosts

The API spans two hosts; per-operation `servers` overrides in the spec route
generated clients automatically:

| Host | Purpose |
| --- | --- |
| `https://cp.graphn.ai` | Control plane — workspace-scoped CRUD (custom models, secrets) |
| `https://model.graphn.ai` | OpenAI-compatible inference (chat, models, TTS) |

## Authentication

All endpoints require a workspace-scoped API key:

```
Authorization: Bearer gn_...
```

Create keys in the [GraphN dashboard](https://app.graphn.ai/settings/api-keys).

## Fixtures

Each operation in `fixtures/<tag>/` has canonical example payloads (extracted
from the official Python SDK's test suite, so they're guaranteed realistic):

```
fixtures/custom-models/createCustomModel.request.json
fixtures/custom-models/createCustomModel.response.201.json
fixtures/custom-models/getCustomModel.response.404.json
fixtures/chat/chatCompletions.streaming.txt
...
```

Use them as golden inputs when integration-testing a generated client.

## Versioning

The spec follows [Semantic Versioning](https://semver.org/). Breaking changes
bump the major; additive changes bump the minor; doc-only fixes bump the patch.

## License

The OpenAPI spec itself is **Proprietary** to Voltage Park, but you are
explicitly licensed to:

- Generate client SDKs from it (in any language).
- Redistribute generated SDKs under the license of your choice.
- Vendor `openapi.yaml` into your project.

Generated SDKs and fixture payloads are not derivative works of the spec for
licensing purposes.

## Support

- Docs: <https://graphn.ai/docs>
- API explorer: <https://graphn.ai/api>
- Issues: <https://github.com/voltagepark/graphn-openapi/issues>
- Email: <support@graphn.ai>
