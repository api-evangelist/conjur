---
name: Store, read, and version secrets
description: Write a new secret value to a Conjur variable and read current or batched values back.
api: openapi/openapi.yml
operations: [createSecret, getSecret, getSecrets]
---

# Store, read, and version secrets

Use this flow to set and retrieve secret values on Conjur variable resources.

## Preconditions
- You hold a valid Conjur access token (see conjur-authenticate-and-fetch-secret).
- The target variable resource already exists (declared via policy — see conjur-load-policy).

## Steps
1. **Set a secret value** — `createSecret` (`POST /secrets/{account}/{kind}/{identifier}`) with the value in the request body. Each call stores a NEW version; Conjur retains the last 20 versions.
2. **Read the current value** — `getSecret` (`GET /secrets/{account}/{kind}/{identifier}`). Append `?version=N` to read a specific historical version.
3. **Batch read** — `getSecrets` (`GET /secrets`) with a `variable_ids` query listing multiple fully-qualified resource ids to fetch several secrets in one call.

## Rules
- Always URL-encode identifiers containing `/`, `@`, `+`, `&` (e.g. `prod/aws/db` -> `prod%2Faws%2Fdb`).
- Writing a secret is consequential: run under a least-privileged, audited identity.
- Batch reads return `404` (ResourcesNotFound) if ANY requested id is missing or not permitted. See errors/conjur-problem-types.yml.
