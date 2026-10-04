# ECU Forge API

This repo documents the **public API** for [ECU Forge](https://ecuforge.byst.re), a free platform of tools for working with engine ECU calibration files. Available tools:

- **KP → XDF** — convert a WinOLS `.kp` calibration file to a TunerPro `.xdf` definition.
- **Decrypt XDF** — decrypt an encrypted TunerPro `.xdf` file back into plain, readable XML.

**Don't want to write code?** ECU Forge also has a plain website with no-code upload forms — go to [ecuforge.byst.re](https://ecuforge.byst.re), pick a tool, upload a file, done. The API below is for scripts, tools, and integrations that need to do this programmatically; both the website and the API run the exact same code, so results are identical either way.

```bash
# Convert a WinOLS .kp to a TunerPro .xdf
curl -X POST https://ecuforge.byst.re/api/v1/convert/kp-to-xdf \
  -F "file=@KP.kp"

# Decrypt an encrypted TunerPro .xdf back into plain XML
curl -X POST https://ecuforge.byst.re/api/v1/decrypt/xdf \
  -F "file=@Encrypted.xdf"
```

**Supported `.kp` files:** native WinOLS 5.0, builds ~5.84–5.91 (the build number is embedded in the file itself, e.g. `"5.91.0 Full"`), both full ECU project files and hand-made map packs ("Kennfeldpaket" — send the matching `.bin` too). Files from third-party tools instead of WinOLS directly, or outside this build range, may not convert correctly — see [`API.md`](API.md#convert-a-kp-file-to-xdf) for details.

**Decrypt XDF** accepts encrypted TunerPro `.xdf` files (they begin with a fixed TunerPro header); the decryption key lives on the server, so there's nothing extra to send. An already plain-text `.xdf` doesn't need this tool — see [`API.md`](API.md#decrypt-an-encrypted-xdf-file) for details.

- **Web tools (no code):** [ecuforge.byst.re/converter](https://ecuforge.byst.re/converter) (KP → XDF) · [ecuforge.byst.re/decrypt-xdf](https://ecuforge.byst.re/decrypt-xdf) (Decrypt XDF)
- **Full API reference:** [`API.md`](API.md) — every endpoint, request/response shapes, error codes, limits, CORS, and examples in curl / JavaScript / PHP.
- **Interactive docs (Swagger UI):** [ecuforge.byst.re/api/docs](https://ecuforge.byst.re/api/docs) — try real requests, including file uploads, from the browser.
- **OpenAPI spec:** [ecuforge.byst.re/openapi.json](https://ecuforge.byst.re/openapi.json) — import into Postman, Insomnia, or any OpenAPI-aware tool instead of hand-writing requests.
- **Changelog:** [`CHANGELOG.md`](CHANGELOG.md)
- **License:** [MIT](LICENSE)

## Availability

This is a free, best-effort service with **no uptime guarantee or SLA** — it can change or go offline at any time without notice. See [`API.md#availability`](API.md#availability) for details before depending on it for anything important.

## Client libraries

None yet — call the HTTP API directly for now (see the examples in [`API.md`](API.md#examples)). A PHP client and others are planned; they'll be linked from this README once available.
