---
name: Upload a file to Auto Drive and retrieve it by CID
description: Authenticate, upload a file to Autonomys Auto Drive permanent storage, confirm the upload, and download it back by its content identifier (CID).
api: openapi/autonomys-auto-drive-openapi.json
base_url: https://mainnet.auto-drive.autonomys.xyz
operations:
  - POST /uploads/file
  - POST /uploads/file/{uploadId}/chunk
  - POST /uploads/{uploadId}/complete
  - GET /objects/{cid}/status
  - GET /objects/{cid}/metadata
  - GET /objects/{cid}/download
---

# Upload a file to Auto Drive and retrieve it by CID

Auto Drive stores data permanently on the Autonomys network, addressed by an IPFS-style CID. All requests require an API key created in the dashboard at https://ai3.storage.

## Auth
Send both headers on every request:
- `Authorization: Bearer <api-key>`
- `X-Auth-Provider: apikey`

## Steps
1. **Start the upload** — `POST /uploads/file` with the file metadata. The response returns an `id` (the `uploadId`).
2. **Send file data** — for larger files, `POST /uploads/file/{uploadId}/chunk` for each chunk.
3. **Complete the upload** — `POST /uploads/{uploadId}/complete`. The response returns the object **CID**.
4. **Confirm archival** — poll `GET /objects/{cid}/status` until the object reports it is stored.
5. **Inspect metadata** — `GET /objects/{cid}/metadata` for name, size, and type.
6. **Retrieve** — `GET /objects/{cid}/download` (or `GET /downloads/{cid}`) to stream the bytes back.

## Rules
- CIDs are permanent and content-addressed — the same content yields the same CID.
- Errors are plain HTTP status codes with a description string (not RFC 9457): expect `400` invalid input, `401` missing/invalid key, `403` insufficient credits/permission, `404` unknown CID, `500` upload failure. See errors/autonomys-problem-types.yml.
- Watch monthly quota (default 100 MB uploads / 5 GB downloads) via `GET /subscriptions/credits`.
