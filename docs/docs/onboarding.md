# Onboarding Material

This section contains some references and material that will assist users and integrators onboarding process.

## References

The definite guide for integrators remains the service [OpenAPI Reference](openapi.md).

The codebase is available under the [MIT License](license.md) in our [GitHub code repository](https://github.com/datagems-eosc/query-recommender).

To get a better understanding of the endpoints and their payloads, you can read the [Api Overview](api-overview.md).

To know more about Api response codes and how to interpret them, you can take a look at the [Status & Error Codes](error-codes.md).

On aspects pertaining to security, you can read more at the [Security](security.md) section.

For an overall view of the service, you can go through the [Architecture](architecture.md).

For any questions, you can consult the [FAQ](faq.md).

If you are facing a problem, check if there is a relevant [issue](https://github.com/datagems-eosc/query-recommender/issues) answering your question.

You can always contact us through the [DataGEMS Help Desk](https://datagems.eu/contact-us/).

## Running locally

The service requires Python 3.10 or later. With [uv](https://docs.astral.sh/uv/):

```bash
git clone https://github.com/datagems-eosc/query-recommender.git
cd query-recommender
uv sync
export LLM_API_URL="https://chat.scayle.es/api/chat/completions"
export LLM_TOKEN="your-llm-token"
uv run python -m qr_api.main
```

The Api is then available at `http://localhost:5000/api/v1`, with the Swagger UI at `http://localhost:5000/api/v1/swagger`. See [Configuration](configuration.md) for all the settings, including how to point the service to a User Profiler.

## Examples

Sample request payloads are available in the code repository, under [qr_api/tests](https://github.com/datagems-eosc/query-recommender/tree/main/qr_api/tests). They cover a request for a known user, a request without context and a request for an unknown user. To send one:

```bash
curl -X POST "http://localhost:5000/api/v1/recommend-next-queries" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d @qr_api/tests/sample_query_1.json
```

`$TOKEN` must be an access token retrieved from the DataGEMS AAI service, with `query-recommender` in its audience.

## Demo application

The code repository includes a chat-style [Streamlit](https://streamlit.io/) demo, `app.py`. You type a natural language query, and the 3 recommended next queries are shown as buttons. Clicking one sends it as the next query.

```bash
pip install streamlit        # not included in the service dependencies
streamlit run app.py
```

In the sidebar you can:

* choose the Api to use: local (`http://localhost:5000/api/v1`, overridable with the `QR_API_URL` environment variable), the remote development instance, or a custom URL
* paste the Keycloak access token, sent to both the Query Recommender and the User Profiler. It can also be provided through the `DATAGEMS_TOKEN` environment variable. The demo warns when the token is missing or expired
* pick a user from the User Profiler (local or remote development instance). The user's interest keywords are shown, and their id is sent as `context.user_id`. If the User Profiler is unreachable, a user id can be typed by hand
* set the request timeout, check the Api status and clear the chat

## Tutorials

You can find useful material and onboarding guidelines in our social channels below, as well as our [platform documentation](https://datagems-eosc.github.io/).

## Keep in touch

Make sure to follow the DataGEMS channels to get the latest news and references:

* [GitHub](https://github.com/datagems-eosc/)
* [Instagram](https://www.instagram.com/datagems_eosc)
* [X](https://x.com/datagems_eosc)
* [YouTube](https://www.youtube.com/@DataGEMS-65n)
* [LinkedIn](https://www.linkedin.com/company/eosc-datagems)
* [Facebook](https://www.facebook.com/datagems.eosc/)
