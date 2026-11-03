# Deployment

The service is part of the DataGEMS platform offered through an existing deployment, following the DataGEMS release and deployment procedures over a managed infrastructure. The purpose of this section is not to detail the deployment processes put in place by the DataGEMS team, but to give an overview of what the service needs to run.

## Docker

The service is offered as a Docker image through the DataGEMS GitHub Organization Packages, as `ghcr.io/datagems-eosc/datagems-eosc/query-recommender:<version>`. This process is described in the relevant [Automations](automations.md) section.

The image is based on `python:3.12-slim`, installs the locked dependencies with [uv](https://docs.astral.sh/uv/) and starts the Api with uvicorn on port `5000`:

```bash
docker run -p 5000:5000 \
  -e LLM_API_URL="https://chat.scayle.es/api/chat/completions" \
  -e LLM_TOKEN="your-llm-token" \
  -e USER_PROFILER_API_URL="http://user-profiler:5000" \
  ghcr.io/datagems-eosc/datagems-eosc/query-recommender:<version>
```

## Kubernetes

In the DataGEMS development cluster the service runs in the `univr` namespace, as:

* a **Deployment** `query-recommender` with a single replica, a readiness probe on TCP port 5000 and resource limits of 500m CPU and 2Gi memory
* a **ClusterIP Service** `query-recommender-service` on port 5000
* an **Ingress** routing `https://datagems-dev.scayle.es/qr/...` to the service, stripping the `/qr` prefix (`ROOT_PATH` is set to `/qr` accordingly). Proxy read and send timeouts are raised to 300 seconds to leave room for slow LLM replies
* a **VaultStaticSecret** that syncs the `LLM_TOKEN` from Vault into the `query-recommender-secrets` Kubernetes secret

The manifests are maintained in the deployment repository of the service.

## Configuration

In order for the service to operate properly, the needed configuration values must be set to match the environment that it must operate in. The needed configuration is described in the relevant [Configuration](configuration.md) section.

## Dependencies

For the service to be able to operate, its underpinning services must be available and accessible:

* the DataGEMS AAI, to verify tokens. Without it, authenticated calls fail with 503
* the LLM endpoint, to generate the suggestions. Without it, recommendation calls fail with 502
* the User Profiler, to personalise the suggestions. Without it, recommendations still work, without personalisation

The [Architecture](architecture.md) section describes the ecosystem in which the service needs to operate.
