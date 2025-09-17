# set-link-transfer

[![npm version](https://img.shields.io/npm/v/set-link-transfer.svg)](https://www.npmjs.com/package/set-link-transfer)
[![npm downloads](https://img.shields.io/npm/dm/set-link-transfer.svg)](https://www.npmjs.com/package/set-link-transfer)

**Secure, time-limited download links for files & text — ESM & TypeScript ready.**

- JWT-protected • TTL auto-cleanup • ZIP on-the-fly  
- Path-traversal safe • Concurrent-download guard • Structured logs

## Quick Start(ESM)

```ts
import express from 'express';
import { createTransferRouter } from 'set-link-transfer';

const app = express();
app.use(express.json());

app.use('/transfer', createTransferRouter({
  jwtSecret: process.env.JWT_SECRET!,   // ≥ 32 chars
  defaultTTL: 3600,                     // optional
  uploadPath: '/upload',                // optional
  downloadPath: '/download'             // optional
}));

app.listen(3000);
```

## API

### creatTransferRouter(options)

Returns an Express router with upload/download endpoints.

| **Option** | **Type** | **Default** | **Description** |
| :------- | :------ | :------- | :------------ |
| jwtSecret | string | ***null*** | **Required.** Signing key (min 32 chars) |
| jwtAlgo | string | 'HS256' | JWT algorithm (none blocked) |
| repo | MemoryRepo | new instance | Custom storage backend (advanced) |
| defaultTTL | number | 3600 | Global TTL (seconds) |
| uploadPath | string | '/upload' | Route prefix for uploads |
| downloadPath | string | '/download' | Route prefix for downloads |

### POST ${downloadPath}

__Creat download link(s).__

## BODY(JSON)

| **Field** | **Type** | **Description** |
| :-------- | :--------- | :--------------- |
| type | "text"/"file" | Content upload kind |
| payload | string/string[] | Text content or __absolute file path(s)__ |
| ttl | number | Override defaultTTL (seconds) |
| zip | boolean | If true and multiple files to single ZIP token |

## Example

```bash
curl -X POST http://localhost:3000/upload \
  -H "Content-Type: application/json" \
  -d '{"type":"text","payload":"hello 👋","ttl":600}'
```

## Response(Example)

```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "url": "http://localhost:3000/download/eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "ttl": 600
}
```

### Multiple files and ZIP in one step(Example)

```bash
curl -X POST http://localhost:3000/upload \
  -H "Content-Type: application/json" \
  -d '{"type":"file","payload":["/tmp/a.txt","/tmp/b.txt"],"zip":true}'
```

### GET ${downloadPath}/:token

Stream the content (or ZIP).
Headers always contain __Content-Disposition: attachment; filename="{id}"__.

## Status codes

| **Code** | **Mean** |
| :--------- | :--------- |
| 200 | OK |
| 400 | Path traversal detected |
| 401 | Invalid / malformed token |
| 409 | Download already in progress (concurrent lock) |
| 410 | Token expired or resource deleted |
| 422 | File disappeared since upload |
| 500 | Server-side archive / read error |

## License

[MIT](LICENSE)
