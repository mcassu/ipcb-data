# ipcb-data

Public JSON data source for the IPCB app, versioned and maintained for client consumption.

## Structure

- `v1/config.json`: General app settings and metadata.
- `v1/categories.json`: Domain categories.
- `v1/items.json`: Consumable item list.

## Versioning

Data is organized by version folders (e.g. `v1`, `v2`) to avoid breaking clients.

## Notes

- Keep field names stable once published.
- Prefer additive changes for backward compatibility.
