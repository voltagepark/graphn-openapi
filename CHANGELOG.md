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

## [0.1.0] - 2026-04-24

### Added

- Initial public release of the GraphN OpenAPI 3.1 specification.
- Custom model lifecycle: `createCustomModel`, `listCustomModels`,
  `getCustomModel`, `refreshCustomModel`, `wakeCustomModel`,
  `deleteCustomModel`, `validateCustomModel`, `getSupportedArchitectures`,
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
