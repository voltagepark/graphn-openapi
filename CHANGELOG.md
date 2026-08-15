# Changelog

All notable changes to the GraphN OpenAPI specification are documented in this
file. The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and the spec adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

Entries here are mirrored verbatim into the public
[`voltagepark/graphn-openapi`](https://github.com/voltagepark/graphn-openapi)
repo by `.github/workflows/release-openapi.yml` on each `openapi/v*` tag.

## How to release

One PR. In the same PR that changes the spec:

1. Edit `openapi.yaml`.
2. Bump `info.version` to the new `X.Y.Z`.
3. Replace the `## [Unreleased]` heading below with `## [X.Y.Z] - YYYY-MM-DD`
   (and add a fresh empty `## [Unreleased]` above it for the next round).

Merge to `main`. The `auto-tag` job in `release-openapi.yml` reads the new
`info.version`, creates the matching `openapi/vX.Y.Z` tag, and the tag-push
re-fires the workflow to mirror the spec to the public repo + cut a GitHub
release. agent-foundry's `/api` Redoc page picks the new spec up
automatically (it fetches `voltagepark/graphn-openapi@latest` via jsdelivr).

No manual tag, no second PR, no `web/public/openapi.yaml` to keep in sync.

## [Unreleased]

## [0.7.0] - 2026-08-15

### Added

The public spec now covers the full CLI-reachable customer API, not just
custom-model import and inference. Generated clients in any language can
target the agent platform from this version.

- **Agents, functions, MCP servers, workflows** — CRUD, publish, run/test/dry-run, bundle, versions.
- **Executions** — list/get. `exec_` / `test_` ids on the control plane; UUID operation ids on the gateway.
- **Triggers** — workspace-scoped CRUD plus nested `/workflows/{id}/triggers`.
- **Knowledge bases** — CRUD, documents, search, batch, async ingest, embedding models. Paths are the customer CP form `/v1/{workspaceId}/knowledgebases/...`.
- **Imported models** — full workspace CRUD (discover/test remain on the inference host).
- **Organizations, workspaces, members, invitations, API keys**.
- **Blueprints** — catalog list/get and workspace deploy.
- **Connectors** — `GET /v1/connectors/catalog`.
- **Embed** — `POST /v1/{workspaceId}/embed`.
- **Storages (REST)** — `/v1/{workspaceId}/storages` overlay used by the UI/SDK.
- **Object storage (S3 overlay)** — `https://storage.graphn.ai/{bucket}/{key}` with MPU and presign query dispatch.
- **Gateway batch and async** — submit, poll, items, JSONL output, cancel.

### Changed

- `info.description` now describes the agent platform and four hosts
  (control plane, inference, gateway, storage).
- Per-operation `servers` overrides added for gateway and storage operations.

## [0.6.0] - 2026-08-11

### Added

- `POST /v1/{orgId}/billing/tour-grant` for claiming the one-time cardless
  onboarding-tour credit grant.

## [0.5.1] - 2026-08-09

### Added

- `invoiceDocumentUrl` on `Invoice` for non-PDF statement artifacts (currently
  HTML). This keeps `invoicePdf` literal so clients that assume PDF MIME /
  filenames are never handed HTML.

## [0.5.0] - 2026-08-08

### Added

- `GET /v1/{orgId}/billing/invoices/{periodStart}` returning one statement's
  detail.
- `periodStart`, `periodEnd`, `openingCents`, `usageCents`, `creditsCents` and
  `closingCents` on `Invoice`, so a client can show how a period reached its
  closing balance instead of a single amount due.
- `emptyReason` (`no_billing_history` | `statements_not_generated_yet`) on the
  list response, which separates an organization that has never transacted from
  one whose statements have not been cut yet. Both previously returned an empty
  list, so a client could not tell a new account from a pending snapshot.

### Changed

- **Breaking:** `Invoice.status` is now an enum of `closed` | `current`. It
  previously carried Stripe's invoice status verbatim.
- **Breaking:** `hostedInvoiceUrl` and `invoicePdf` are no longer required.
  Statements are assembled from the billing ledger rather than Stripe, so there
  is no hosted page or PDF for a statement to link. Both remain as optional
  properties for Stripe-era rows.
- The `billing/invoices` endpoints are documented as statements. Billing is
  prepaid credits, so a closed period is a usage statement, not a payable
  invoice. The paths keep the `invoices` name so client routing is unaffected.

## [0.4.0] - 2026-07-31

### Added

- `allowNegativeBalance` and `negativeLimitCents` on `BillingEntitlements`, so a
  client can tell an organization on invoice terms apart from one that is
  overdrawn and blocked.

## [0.3.0] - 2026-07-24

### Added

- Organization billing account, payment-method, invoice, entitlement,
  free-grant, usage, and top-up endpoints.
- Admin billing state, override, and credit-grant endpoints, including explicit
  billing-account existence and concrete OpenMeter grant IDs.

## [0.2.2] - 2026-05-26

### Added

- `ValidateModelRequest.weight_source` (`huggingface` default,
  `s3_assume_role` opt-in) plus `s3_url` / `s3_role_arn` /
  `s3_external_id` siblings. The `s3_assume_role` validate path
  unlocks pre-deploy architecture checks for BYOM imports without
  forcing a multi-minute download Job to surface a "this model
  isn't supported" error. AF chains the same two-hop AssumeRole
  the smart-loader uses and probes `<s3_url>config.json` directly.
  `s3_presigned` is intentionally not accepted: presigned URLs
  deliver a single archive object and architecture detection
  requires the full extract.
- `ValidateModelRequest.base_model_id`: LoRA base override / hint
  on the validate path, mirroring `CustomModelCreate.base_model_id`.
  Pre-existing in AF's Pydantic schema; codifying it here closes the
  contract drift the AF-side OpenAPI contract test was xfailing on.

### Changed

- `CustomModelCreate.s3_url` semantics for `weight_source=s3_assume_role`:
  the AF smart-loader's AssumeRole arm now uses `aws s3 sync`
  (raw-directory) instead of `aws s3 cp + tar/pigz/xz/zip` extract.
  The spec doesn't formally restrict the URL shape (still a free-form
  string up to 2048 chars) but AF returns 422 when the URL doesn't
  end with `/` on this weight source. Customers should point at the
  prefix containing `config.json` + safetensors in HuggingFace
  layout. tar.gz / zip uploads are unsupported on this path going
  forward.

