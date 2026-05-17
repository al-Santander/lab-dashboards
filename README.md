# lab-dashboards

Source-of-truth for Lab Workspace observer dashboard descriptors. Consumed by `labg-observer` through the `labg-versioning` `dashboards` domain (HL-964).

## File shape

Each file is one dashboard, keyed by the filename slug (e.g. `overview.json` → slug `overview`). The wire shape mirrors the `PUT /api/v2/observer/dashboards/:slug` request DTO:

```json
{
  "name": "Platform Overview",
  "descriptor": {
    "kind": "dashboard",
    "timeRange": { "from": "now-1h", "to": "now", "refreshIntervalMs": 30000 },
    "root": { "id": "root", "type": "panel:grid", "props": { "...": "..." } }
  }
}
```

- `name` populates the `ObserverDashboard.name` row column for display.
- `descriptor` is schema-validated against `descriptive-components/schema.json` served by labg-settings.
- `descriptor` root keys are closed: only `{kind, root, schemaVersion, timeRange, formDefaults}` are declared (`unevaluatedProperties: false`). Any other key → 422 `VALIDATION_ERROR`.
- `transform` ops live on `dataSource`, not on widget `props` (HL-960).

See `home-lab/docs/reference/guides/descriptor-dashboard-authoring` (Knowledge Module) for the full authoring guide.

## Sync

`labg-versioning` pulls this repo on its configured interval (default 60 min on the `dashboards` domain) and fans changes to `labg-observer` via the `lab.versioning.dashboards.content.updated` NATS event. labg-observer revalidates + upserts on every refresh — its dashboard row is the runtime cache, this repo is the audit log + cold-boot seed.
