# Configuration

The service is configured entirely through environment variables. They are read once, when the service starts, so any change requires a restart.

## Environment variables

### LLM

| Variable | Default | Description |
|---|---|---|
| `LLM_API_URL` | *(none, required)* | Full URL of the OpenAI-compatible chat completions endpoint, e.g. `https://chat.scayle.es/api/chat/completions`. The list of available models is read from `/api/models` on the same scheme, host and port |
| `LLM_TOKEN` | *(none, required)* | Bearer token used to call the LLM endpoint. This is a secret |
| `LLM_SSL_VERIFY` | `true` | TLS certificate verification for the LLM endpoint. `true` verifies against the system CAs, a file path verifies against that CA bundle or certificate, `false` disables verification (for an endpoint with a self-signed certificate). With `false`, the service prints a warning at startup |

### User Profiler

| Variable | Default | Description |
|---|---|---|
| `USER_PROFILER_API_URL` | `http://localhost:8000` | Base URL of the User Profiler Api, used to fetch the users' interest keywords. In the DataGEMS cluster this is `http://user-profiler-service:5000` |

If the User Profiler is not reachable at this URL, recommendations are still produced, without personalisation.

### Authentication

| Variable | Default | Description |
|---|---|---|
| `OIDC_ISSUER_URL` | `https://datagems-dev.scayle.es/oauth/realms/dev` | The expected token issuer (*iss* claim), i.e. the DataGEMS AAI realm |
| `OIDC_JWKS_URL` | `{OIDC_ISSUER_URL}/protocol/openid-connect/certs` | Where the issuer public keys are fetched from |
| `OIDC_CLIENT_ID` | `query-recommender` | The client id that must appear in the token audience (*aud* claim) |
| `JWKS_CACHE_TTL_SECONDS` | `300` | How long the public keys are cached, in seconds |

See [Security](security.md) for how these are used.

### Http

| Variable | Default | Description |
|---|---|---|
| `ROOT_PATH` | *(empty)* | The path prefix under which a reverse proxy exposes the service, e.g. `/qr`. It is used so that the generated OpenAPI document and the Swagger UI work behind the proxy. The proxy is expected to strip the prefix before forwarding requests |

The service listens on port `5000`, both in the Docker image and when started with `python -m qr_api.main`.

## Model

The preferred model is `llama-3.3`. At each request the service checks it against the models listed by the LLM endpoint, and falls back to the first available model if it is not offered. The preferred model is currently set in code, not through configuration.

## Example

```bash
export LLM_API_URL="https://chat.scayle.es/api/chat/completions"
export LLM_TOKEN="your-llm-token"
export USER_PROFILER_API_URL="http://localhost:8000"
python -m qr_api.main
```

```powershell
$env:LLM_API_URL = "https://chat.scayle.es/api/chat/completions"
$env:LLM_TOKEN = "your-llm-token"
$env:USER_PROFILER_API_URL = "http://localhost:8000"
python -m qr_api.main
```

When working against an LLM endpoint with a self-signed certificate, save the endpoint's certificate once and point `LLM_SSL_VERIFY` to it, rather than disabling verification:

```bash
openssl s_client -connect <llm-host>:<port> </dev/null 2>/dev/null | openssl x509 > llm.pem
export LLM_SSL_VERIFY="$PWD/llm.pem"
```

## Secrets

`LLM_TOKEN` is the only secret the service needs. It should not be stored alongside the other configuration values. In the DataGEMS deployment it is kept in Vault and exposed to the container through a Kubernetes secret, as described in [Deployment](deployment.md).