## [0.2.1] - 2026-05-21

### Added

- `CustomModelCreate.base_model_id`: caller-supplied base for LoRA
  imports. For `weight_source=s3_*` this is the only way to classify
  the bundle as a LoRA adapter at create-time (omitting it routes
  the request through the base path and surfaces an actionable
  `failed` status if the bundle turns out to be a LoRA adapter). For
  `weight_source=huggingface` it **overrides**
  `adapter_config.json::base_model_name_or_path` from the upstream
  repo, which makes adapters trained against a local filesystem
  path (e.g. `C:/users/.../base`) usable without editing the
  adapter config. Must be in the platform's LoRA base allowlist.

### Changed

- `CustomModel.artifact_type` description rewritten to drop the
  "lazy detection on the roadmap" caveat; classification is eager
  for both `huggingface` (via `adapter_config.json` probe) and
  `s3_*` (via the new `base_model_id` field on `CustomModelCreate`).
- `CustomModel.base_model_id` description rewritten to call out
  the new override semantics for HuggingFace imports.

## [0.2.0] - 2026-05-20

### Removed (BREAKING)

- `WeightSource` enum loses the `lora_huggingface`, `lora_s3_presigned`,
  and `lora_s3_assume_role` values. Callers now always pass one of the
  three base values (`huggingface | s3_presigned | s3_assume_role`).
  Whether the import is a base checkpoint or a LoRA adapter is no
  longer declared on the wire; it's derived (eager for HuggingFace via
  `adapter_config.json`, lazy for S3) and surfaced on the model record
  via the new `artifact_type` field. Existing records with a legacy
  `weight_source` are silently translated on read.

### Added

- `CustomModel.artifact_type` (`base | lora | null`): explicit
  classification of the import. `null` means the platform has not yet
  classified the bundle (S3 imports until lazy detection lands).
- `CustomModel.base_model_id`, `CustomModel.lora_adapter_name`,
  `CustomModel.lora_rank`: previously implicit, now first-class
  read-only fields on the model record.

## [0.1.5] - 2026-05-18

### Added

- `ValidateModelResponse` gains three fields so the import UX can
  collapse its two-row "Base vs LoRA" picker into a single Source
  row: `artifact_type` (`base` | `lora`), `detected_base_model_id`,
  and `lora_rank`. AF sets them after probing `adapter_config.json`
  server-side; the `base` default means existing SDK callers see no
  behaviour change.
