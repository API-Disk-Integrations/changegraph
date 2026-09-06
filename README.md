# ChangeGraph API

Monitor companies, products and websites and emit evidence-backed pricing,
hiring, product, technology, executive and expansion events.

- [Product and pricing](https://changegraph-api.com/?utm_source=github&utm_medium=developer&utm_campaign=changegraph-github&utm_content=readme#pricing)
- [Developer documentation](https://changegraph-api.com/docs?utm_source=github&utm_medium=developer&utm_campaign=changegraph-github&utm_content=readme)
- [Create a free account](https://changegraph-api.com/signup?utm_source=github&utm_medium=developer&utm_campaign=changegraph-github&utm_content=readme)
- [OpenAPI contract](https://changegraph-api.com/openapi.json)
- [Postman collection](./postman_collection.json)

## Quickstart: see a detected change without an account

This public demo runs the production detection engine without storing or
metering the payload.

```bash
curl -sS -X POST https://changegraph-api.com/v1/demo/detect \
  -H 'content-type: application/json' \
  -d '{"subject":"Acme Corp","before":{"plan.pro.monthly":"49","headcount.engineering":"120"},"after":{"plan.pro.monthly":"59","headcount.engineering":"145"}}'
```

A successful response has `eventCount: 2`. Each item in `events` identifies the
change kind, direction, before/after values, confidence and supporting evidence:

```json
{
  "subject": "Acme Corp",
  "eventCount": 2,
  "events": [
    {
      "kind": "hiring",
      "direction": "increased",
      "factKey": "headcount.engineering",
      "before": "120",
      "after": "145",
      "confidence": 80,
      "evidence": [
        {"source": "demo://before", "observedAt": "2026-09-06T00:00:00.000Z", "factKey": "headcount.engineering", "before": "120", "after": null},
        {"source": "demo://before", "observedAt": "2026-09-06T00:00:00.000Z", "factKey": "headcount.engineering", "before": null, "after": "145"}
      ]
    },
    {
      "kind": "pricing",
      "direction": "increased",
      "factKey": "plan.pro.monthly",
      "before": "49",
      "after": "59",
      "confidence": 80,
      "evidence": [
        {"source": "demo://before", "observedAt": "2026-09-06T00:00:00.000Z", "factKey": "plan.pro.monthly", "before": "49", "after": null},
        {"source": "demo://before", "observedAt": "2026-09-06T00:00:00.000Z", "factKey": "plan.pro.monthly", "before": null, "after": "59"}
      ]
    }
  ],
  "requestId": "req_example"
}
```

That is the first useful result: a normalized event that an alert, CRM or
research workflow can act on. Each safe synthetic evidence row shows the
before/after observation supporting the event; the `demo://` sources make clear
that these are demonstration records rather than customer evidence.

## Create and use a free API key

Request the verification email. This attribution tuple identifies this README.

```bash
curl -sS -X POST https://changegraph-api.com/v1/keys \
  -H 'content-type: application/json' \
  -d '{"email":"you@example.com","source":{"source":"github","medium":"developer","campaign":"changegraph-github","content":"readme"}}'
```

Copy the one-time token from the email, exchange it, and copy the returned
`apiKey` immediately. The key is shown only once.

```bash
curl -sS -X POST https://changegraph-api.com/v1/keys/claim \
  -H 'content-type: application/json' \
  -d '{"token":"PASTE_ONE_TIME_TOKEN_FROM_EMAIL"}'

export KEY='PASTE_API_KEY_FROM_CLAIM_RESPONSE'
```

Create a monitor and capture its real ID, establish a baseline, then submit a
changed observation. Python is used only to read the `id` from the JSON reply.

```bash
MONITOR_ID="$(curl -sS -X POST https://changegraph-api.com/v1/monitors \
  -H "Authorization: Bearer $KEY" \
  -H 'content-type: application/json' \
  -d '{"subject":"Acme Corp","kinds":["pricing","hiring","executive"]}' \
  | python3 -c 'import json,sys; print(json.load(sys.stdin)["id"])')"

curl -sS -X POST "https://changegraph-api.com/v1/monitors/$MONITOR_ID/snapshots" \
  -H "Authorization: Bearer $KEY" \
  -H 'content-type: application/json' \
  -d '{"source":"https://acme.example/pricing","facts":{"plan.pro.monthly":"49","headcount.engineering":"120"}}'

curl -sS -X POST "https://changegraph-api.com/v1/monitors/$MONITOR_ID/snapshots" \
  -H "Authorization: Bearer $KEY" \
  -H 'content-type: application/json' \
  -d '{"source":"https://acme.example/pricing","facts":{"plan.pro.monthly":"59","headcount.engineering":"145"}}'
```

The first snapshot returns `baseline: true`; the second returns
`baseline: false`, `eventCount: 2`, and the detected events.

## SDKs

- [Python SDK](./sdk/python/changegraph.py) — reads `CHANGEGRAPH_API_KEY`
- [TypeScript SDK](./sdk/typescript/index.ts)

Copy the file you need into your project. The OpenAPI document remains the
authoritative operation and schema contract.

## Collection scope

The runnable Postman collection includes the public demo, the no-key checkout
path, key bootstrap, and API-key product operations. It intentionally excludes
the provider-only billing webhook and browser-session subscription, invoice,
and payment routes: those require a signed hub request or the dashboard's
HttpOnly session and CSRF controls, and a bearer API key cannot run them. The
OpenAPI document linked above remains the reference for those operations.

## Authentication and troubleshooting

API operations accept `Authorization: Bearer <API_KEY>` (or `x-api-key` where
documented). Public demo and health operations require no credential.

- `401`: set `KEY` to the API key returned by `/v1/keys/claim`; request a new
  key if it was revoked.
- `400 invalid_request`: make both `before` and `after` non-empty string maps.
  A client-side schema tool may label the same input problem `422` before send.
- `429`: wait for `Retry-After` when present, then retry with backoff.

Errors use a stable `error.code` and a request ID. Include that request ID when
asking for support; never include the API key or claim token.

## Distribution attribution

The key request above uses the stable tuple
`github / developer / changegraph-github / readme`. The Postman collection and
SDKs carry their own source metadata. Attribution compares qualified activation
and retained use; it does not claim that this channel already performs.

## License

[MIT](./LICENSE)
