# ipcb-data

Public JSON data source for the IPCB app, published through a stable bootstrap config.

## Structure

- `ipcb-data-bootstrap/config.json`: Stable entrypoint with the current data `tag` and CDN `baseUrl`.
- `churches/index.json`: Church registry by `id`.
- `churches/{churchID}/manifest.json`: Church metadata + module flags (`home`, `events`, `saf`, `more`).
- `churches/{churchID}/home.json`: Home screen payload.
- `churches/{churchID}/events.json`: Events payload.
- `churches/{churchID}/saf.json`: SAF payload (only present when `modules.saf = true`).
- `churches/{churchID}/more.json`: Additional links/content payload.
- `v1/*`: Legacy versioned payloads kept for compatibility during migration.

## Bootstrap

The app should start from the stable bootstrap config:

```txt
https://cdn.jsdelivr.net/gh/mcassu/ipcb-data@codex/ipcb-data-bootstrap/config.json
```

Current bootstrap shape:

```json
{
  "tag": "1.2.0",
  "baseUrl": "https://cdn.jsdelivr.net/gh/mcassu/ipcb-data@codex/ipcb-data-bootstrap/",
  "updatedAt": "2026-06-02T00:00:00-03:00",
  "locale": "pt-BR",
  "defaultCurrency": "BRL",
  "status": "active"
}
```

This keeps the initial URL stable while allowing the app to react to the published `tag`. New clients should not depend on `v1`, `v2`, or other version folders for the main loading flow.

## Event Model (`events.json`)

`events.json` now supports a reusable event shape through `eventTemplate`.

Core fields:

- `image.url` and `image.alt`: Event banner image.
- `name`: Event title.
- `description`: Full event description.
- `type`: Main event type (examples: `culto`, `escola-biblica-dominical`, `acampamento`, `almoco-da-familia`, `reuniao-de-oracao`, `excursao`).
- `location.latitude` and `location.longitude`: Coordinates for map display.
- `liturgy`: Use when the event has a service flow or program details.
- `modality`: `inPerson` and/or `online`, with optional `onlineUrl`.
- `donation`: `enabled` + optional PIX fields (`pixKey`, `pixKeyType`, `beneficiary`).
- `highlights`: Generic bullet points that can fit different event types.
- `categoryTags`: Search/filter tags.

## Versioning

Versioning is controlled by the bootstrap `tag`. The folder-based `v1/*` payloads are legacy compatibility files only.

## Notes

- Keep field names stable once published.
- Prefer additive changes for backward compatibility.
- Recommended app loading flow:
  - Fetch `ipcb-data-bootstrap/config.json` first.
  - Read the current `tag`, `baseUrl`, and `status`.
  - Use `selectedChurchID` directly as church key.
  - Build module paths from `churches/{churchID}/`.
  - Fetch `manifest.json` first.
  - Lazy load only enabled modules from the manifest.
  - Cache payloads locally by `tag + churchID + module`, then refresh in background.
