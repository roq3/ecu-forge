# ECU Forge API

Public, free API for converting engine ECU calibration files. No API keys, no accounts, no rate-limited tiers — anyone can call it directly, including straight from browser JavaScript (CORS is fully open).

- **Base URL:** `https://ecuforge.byst.re/api/v1`
- **Interactive docs:** [`https://ecuforge.byst.re/api/docs`](https://ecuforge.byst.re/api/docs) (Swagger UI, generated from the live API — lets you try requests, including file uploads, straight from the browser) and [`/redoc`](https://ecuforge.byst.re/redoc) for a read-only reference view.
- **OpenAPI spec:** [`https://ecuforge.byst.re/openapi.json`](https://ecuforge.byst.re/openapi.json) — the same definition backing the docs above, importable into Postman, Insomnia, or any other OpenAPI-aware client instead of hand-writing requests.
- **Auth:** none. The API is intentionally public and anonymous.

This document covers the currently available endpoint. More conversions may be added over time under the same `/api/v1/convert/...` structure — check `/api/docs` for the current, authoritative list if this file is out of date.

## Availability

This is a free, best-effort service with **no uptime guarantee or SLA**. It can be changed, rate-limited, or taken down entirely at any time, without notice. Don't build anything business-critical on top of it as-is — if you need guaranteed availability, self-host the converter (see the main [ECU Forge](https://ecuforge.byst.re) project) instead of depending on this public instance.

## Client libraries

There is no official client library yet — for now, call the HTTP API directly (see the `curl` and JavaScript examples below, which work with zero dependencies). PHP and other language clients are planned; when available they'll be linked from this repository's README.

## Contents

- [Convert a .kp file to .xdf](#convert-a-kp-file-to-xdf)
- [Downloading the result](#downloading-the-result)
- [Errors](#errors)
- [Limits](#limits)
- [CORS](#cors)
- [Examples](#examples)

## Convert a .kp file to .xdf

```
POST /api/v1/convert/kp-to-xdf
Content-Type: multipart/form-data
```

Converts an uploaded [WinOLS](https://en.wikipedia.org/wiki/WinOLS) `.kp` calibration project file into a [TunerPro](http://www.tunerpro.net/) `.xdf` definition file.

**Supported `.kp` files:** native WinOLS 5.0, builds ~5.84–5.91 — the build number is embedded in the file itself (e.g. `"5.91.0 Full"`). Files produced by a third-party tool instead of WinOLS directly, or from outside this build range, may use a different internal layout and won't convert correctly (you'll get a `conversion_failed` error, see [Errors](#errors)).

### Request fields

| Field | Required | Description |
|---|---|---|
| `file` | yes | The `.kp` file to convert. |
| `bin` | no | The original ECU firmware image the `.kp` file was created against. Providing it lets the converter validate map addresses against real data instead of guessing, which produces a result with far fewer unverified (`[?]`) maps. Conversion still works without it — just with lower confidence. |

### Success response — `200 OK`

```json
{
  "status": "ok",
  "download_url": "https://ecuforge.byst.re/dl/aZ3xQ9kLmPqRtVwXyB2cFhJn",
  "expires_at": "2026-09-19T15:32:00Z",
  "filename": "KP.xdf",
  "summary": {
    "total_maps": 58,
    "verified_maps": 57,
    "unverified_maps": ["Fuel correction by fuel temperature"]
  }
}
```

| Field | Meaning |
|---|---|
| `download_url` | A one-time, single-use link to fetch the converted file — see [Downloading the result](#downloading-the-result). This is **not** a direct file URL and can't be reused. |
| `expires_at` | When `download_url` stops working if it's never used. |
| `summary.total_maps` | Number of calibration maps found in the `.kp` file. |
| `summary.verified_maps` | How many of those maps had their addresses confirmed against the `.bin` (or against internal structural checks, if no `.bin` was given). |
| `summary.unverified_maps` | Titles of maps that could not be confirmed — worth double-checking manually in TunerPro before using the result on a live vehicle. |

## Downloading the result

```
GET /dl/{token}
```

`download_url` from the conversion response points here. The token is:

- **single-use** — the first successful request consumes it; requesting the same URL again returns `404 Not Found`.
- **time-limited** — it also expires a short time after being issued, even if never used.

A successful download returns the file with:

```
Content-Type: application/xml
Content-Disposition: attachment; filename="KP.xdf"
```

Don't cache or store `download_url` for later — treat it as valid for one immediate download right after conversion.

## Errors

All error responses share this shape:

```json
{
  "status": "error",
  "code": "file_too_large",
  "message": "File exceeds the 1 MB limit"
}
```

| HTTP status | `code` | When |
|---|---|---|
| `400` | `missing_file` | No `file` field was sent, or it was empty. |
| `413` | `file_too_large` | `file` or `bin` exceeds the configured size limit (see [Limits](#limits)). |
| `422` | `conversion_failed` | The file isn't a recognizable `.kp`, or the converter couldn't process it (e.g. unsupported WinOLS version). |
| `500` | `internal_error` | Unexpected server error. |

`GET /dl/{token}` returns `404 Not Found` with `{"detail": "Not Found"}` when the token doesn't exist, was already used, or has expired — intentionally without distinguishing which, since that distinction isn't useful to a caller and reveals nothing to someone guessing tokens. Note this error shape (`{"detail": ...}`) is different from the `{"status", "code", "message"}` shape used by `/api/v1/convert/kp-to-xdf` above — `/dl/{token}` isn't under `/api/v1` and doesn't follow the same error contract.

## Limits

| Limit | Value |
|---|---|
| Max `.kp` size | 1 MB |
| Max `.bin` size | 4 MB |

Real-world `.kp` files are typically tens of KB up to a few hundred KB; firmware images (`.bin`) are usually a few hundred KB up to a couple of MB. These limits may change — `/api/docs` always reflects the current configuration.

There is currently no rate limiting; please be a reasonable citizen (don't hammer the endpoint in a tight loop) so the API stays free and open for everyone.

## CORS

The API sends fully open CORS headers:

```
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: POST, GET, OPTIONS
```

That means you can call it directly from a browser-based tool hosted on any domain — no server-side proxy needed.

## Examples

### curl — convert with just a `.kp` file

```bash
curl -X POST https://ecuforge.byst.re/api/v1/convert/kp-to-xdf \
  -F "file=@KP.kp"
```

### curl — convert with `.kp` + `.bin` for a more confident result

```bash
curl -X POST https://ecuforge.byst.re/api/v1/convert/kp-to-xdf \
  -F "file=@KP.kp" \
  -F "bin=@EXTFLASH.bin"
```

### curl — download the result

```bash
curl -o KP.xdf "https://ecuforge.byst.re/dl/aZ3xQ9kLmPqRtVwXyB2cFhJn"
```

### JavaScript (browser, no dependencies)

```javascript
async function convertKpToXdf(kpFile, binFile) {
  const form = new FormData();
  form.append("file", kpFile);
  if (binFile) form.append("bin", binFile);

  const res = await fetch("https://ecuforge.byst.re/api/v1/convert/kp-to-xdf", {
    method: "POST",
    body: form,
  });
  const data = await res.json();
  if (data.status !== "ok") {
    throw new Error(`${data.code}: ${data.message}`);
  }
  return data; // data.download_url, data.summary, ...
}
```

### PHP (plain cURL, no library required)

```php
<?php
$ch = curl_init("https://ecuforge.byst.re/api/v1/convert/kp-to-xdf");
curl_setopt($ch, CURLOPT_POST, true);
curl_setopt($ch, CURLOPT_POSTFIELDS, [
    "file" => new CURLFile("KP.kp"),
    // "bin" => new CURLFile("EXTFLASH.bin"), // optional
]);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);

$response = json_decode(curl_exec($ch), true);
curl_close($ch);

if ($response["status"] !== "ok") {
    throw new Exception("{$response["code"]}: {$response["message"]}");
}

echo $response["download_url"];
```
