# Security

Key aspects of the Security checklist and practices that DataGEMS services must pass have been defined in the processes and documents governing the platform development and quality assurance. In this section we present a selected subset of these that are directly, publicly available and affect the usage and configuration of the service.

## Authentication

The `recommend-next-queries` and `authtest` endpoints require authentication using Bearer tokens in the form of JWTs (JSON Web Tokens). Clients must include a valid token in the Authorization header of each HTTP request, using the following format:

```
Authorization: Bearer <token>
```

The Api root (`GET /api/v1`) and the OpenAPI documentation endpoints are public.

The service only accepts JWTs issued by the trusted identity provider, the [DataGEMS AAI service](https://github.com/datagems-eosc/dg-aai). When a token is received, the service:

1. verifies the token signature with the public keys (JWKS) published by the issuer. Only `RS256` signatures are accepted
2. checks that the issuer *iss* claim matches the configured DataGEMS AAI realm
3. checks the expiration *exp* claim, to confirm the token is still valid
4. checks that the audience *aud* claim contains the client id of this service (`query-recommender` by default). *aud* can be either a single string or a list

Only if all these checks pass will the request be processed. A missing or invalid token results in `401 Unauthorized`, while a valid token that was not issued for this service results in `403 Forbidden`. If the public keys cannot be fetched from the issuer, the service answers `503 Service Unavailable`. See [Status & Error Codes](error-codes.md) for the response bodies.

The public keys are cached in memory for 5 minutes by default, so a key rotation at the issuer can take up to that long to be picked up.

You can check that a token is accepted by calling the `POST /api/v1/authtest` endpoint, described in the [Api Overview](api-overview.md).

The location of the configuration governing this behaviour is described in the relevant [Configuration](configuration.md) section.

## Authorization

There are no per-user permissions within the service. Any caller presenting a valid token with the right audience can request recommendations, for any `user_id`.

## Calls to other services

To personalise the recommendations, the service calls the User Profiler on behalf of the caller. It does so by forwarding the caller's access token as is, in the Authorization header. There is no token exchange: for personalisation to work, the token must also be accepted by the User Profiler, which means its audience must include the User Profiler client id as well. If the User Profiler rejects the token, the service logs the rejection and falls back to recommendations without keywords, so a misconfigured audience shows up as less personal suggestions rather than as an error.

The caller's token is never sent to the LLM endpoint. The LLM endpoint is called with a dedicated service token.

## TLS towards the LLM endpoint

The service verifies the TLS certificate of the LLM endpoint against the system certificate authorities by default. This can be pointed to a specific CA bundle or certificate, or, for development against an endpoint with a self-signed certificate, disabled. Disabling verification prints a warning at startup. See [Configuration](configuration.md).

## Secrets

The LLM token is a secret and must be handled separately from the other configuration values. In the DataGEMS deployment it is kept in Vault and injected into the container as an environment variable. See [Configuration](configuration.md) and [Deployment](deployment.md).
