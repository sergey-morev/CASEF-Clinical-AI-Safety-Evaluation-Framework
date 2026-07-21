# CASEF v0.6.1 common executable schema definitions

## Scope

This directory contains the first executable CASEF schema artifact: reusable common scalar and hash definition libraries using JSON Schema Draft 2020-12.

The definitions are subordinate to [`docs/canonical_serialization_contract.md`](../../../docs/canonical_serialization_contract.md). They implement lexical and structural rules only and own no canonical record content, evidence eligibility, gate semantics, or qualification authority.

## Files

- `scalars.schema.json` defines the shared identifier, authored-version, record-lineage version, canonical UTC timestamp, absolute-URI, and JSON-Pointer lexical forms.
- `hashes.schema.json` defines the SHA-256 digest component, canonical CASEF content-hash representation, and input-hash domain vocabulary.
- `tests/scalars_cases.json` contains machine-readable valid and invalid values for each scalar definition.
- `tests/hashes_cases.json` contains machine-readable valid and invalid values for each hash definition.

## Definition-container behavior

Each schema root is a closed empty object used as a definition container. Consumers reference exact `$defs` fragments. Validating `{}` against a root has no record-level meaning, and these files are not canonical record schemas.

## Development and release state

The schemas use their future immutable release-tag `$id` values. They remain development artifacts until the immutable `v0.6.1` Git tag exists. No released-schema claim is made, and no schema catalog exists.

## Validation boundary

Lexical schema success does not prove semantic correctness. In particular:

- the timestamp regular expression does not establish full calendar validity or settle leap-second semantics;
- URI validity does not prove locator correctness, content integrity, or authority;
- hash-pattern validity does not compute or verify a digest; and
- no RFC 8785 canonicalization or SHA-256 computation implementation is included.

## Test vectors

Each test-vector file names the exact schema `$id` and provides one case group per referenced `$defs` fragment. Every value in `valid` must validate against that fragment. Every value in `invalid` must produce at least one validation error. URI vectors must be checked with Draft 2020-12 URI format validation enabled.

The vectors are lexical and structural test data, not canonical records or evidence.

## Next dependency layer

Record, artifact, and versioned-contract references and role-bound actor identities are intentionally deferred to a separate reviewed task.
