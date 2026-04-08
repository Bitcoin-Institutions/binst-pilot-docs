# Inscription Schema

The `binst` metaprotocol defines a formal JSON schema for Ordinals inscriptions.

## Envelope Format

Every BINST inscription uses the Ordinals envelope:

- **Content type** (tag 1) = `application/json`
- **Metaprotocol** (tag 7) = `binst`
- **Parent** (tag 3) = parent inscription ID (provenance chain)
- **Metadata** (tag 5) = optional CBOR-encoded metadata
- **Body** = JSON matching the schema below

## Entity Types

| Type | Parent requirement | Purpose |
|---|---|---|
| `institution` | None (root of its tree) | Institution identity and metadata |
| `process_template` | Institution inscription | Immutable process blueprint |
| `process_instance` | Process template inscription | Running execution of a template |
| `step_execution` | Process instance inscription | Record of a step execution (optional) |

## Provenance Hierarchy

```text
institution (root — no parent required)
 └─ process_template (child of institution)
     └─ process_instance (child of template)
         └─ step_execution (child of instance)
```

## Schema Version

`"v": 0` — pilot / testnet4. Breaking changes increment the version.

## Example: Institution

```json
{
  "v": 0,
  "type": "institution",
  "name": "Acme Financial",
  "admin": "a3f4b2c1d5e6f7890123456789abcdef0123456789abcdef0123456789abcdef",
  "citrea_contract": "0x1234...5678",
  "membership_rune": "ACME•MEMBER",
  "description": "Acme Financial pilot institution",
  "website": "https://acme.example"
}
```

## Example: Process Template

```json
{
  "v": 0,
  "type": "process_template",
  "name": "KYC Onboarding",
  "institution_inscription_id": "abc123...i0",
  "steps": [
    { "name": "Submit Documents", "action_type": "upload" },
    { "name": "Review", "action_type": "approval" },
    { "name": "Final Approval", "action_type": "approval" }
  ]
}
```

## Example: Process Instance

```json
{
  "v": 0,
  "type": "process_instance",
  "template_inscription_id": "def456...i0",
  "created_by": "a3f4b2c1...",
  "status": "in_progress"
}
```

## Example: Step Execution

```json
{
  "v": 0,
  "type": "step_execution",
  "instance_inscription_id": "ghi789...i0",
  "step_index": 0,
  "actor": "b5e6c7d8...",
  "status": "completed",
  "timestamp": "2026-03-15T14:30:00Z"
}
```

## Validation

The full JSON Schema (2020-12) is available at [`binst-metaprotocol.json`](https://github.com/Bitcoin-Institutions/binst-pilot/blob/main/binst-protocol/schema/binst-metaprotocol.json).

```bash
# Validate with ajv-cli
ajv validate -s binst-metaprotocol.json -d examples/institution.json
```
