---
name: Load and update Conjur policy
description: Apply policy-as-code to declare roles, resources, grants, and permissions in Conjur.
api: openapi/openapi.yml
operations: [whoAmI, loadPolicy, replacePolicy, updatePolicy]
---

# Load and update Conjur policy

Conjur is configured with declarative YAML policy. Use this flow to create or change roles, variables, hosts, and grants.

## Preconditions
- You hold a valid Conjur access token for an identity with `create`/`update` privilege on the target policy branch.

## Steps
1. **Confirm identity** — `whoAmI` (`GET /whoami`) to verify the authenticated role before making changes.
2. **Choose the load mode against `/policies/{account}/policy/{identifier}`:**
   - `loadPolicy` (`POST`) — append: adds new objects, leaves existing ones untouched.
   - `replacePolicy` (`PUT`) — replace: makes the branch match the submitted policy exactly, removing objects not present (destructive, fully idempotent).
   - `updatePolicy` (`PATCH`) — update: modify existing objects without removing others.
3. Submit the policy YAML as the request body.

## Rules
- Prefer `POST` (append) for additive changes; reserve `PUT` (replace) for authoritative, reviewed policy because it DELETES objects absent from the submitted document.
- Policy loads are consequential and audited — run under a least-privileged identity.
- A `422` indicates malformed policy YAML or an invalid parameter; a `403` indicates insufficient privilege on the branch. See errors/conjur-problem-types.yml.
