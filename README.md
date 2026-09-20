# ECU Forge API

This repo documents the **public API** for [ECU Forge](https://ecuforge.byst.re), a free platform for converting engine ECU calibration files — starting with WinOLS `.kp` → TunerPro `.xdf`.

**Don't want to write code?** ECU Forge also has a plain website with a no-code upload form — go to [ecuforge.byst.re/converter](https://ecuforge.byst.re/converter), pick a file, click Convert, done. The API below is for scripts, tools, and integrations that need to do this programmatically; both the website and the API run the exact same converter, so results are identical either way.

```bash
curl -X POST https://ecuforge.byst.re/api/v1/convert/kp-to-xdf \
  -F "file=@KP.kp"
```

**Supported `.kp` files:** native WinOLS 5.0, builds ~5.84–5.91 (the build number is embedded in the file itself, e.g. `"5.91.0 Full"`), both full ECU project files and hand-made map packs ("Kennfeldpaket" — send the matching `.bin` too). Files from third-party tools instead of WinOLS directly, or outside this build range, may not convert correctly — see [`API.md`](API.md#convert-a-kp-file-to-xdf) for details.

- **Web converter (no code):** [ecuforge.byst.re/converter](https://ecuforge.byst.re/converter)
- **Full API reference:** [`API.md`](API.md) — every endpoint, request/response shapes, error codes, limits, CORS, and examples in curl / JavaScript / PHP.
- **Interactive docs (Swagger UI):** [ecuforge.byst.re/api/docs](https://ecuforge.byst.re/api/docs) — try real requests, including file uploads, from the browser.
- **OpenAPI spec:** [ecuforge.byst.re/openapi.json](https://ecuforge.byst.re/openapi.json) — import into Postman, Insomnia, or any OpenAPI-aware tool instead of hand-writing requests.
- **Changelog:** [`CHANGELOG.md`](CHANGELOG.md)
- **License:** [MIT](LICENSE)

## Availability

This is a free, best-effort service with **no uptime guarantee or SLA** — it can change or go offline at any time without notice. See [`API.md#availability`](API.md#availability) for details before depending on it for anything important.

## Client libraries

None yet — call the HTTP API directly for now (see the examples in [`API.md`](API.md#examples)). A PHP client and others are planned; they'll be linked from this README once available.