- `PATCH /v1/{workspaceId}/custom-models/{modelId}` (`updateCustomModel`)
  for in-place updates to a small whitelist of post-create-mutable
  fields: `name`, `min_replicas`, `max_replicas`, `cooldown_seconds`.
  The bounds fields are applied to the live KServe `InferenceService`
  with no rolling restart -- useful for pinning a hot model online
  before a traffic spike or widening the autoscaling ceiling.
- `CustomModelUpdate` schema for the new PATCH payload.

## [0.1.4] - 2026-05-08

Patch release. Re-introduces the `getSupportedArchitectures` operation
that was removed in `0.1.1` (this time backed by a real control-plane
proxy to AF), and adds the `model_size_gb` hint that AF's
`ValidateModelRequest` already accepts.

### Added

- `getSupportedArchitectures` (`GET /v1/{workspaceId}/custom-models/supported-architectures`)
  and the `SupportedArchitectures` / `ArchitectureInfo` component
  schemas. The control plane proxies to AF's `/v1/custom-models/supported-architectures`
  (registered in `app/custom_models/router.py`), so the endpoint now
  returns the real architecture catalog instead of 404.

  Re-adding the operation that `0.1.1` removed is a non-breaking
  superset for clients — a 404 path becomes a documented 200, and no
  existing consumer was relying on the absence.

- `ValidateModelRequest.model_size_gb` (integer, optional, 1–500). When
  the caller already knows the on-disk weights size, supplying this hint
  lets the platform size the model-weights PVC from the hint instead of
  waiting for a HuggingFace head-bytes probe — the probe still runs and
  wins when AF can resolve a real number, but providing this avoids the
  validate-time stall on very large models where the probe is slow.

  AF has accepted this field on the request since the `model_size_gb`
  PVC sizing landed (see `app/custom_models/schemas.py`); the spec now
  reflects what the live API has been silently honoring.

## [0.1.3] - 2026-05-08

Patch release. Tightens `CustomModelCreate.s3_role_arn` to require
the role name to start with `graphn-byom-`, matching the IAM scoping
the platform now enforces.

### Changed

- `CustomModelCreate.s3_role_arn` `pattern` tightened from
  `^arn:aws:iam::\d{12}:role/.+$` to require the role name (the
  segment after `:role/`) to start with `graphn-byom-` (e.g.
  `arn:aws:iam::123456789012:role/graphn-byom-s3-reader`). GraphN's
  platform IAM policy is now scoped to that prefix as a defense-in-
  depth boundary, and the customer-facing CloudFormation template
  enforces the same constraint at stack-create time, so callers using
  the Quick Create flow are unaffected. Manual integrations that
  named their role something else will need to recreate it under the
  `graphn-byom-` prefix; otherwise the API now returns 422 at
  create-model time instead of the deploy failing later with
  `AccessDenied` from STS.

## [0.1.2] - 2026-04-25

Patch release. Tightens `huggingface_model_id` requirement to all
weight sources (matching what the web UI has always sent), updates the
contact link, and scrubs `vLLM` / `--served-model-name` references
from customer-facing descriptions in favor of engine-agnostic phrasing.

### Fixed

- `info.contact.url` now points at the public OpenAPI repo's issue
  tracker (`github.com/voltagepark/graphn-openapi/issues`) instead of
  the placeholder `https://graphn.ai/support`, which never resolved to
  a real page (the SPA's catch-all route returned 200 for it). Same
  class of fix as the `docs.graphn.ai` removal in v0.1.1.

### Changed

- `CustomModelCreate.huggingface_model_id` is now **required for all
  `weight_source` values**, not just `huggingface`. It is the
  canonical model identifier the inference endpoint advertises and
  the value clients pass in `model` for chat completions, so without
  it S3 imports produced models that could never be addressed via
  inference (404). The web UI has always required this as the
  "Model ID" field; the spec and the control-plane validator now
  match that contract.

  This is a tightening of the schema, but matches the actual
  successful-deployment shape used by the web UI today, so existing
  S3 deployments that worked are unaffected. Callers that
  previously omitted `huggingface_model_id` for S3 sources will now
  receive a 422 at create time instead of silently producing an
  unreachable model.

- `s3_url` and `s3_role_arn` descriptions updated to call out that
  their conditional requirement is enforced by the control plane
  (and the Python SDK) rather than encoded via JSON Schema
  `if/then`, for OAS-3.0-tooling compatibility.

- `VALIDATION_ERROR` example updated to reflect the broadened
  `huggingface_model_id` requirement.

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
