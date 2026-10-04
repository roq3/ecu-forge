# Changelog

All notable changes to the public ECU Forge API are documented here. Format loosely follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [1.1.0] — 2026-10-04

### Added

- `POST /api/v1/decrypt/xdf` — decrypt an encrypted TunerPro `.xdf` file (RC4) back into plain XML. The decryption key is held server-side, so no extra fields are needed. New error codes `not_encrypted` and `decryption_failed` (both `422`). Results are delivered through the same single-use `GET /dl/{token}` download links as the converter.
- No-code web tool for the same operation at [`/decrypt-xdf`](https://ecuforge.byst.re/decrypt-xdf).

## [1.0.0] — 2026-09-20

Initial public release.

### Added

- `POST /api/v1/convert/kp-to-xdf` — convert a WinOLS `.kp` file (with an optional `.bin` firmware image for higher-confidence results) to a TunerPro `.xdf` file.
- `GET /dl/{token}` — single-use, time-limited download link for a conversion result.
- Fully open CORS (`Access-Control-Allow-Origin: *`) — callable directly from browser JavaScript on any domain.
- No authentication, no API keys, no rate-limited account tiers.
- Interactive Swagger UI (`/api/docs`), ReDoc (`/redoc`), and OpenAPI spec (`/openapi.json`).
