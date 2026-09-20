# Changelog

All notable changes to the public ECU Forge API are documented here. Format loosely follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [1.0.0] — 2026-09-20

Initial public release.

### Added

- `POST /api/v1/convert/kp-to-xdf` — convert a WinOLS `.kp` file (with an optional `.bin` firmware image for higher-confidence results) to a TunerPro `.xdf` file.
- `GET /dl/{token}` — single-use, time-limited download link for a conversion result.
- Fully open CORS (`Access-Control-Allow-Origin: *`) — callable directly from browser JavaScript on any domain.
- No authentication, no API keys, no rate-limited account tiers.
- Interactive Swagger UI (`/api/docs`), ReDoc (`/redoc`), and OpenAPI spec (`/openapi.json`).
