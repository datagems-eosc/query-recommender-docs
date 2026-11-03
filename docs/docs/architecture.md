# Service Architecture

The Query Recommender is a small, stateless HTTP Api written in Python with [FastAPI](https://fastapi.tiangolo.com/) and served by [uvicorn](https://www.uvicorn.org/). It does not own a data store: every recommendation is computed on demand from the incoming request, the user's profile and an LLM call. Nothing is persisted between requests.

## Ecosystem

The service sits between the DataGEMS clients that want to offer "what to ask next" suggestions and two underpinning services:

* **DataGEMS AAI (Keycloak)**: issues the access tokens that callers present. The service downloads the realm's public keys (JWKS) to verify them. See [Security](security.md).
* **[User Profiler](https://github.com/datagems-eosc/user-profiler)**: holds each user's interest keywords, extracted from their past activity. The service reads them to personalise the suggestions.
* **LLM endpoint**: an OpenAI-compatible chat completions Api (in the DataGEMS deployment, the one served at `chat.scayle.es`). It generates the suggestions.

```
                 ┌──────────────────────┐
  client ──────▶ │  Query Recommender   │ ──── JWKS ─────▶  DataGEMS AAI (Keycloak)
  (Bearer token) │       Api            │
                 │                      │ ── profile ────▶  User Profiler  (caller's token forwarded)
                 │                      │
                 │                      │ ── models ─────▶  LLM endpoint   /api/models
                 │                      │ ── chat ───────▶  LLM endpoint   /api/chat/completions
                 └──────────────────────┘
```

## Request flow

A call to `POST /api/v1/recommend-next-queries` goes through the following steps:

1. **Authentication.** The bearer token is verified (signature, issuer, expiry) against the AAI public keys, then its audience is checked for the service's client id. Failures stop the request with 401, 403 or 503 as described in [Status & Error Codes](error-codes.md).
2. **User keywords.** If the request `context` contains a `user_id`, the service calls `GET {USER_PROFILER_API_URL}/user/{user_id}/profile`, forwarding the caller's token, and takes the `keywords` of the profile. The User Profiler returns keywords strongest (most recently seen) first, and only the top 15 are kept. This step never fails the request: if there is no `user_id`, the user is unknown (404), the token is rejected (401/403), or the profiler is unreachable, the recommendation is generated with no keywords.
3. **Model selection.** The service lists the models available at the LLM endpoint (`/api/models`, on the same host as the chat endpoint) and uses the preferred model (`llama-3.3`) if it is there, otherwise the first model listed. If the list cannot be retrieved, it tries the preferred model name as is.
4. **Generation.** The LLM receives a system prompt asking for 3 short next search queries, close to the current one but not repeating it, favouring the most relevant keywords, plus a user message with the current query and the keywords. The reply is requested as JSON of the form `{"queries": [...]}`, constrained with a JSON schema `response_format`. If the LLM endpoint rejects `response_format` with a 400, the call is retried once relying on the prompt alone.
5. **Validation and clean-up.** The reply is parsed (tolerating markdown fences or extra text around the JSON object) and validated against the expected shape. Queries are trimmed, and empty ones, duplicates and repeats of the current query (case-insensitive) are dropped. At most 3 are returned.
6. **Response.** The service answers `200 OK` with `{"next_queries": [...]}`. If the LLM could not be reached, or its reply was not valid, it answers `502 Bad Gateway` instead.

The caller's token is only ever forwarded to the User Profiler. The LLM endpoint is called with the service's own LLM token.

## Code layout

The service code lives in the [query-recommender](https://github.com/datagems-eosc/query-recommender) repository:

* `qr_api/main.py`: the FastAPI application, the Api root and the router mounting under `/api/v1`
* `qr_api/resources/endpoints.py`: the `recommend-next-queries` endpoint
* `qr_api/resources/security.py`: token validation and the `authtest` endpoint
* `qr_api/resources/user_tools.py`: the User Profiler client
* `qr_api/resources/LLM_tools.py`: model selection, prompt, LLM call and reply validation
* `qr_api/resources/data_models.py`: the request and response models
* `qr_api/tests/`: sample request payloads
* `app.py`: a Streamlit chat demo that calls the Api, described in [Onboarding](onboarding.md)
