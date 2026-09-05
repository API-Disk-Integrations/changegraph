# ChangeGraph API TypeScript SDK

Monitor companies, products and websites and emit evidence-backed pricing, hiring, product, technology, executive and expansion events.

This package is the zero-runtime-dependency TypeScript/JavaScript client from
the audited public integration repository. It supports ESM and CommonJS on
Node.js 18 or newer. Import and construction perform no network request.

## Install

```sh
npm install changegraph
```

## Authenticated client

```ts
import { ChangeGraph } from 'changegraph'

const client = new ChangeGraph({
  apiKey: process.env.CHANGEGRAPH_API_KEY,
})
```

Never place an API key in browser code, source control, logs, or examples.
Requesting a sandbox key is an email-verification and claim flow; it does not
return a key in the initial response.

- [Product, docs, demo, pricing, privacy, and terms](https://changegraph-api.com/?utm_source=npm&utm_medium=package&utm_campaign=changegraph&utm_content=readme)
- [Source and changelog](https://github.com/API-Disk-Integrations/changegraph)
- [Issues](https://github.com/API-Disk-Integrations/changegraph/issues)

Security reports must not be filed in a public issue. Use the repository's
private security-reporting path after the owner confirms it is enabled.

MIT licensed. The API service remains governed by the product site's terms.
