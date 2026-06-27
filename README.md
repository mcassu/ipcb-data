# ipcb-data

Public JSON data source for the IPCB app, published through a stable bootstrap config.

## Structure

- `ipcb-data-bootstrap/config.json`: Stable entrypoint with the current data `tag` and CDN `baseUrl`.
- `churches/index.json`: Church registry by `id`, with optional city/state `address`, `latitude`, `longitude`, and `modules.saf`.
- `churches/{churchID}/home.json`: Home screen payload + additional links/content.
- `churches/{churchID}/events/index.json`: Available event years for a church.
- `churches/{churchID}/events/{year}/index.json`: Event summary list with `id`, `title`, `date`, and `type`.
- `churches/{churchID}/events/{year}/detail/{eventID}.json`: Full event detail payload.
- `churches/{churchID}/saf.json`: SAF payload (only present when `church.modules.saf = true`).
- `missions.json`: Denomination-level missions payload, shared by all churches.
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

## Event Loading

Events are split to keep the initial payload small:

1. Fetch `churches/{churchID}/events/index.json` to get available years.
2. Fetch `churches/{churchID}/events/{year}/index.json` to list that year's events.
3. Fetch `churches/{churchID}/events/{year}/detail/{eventID}.json` only when opening an event.

## Home Payload

The home payload keeps the existing hero and highlights fields, and can include additive content blocks:

- `notices`: List of church notice messages as strings.
- `dailyVerses`: List of 30 daily verses, with `day`, `text`, and `reference`.

## SAF Payload

The SAF payload can include additive presentation fields for the SAF module:

- `subtitle`: Text shown below the SAF title.
- `verse`: Highlight verse object with `text` and formatted `reference`.
- `instagramUrl`: Optional Instagram URL. When `null` or empty, clients should hide the Instagram quick-access card.
- `leadership`: List of SAF leadership members, each with `role` and `name`.

## Missions Payload

The missions payload is global to the denomination and should be loaded from the repository root:

- `title` and `subtitle`: Main screen heading content.
- `verse`: Highlight verse object with `text` and formatted `reference`.
- `quickAccess`: List of quick-access cards with `id`, `label`, `icon`, and optional `url`.
- `mission`: Text block for the "Nossa Missão" card.
- `contribution`: Contribution card content, including `actionLabel`, optional `deepLink`, and optional donation fields.

Example yearly summary:

```json
{
  "events": [
    {
      "id": "evt-2026-0001",
      "title": "Escola Biblica Dominical",
      "date": "2026-06-07T09:00:00-03:00",
      "type": "worship"
    }
  ]
}
```

## Event Model

Event detail payloads use the full event shape.

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
  - Use `churches/index.json` to determine the selected church metadata and `modules.saf`.
  - Fetch `home.json` for the selected church.
  - Lazy load event years and summaries from `churches/{churchID}/events/`.
  - Lazy load separate payloads only for modules that still have their own file, such as `saf.json`.
  - Cache payloads locally by `tag + churchID + module`, then refresh in background.
