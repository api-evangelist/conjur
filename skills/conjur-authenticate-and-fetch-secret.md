---
name: Authenticate to Conjur and fetch a secret
description: Exchange credentials for a short-lived Conjur access token and use it to read a secret value.
api: openapi/openapi.yml
operations: [getAPIKey, getAccessToken, getSecret]
---

# Authenticate to Conjur and fetch a secret

Use this flow to read a secret from Conjur / CyberArk Secrets Manager as a host or user identity.

## Preconditions
- You know the Conjur `account` (e.g. `default`) and the appliance base URL.
- You have a login (user or host id) and either a password or an API key.

## Steps
1. **(Optional) Retrieve your API key** — `getAPIKey` (`GET /authn/{account}/login`) using HTTP Basic auth with your username and password. Returns the API key for subsequent token requests.
2. **Get an access token** — `getAccessToken` (`POST /authn/{account}/{login}/authenticate`) with the API key as the request body. The response is a short-lived, base64-encoded access token.
3. **Read the secret** — `getSecret` (`GET /secrets/{account}/{kind}/{identifier}`) sending the token as `Authorization: Token token="<base64-token>"`. URL-encode the identifier (e.g. `prod/db/password` -> `prod%2Fdb%2Fpassword`).

## Rules
- The access token is short-lived; re-authenticate (step 2) when it expires rather than caching indefinitely.
- On `401` the token is missing/invalid — re-run step 2. On `404` the variable does not exist, has no value set, or you lack privileges. On `403` you lack the necessary privilege.
- Cloud/machine identities should prefer a cloud-native authenticator (authn-iam/azure/gcp/k8s/jwt/oidc) instead of a stored API key. See conventions/conjur-conventions.yml.
