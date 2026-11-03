# Logging

The service writes its logs to standard output, where they are collected by the container runtime. In the DataGEMS cluster they can be read with the usual Kubernetes tooling, e.g. `kubectl logs -n univr deployment/query-recommender`.

## Access logs

Each request served is logged by uvicorn in its standard access log format, with the client address, method, path and status code:

```
INFO:     10.0.0.12:53124 - "POST /api/v1/recommend-next-queries HTTP/1.1" 200 OK
```

## Troubleshooting logs

The service logs the conditions that are handled without failing the request, or that are hidden from the caller behind a generic error:

| Message | Meaning |
|---|---|
| `JWT validation failed: ...` | A token was rejected (401). The message gives the reason, e.g. an expired signature |
| `User Profiler rejected the token (401/403): ...` | The User Profiler did not accept the forwarded token, so the recommendation was made without keywords. Usually means the token audience does not include the User Profiler |
| `Error while calling User Profiler API: ...` | The User Profiler could not be reached, so the recommendation was made without keywords |
| `Could not reach models endpoint: ...` | The list of available LLM models could not be fetched, so the preferred model name was used as is |
| `Error while calling API: ...` | The LLM call failed (502 to the caller) |
| `LLM reply contains no JSON object: ...` | The LLM replied without any JSON (502 to the caller). The reply is included |
| `LLM reply does not match the expected schema: ...` | The LLM replied with JSON of the wrong shape (502 to the caller) |
| `WARNING: TLS certificate verification for the LLM API is disabled ...` | Printed at startup when `LLM_SSL_VERIFY=false` |

The service does not currently produce structured (JSON) logs, propagate a correlation identifier, or emit accounting entries.
