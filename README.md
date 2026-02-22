# Evidence Chain Module – Reusable Timestamp & Hash Anchoring

## Overview

This module defines a reusable evidence-chain specification for establishing immutable creation and modification timestamps via cryptographic hashing and blockchain anchoring. It is designed to be embedded into YAML frontmatter for IP-sensitive artifacts. The module is maintained by Nova Itera Research Group, LLC at version 1.1.0 and is marked stable. It supports configurable blockchain parameters, hash algorithms, and a canonical payload structure to ensure reproducible hashes across verification parties.

## What this contains

- Module descriptor (`evidence_chain_module`) with `name`, `module_id`, `version`, `description`, `maintainer`, and `status`.
- `applies_to` list indicating valid embedding targets: `page_metadata` and `artifact_metadata`.
- `params` block defining overridable defaults: `chain` (bitcoin), `network` (mainnet), `content_hash_algo` (sha256), and `content_fields`.
- `evidence_chain` schema including:
  - `validation_methods`: Git commit timestamps, IPFS content addressing, blockchain timestamp anchoring, Notion API snapshot verification.
  - `page_metadata`: fields for `page_id`, `created_timestamp_unix_ms`, `last_modified_timestamp_unix_ms`.
  - `blockchain_timestamp`: anchoring state machine fields (`status`, `chain`, `network`, `content_hash_algo`, `content_hash`, `tx_id`, `anchored_unix_ms`, `anchor_reference`, `error_message`).
  - `provenance.canonical_payload_spec`: description of canonical JSON payload, field list, and example payload structure.
  - `verification_process.steps`: six-step anchoring workflow.
  - `verification_instructions`: ordered five-step procedure for third-party independent validation.

## Usage

- Embed the `evidence_chain_module` and `evidence_chain` blocks into YAML frontmatter or equivalent configuration for any IP-sensitive page or artifact.
- Override `params.chain`, `params.network`, `params.content_hash_algo`, and `params.content_fields` in consuming specifications as required.
- Populate `page_metadata` fields (`page_id`, `created_timestamp_unix_ms`, `last_modified_timestamp_unix_ms`) from an authoritative metadata source at artifact creation time.
- Follow `verification_process.steps` as the operational anchoring runbook.
- Distribute `verification_instructions` to third parties for independent validation.

## Conventions / Schemas

```yaml
evidence_chain_module:
  name: "Evidence Chain – Timestamp & Hash Anchoring"
  module_id: "evidence_chain_v1"
  version: "1.1.0"
  description: >
    Reusable evidence-chain module for establishing immutable creation and
    modification timestamps via cryptographic hashing and blockchain anchoring.
    Designed to be embedded into YAML frontmatter for IP-sensitive artifacts.
  maintainer: "Nova Itera Research Group, LLC"
  status: "stable"

applies_to:
  - "page_metadata"
  - "artifact_metadata"

params:
  chain: "bitcoin"             # default chain (e.g., bitcoin, ethereum)
  network: "mainnet"           # default network (e.g., mainnet, testnet)
  content_hash_algo: "sha256"  # supported: sha256, blake2b, etc.
  content_fields:              # fields that must be included in hash payload
    - "page_id"
    - "created_timestamp_unix_ms"
    - "last_modified_timestamp_unix_ms"

evidence_chain:
  validation_methods:
    - "Git commit timestamps"
    - "IPFS content addressing"
    - "Blockchain timestamp anchoring"
    - "Notion API snapshot verification"

  page_metadata:
    page_id: null                      # to be set by calling system
    created_timestamp_unix_ms: null    # canonical origin time
    last_modified_timestamp_unix_ms: null

  blockchain_timestamp:
    status: "pending"                  # "pending" | "anchored" | "failed"
    chain: "<param:chain>"             # resolved from evidence_chain_module.params.chain
    network: "<param:network>"         # resolved from evidence_chain_module.params.network
    content_hash_algo: "<param:content_hash_algo>"
    content_hash: null                 # hash of canonical payload; set by anchoring tool
    tx_id: null                        # blockchain transaction ID
    anchored_unix_ms: null             # time of anchoring on chain
    anchor_reference: null             # block height or explorer URL
    error_message: null                # optional failure reason

  provenance:
    canonical_payload_spec:
      description: >
        Canonical JSON payload used to derive content_hash. Order and
        serialization must be stable to ensure reproducible hashes.
      fields: "<param:content_fields>"
      example_payload_structure:
        page_id: "NOTION_PAGE_ID"
        created_timestamp_unix_ms: 1738057800000
        last_modified_timestamp_unix_ms: 1738057800000

  verification_process:
    steps:
      - "Capture initial creation and last-modified timestamps from authoritative metadata source"
      - "Construct canonical payload using configured content_fields"
      - "Compute content hash with content_hash_algo"
      - "Submit hash to blockchain anchoring service on configured chain/network"
      - "Record tx_id, anchored_unix_ms, and anchor_reference"
      - "Expose verification instructions to third parties for independent validation"

  verification_instructions: |
    1. Retrieve the original artifact and reconstruct the canonical payload using the
       agreed content_fields and authoritative metadata source.
    2. Compute the hash using the specified content_hash_algo.
    3. Compare the computed hash to content_hash stored in this evidence_chain.
    4. Look up tx_id on the specified chain/network and confirm:
       - The on-chain hash matches content_hash
       - The block timestamp is consistent with claimed creation time
    5. If both checks pass, the artifact's existence and integrity at or before
       the block timestamp is cryptographically supported.
```

## Examples

Example canonical payload structure (used as input to the hash function):

```yaml
provenance:
  canonical_payload_spec:
    example_payload_structure:
      page_id: "NOTION_PAGE_ID"
      created_timestamp_unix_ms: 1738057800000
      last_modified_timestamp_unix_ms: 1738057800000
```

`blockchain_timestamp.status` valid values: `"pending"` | `"anchored"` | `"failed"`

## Provenance

This README is generated from a Notion page whose body constitutes the sole source of truth for this module. The Notion page title is "Evidence Chain Module – Reusable Timestamp & Hash Anchoring". No content in this README has been invented outside of that page body.
