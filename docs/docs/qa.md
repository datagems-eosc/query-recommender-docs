# Quality Assurance

Key aspects of the Quality Assurance checklist that DataGEMS services must pass have been defined in the processes and documents governing the platform development and quality assurance. In this section we present a selected subset of these that are directly, publicly available.

## Reproducible builds

The service dependencies are pinned in the `uv.lock` file of the code repository, and the Docker image is built from that lock file (`uv sync --frozen`). Every published image is tagged with the version tag of the commit it was built from. See [Automations](automations.md).

## Output validation

As the recommendations are produced by an LLM, the service does not trust its output. The LLM is asked to reply in a fixed JSON format, constrained with a JSON schema where the endpoint supports it. Every reply is then validated against that format, and the suggestions are cleaned (empty entries, duplicates and repeats of the current query are removed, and at most 3 are kept) before being returned. Replies that cannot be validated are reported to the caller as a 502 instead of being passed on. See [Architecture](architecture.md).

## Testing

Sample request payloads are kept in the code repository under [qr_api/tests](https://github.com/datagems-eosc/query-recommender/tree/main/qr_api/tests), covering a known user, a request without context and an unknown user. They can be sent to a local or deployed instance to smoke test it, as shown in [Onboarding](onboarding.md).

The Streamlit demo described in [Onboarding](onboarding.md) can be used for manual, end to end testing against a local or deployed instance, including the User Profiler integration.
