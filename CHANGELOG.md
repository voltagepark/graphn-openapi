# Changelog

All notable changes to the GraphN OpenAPI specification are documented in this
file. The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and the spec adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

Entries here are mirrored verbatim into the public
[`voltagepark/graphn-openapi`](https://github.com/voltagepark/graphn-openapi)
repo by `.github/workflows/release-openapi.yml` on each `openapi/v*` tag.

When you change the spec, add an entry under **[Unreleased]**. The release
workflow promotes that section to the new version on tag push.

## [Unreleased]

## [0.1.1] - 2026-04-25

Patch release. No new operations; small spec hygiene + docs fixes.

### Removed

- `getSupportedArchitectures` (`GET /v1/{workspaceId}/custom-models/supported-architectures`)
  and the `SupportedArchitectures` / `ArchitectureInfo` component schemas.
  This operation was declared in v0.1.0 but never implemented in the
  control plane (returned 404). Removed before any client could rely
  on it.

### Fixed

- Public mirror's `README.md` now points to the real docs host
  (`https://graphn.ai/docs` and `https://graphn.ai/api`) instead of
  the placeholder `docs.graphn.ai` subdomain that was never set up.

## [0.1.0] - 2026-04-24

### Added

- Initial public release of the GraphN OpenAPI 3.1 specification.
- Custom model lifecycle: `createCustomModel`, `listCustomModels`,
  `getCustomModel`, `refreshCustomModel`, `wakeCustomModel`,
  `deleteCustomModel`, `validateCustomModel`,
  `getCustomModelAccess`, `getGpuHours`.
- Workspace secrets: `createSecret`, `listSecrets`, `getSecret`,
  `updateSecret`, `deleteSecret`.
- OpenAI-compatible inference (`model.graphn.ai`):
  `createChatCompletion`, `listModels`, `createSpeech`, `listVoices`.
- Imported (BYO) model discovery: `listImportedModels`, `testImportedModel`.
- Per-operation `servers` overrides so generated clients route control-plane
  vs inference traffic correctly out of the box.
- Bearer (`gn_...`) API key auth scheme.
- Standard error envelope: `{ "code": "...", "message": "..." }`.
