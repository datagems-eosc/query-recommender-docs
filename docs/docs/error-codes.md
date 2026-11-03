# Status & Error Codes

The HTTP Api exposed by the service exposes restful endpoints to offer its functionality. These endpoints are documented using the [OpenAPI specification](http://openapis.org/) and available for integrators to consult in the respective [API section](openapi.md).

In this section, we describe further the HTTP Status Codes that the service may return and the error information that accompanies them.

## Status Codes

The service makes use of the following subset of HTTP Status Codes:

* 200 OK - the request has succeeded
* 401 Unauthorized - the request lacks valid authentication credentials: the token is missing, malformed, expired, has an invalid signature or was issued by another issuer
* 403 Forbidden - the token is valid, but it was not issued for this service (its audience does not include the service client id)
* 422 Unprocessable Content - the request body does not match the expected model, for example a missing `current_query`
* 502 Bad Gateway - the LLM endpoint could not be reached, returned an error, or replied with output that is not in the expected format
* 503 Service Unavailable - the public keys needed to verify the token could not be fetched from the identity provider
* 500 Internal Server Error - an unexpected condition prevented the service from fulfilling the request

Authentication is checked before the request body, so a request with both a bad token and a bad body gets a 401 or 403, not a 422.

Failures of the User Profiler never produce an error: the service falls back to recommendations without user keywords, as described in the [Architecture](architecture.md).

## Error responses

Error responses follow the standard FastAPI format: a JSON object with a `detail` property. For all status codes except 422, `detail` is a human readable message:

| Status | `detail` | Cause |
|---|---|---|
| 401 | `Not authenticated` | No `Authorization: Bearer` header on `recommend-next-queries` |
| 401 | `Missing bearer token` | No `Authorization: Bearer` header on `authtest` |
| 401 | `Invalid token` | Bad signature, wrong issuer, expired or malformed token |
| 403 | `Token missing required audience: query-recommender` | The token audience does not include the service client id |
| 502 | `The LLM could not be reached or returned an invalid response` | The LLM call failed or its reply could not be parsed |
| 503 | `Could not fetch JWKS: ...` | The identity provider public keys are not reachable |

For example:

```json
{
    "detail": "Token missing required audience: query-recommender"
}
```

401 responses produced when the header is missing on `recommend-next-queries` also include a `WWW-Authenticate: Bearer` header.

## Validation errors

For 422 responses, `detail` is a list with one entry per problem found in the request. Each entry has a `loc` pointing at the problematic property, a `msg` describing the problem, a machine friendly `type` and the `input` that was received.

For example, a request to `recommend-next-queries` with an empty body object:

```json
{
    "detail": [
        {
            "type": "missing",
            "loc": ["body", "current_query"],
            "msg": "Field required",
            "input": {}
        }
    ]
}
```

Or a request to `authtest` with an extra property:

```json
{
    "detail": [
        {
            "type": "extra_forbidden",
            "loc": ["body", "limit"],
            "msg": "Extra inputs are not permitted",
            "input": 10
        }
    ]
}
```

## Bad Gateway

A 502 means the request was valid and authenticated, but the recommendation could not be produced by the LLM. Possible causes are the LLM endpoint being down or unreachable, the LLM token being invalid, the selected model being unavailable, or the model replying with something that is not a JSON object of the form `{"queries": [...]}`. The exact cause is logged by the service (see [Logging](logging.md)) but is not returned to the caller. As LLM replies vary between calls, retrying the request may succeed.
