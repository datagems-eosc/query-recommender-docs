# Maintenance

The service is part of the DataGEMS platform offered through an existing deployment, following the DataGEMS release and deployment procedures over a managed infrastructure along with the maintenance activities that are scheduled within the platform. The purpose of this section is not to detail the maintenance activities put in place by the DataGEMS team.

## Healthchecks

The Api root, `GET /api/v1`, does not require authentication and answers `200 OK` as soon as the service is up. It can be used as a liveness check:

```bash
curl https://datagems-dev.scayle.es/qr/api/v1
```

```json
{ "message": "API V1 is running", "endpoints": { ... } }
```

This check does not cover the underpinning services. To also verify authentication end to end, call `POST /api/v1/authtest` with a valid token. To verify the LLM endpoint, a call to `recommend-next-queries` is needed.

In Kubernetes, the readiness of the container is checked with a TCP probe on port 5000.

## Versions & Updates

The service follows a semantic versioning scheme and structures versions as MAJOR.MINOR.PATCH:

* MAJOR (X.0.0): Breaking changes that are incompatible with previous versions.
* MINOR (X.Y.0): New features added in a backward-compatible way.
* PATCH (X.Y.Z): Bug fixes and security patches that do not affect compatibility.

Each version is released by pushing a `vX.Y.Z` tag, which builds and publishes the matching Docker image, as described in [Automations](automations.md).

## Backups

The service is stateless and keeps no data of its own, so there is nothing to back up. The user profiles it reads are owned and backed up by the User Profiler.

## Troubleshooting

Troubleshooting is primarily done through the logs, described in the respective [logging](logging.md) section. Common symptoms and their usual causes are:

* **401 on every call**: the token is expired, or was issued by a different realm than `OIDC_ISSUER_URL`
* **403 on every call**: the token audience does not include `query-recommender`. The client must request a token for this audience
* **502 on every call**: the LLM endpoint is unreachable, `LLM_TOKEN` is wrong, or TLS verification is failing (look for `CERTIFICATE_VERIFY_FAILED` in the logs)
* **Recommendations work but are never personalised**: look for User Profiler errors in the logs. Usually `USER_PROFILER_API_URL` is wrong, or the token is not accepted by the User Profiler
