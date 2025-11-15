# Nucleares Dashboard – Fork Change Log

Summary of the modifications applied on top of `github.com/KerballOne/Nucleares` to produce this release-ready configuration bundle.

## Configuration & Secrets
- Re-enabled `default_config`, registered the `nucleares-dashboard` Lovelace YAML view, and excluded high-churn REST entities from `history`/`recorder` to keep database size under control. (`HA_Dashboard/configuration.yaml`)
- Converted `rest_command.nucleares_webserver_set` into a JSON POST with metadata, headers, and bearer token pulled from secrets, and routed **all** REST sensors through secret-driven HTTPS endpoints with richer templates instead of hard-coded LAN IPs. (`HA_Dashboard/configuration.yaml`)
- Added `HA_Dashboard/secrets.yaml.example` documenting every required secret (token, trusted user, and backend URLs) so real credentials stay out of version control.

## Template Sensors & Helpers
- Hardened template sensors against missing telemetry, defaulted all math to safe floats, and refactored the total power calculation to ignore offline breakers while iterating over generators. (`HA_Dashboard/template_sensors.yaml`)
- No structural changes were needed for `helpers.json` or `template.yaml`, so they remain byte-identical to upstream for compatibility.

## Automations
- Refactored the main demand-following automation with clearer variables, clamped pump/valve adjustments, sim-speed-aware delays, and richer persistent notifications. (`HA_Dashboard/automations.yaml`)
- Introduced the **Loop Flow Ratio Guard** automation that highlights the affected loop and raises alerts whenever the secondary/primary flow ratio leaves the 3.0–4.0 safety band for ≥10 s. (`HA_Dashboard/automations.yaml`)

## Documentation & Resources
- Rewrote `README.md` into a complete onboarding guide covering requirements, Home Assistant installation, setup instructions, secret configuration, helper/automation overviews, and operating tips with links to official documentation.
- Added `.storage/lovelace_resources` so the bundled dashboard automatically loads the ApexCharts custom card without extra manual steps.
- Included `valve.json`, an annotated snapshot of the backend valve payload, to help developers understand the data structure used by the dashboard.

Use this document when opening a PR against the upstream project or for internal release notes.
