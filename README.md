# Nucleares Home Assistant Dashboard

This repo captures the Home Assistant configuration that drives the Nucleares reactor simulator dashboard. It combines
REST sensors that poll the public `nuclearesoa` API, helper entities, YAML dashboards, and safety-minded automations so
operators can keep the simulated plant within spec from a single Lovelace view.

## Highlights

- Mirrors the in-simulator valve, pump, and generator state inside Home Assistant for historical insight.
- Ships an opinionated Lovelace dashboard (`dashboard.yaml`) that matches the control-room layout.
- Includes actionable automations for demand following, core temperature tracking, hourly logging, and the new loop flow guard.
- Uses helpers (`input_select` / `input_number`) so operators can retarget loops or adjust setpoints without editing YAML.

## Requirements

- Home Assistant 2024.6 or newer running in YAML mode.
- API access to a running Nucleares OA backend (populate the `nucleares_backend_*` URL secrets in `secrets.yaml` with your host) plus a valid `nucleares_command_token`.
- The [`rest` integration](https://www.home-assistant.io/integrations/rest/) enabled (ships with `default_config`).
- Optional: HACS for installing custom cards referenced by `dashboard.yaml` (if any).

## Home Assistant Setup

If you're new to Home Assistant, start by installing one of the supported builds from the [official installation guide](https://www.home-assistant.io/installation/) - Home Assistant OS on a Raspberry Pi or virtual machine is the quickest path, while Container/Core installs are ideal for existing Docker or Python environments. Once the instance is running:

1. Create an administrator account, then enable **Advanced Mode** for that profile so the **Developer Tools** and YAML reloader are available.
2. Expose the `/config` directory via the built-in File Editor add-on, Samba share, or VS Code server so you can copy the `HA_Dashboard` folder straight into your config.
3. Restart Home Assistant after the files land in `/config` (or use **Developer Tools -> YAML -> Reload** for Automations/Templates/Rest commands) to ensure the dashboard and sensors register.

For deeper background, refer to the [Home Assistant docs](https://www.home-assistant.io/docs/) and the [community guides forum](https://community.home-assistant.io/c/guides/10) for walkthroughs on networking, backups, and advanced YAML practices.

## Setup & Usage

1. Copy the `HA_Dashboard` directory into your Home Assistant config folder (usually `/config`).
2. Merge or replace `configuration.yaml` entries as needed. This file already references `automations.yaml`,
   `dashboard.yaml`, `template_sensors.yaml`, and helpers stored in `helpers.json`.
3. Copy `secrets.yaml.example` to `secrets.yaml`, then fill in your personal `nucleares_command_token`,
   `nucleares_trusted_user_id`, and all `nucleares_backend_*` URLs so REST reads/writes point at your simulator without
   committing secrets.
4. Restart Home Assistant or reload the specific YAML sections (Template, Automations, Rest commands) from
   **Developer Tools -> YAML**.
5. Open the **Nucleares** dashboard from the sidebar to monitor demand, coolant loops, and generator output.

## Required Secrets

| Key | Purpose |
| --- | --- |
| `nucleares_command_token` | Token issued by the simulator API that authorizes pump/valve write operations via `rest_command.nucleares_webserver_set`. |
| `nucleares_trusted_user_id` | The Home Assistant user-id that should bypass login when connecting from the trusted network (typically useful for kiosk displays). |
| `nucleares_backend_commands_url` | Full URL to the backend `/api/commands` endpoint used by the REST command. |
| `nucleares_backend_state_url` | URL that returns the flattened plant state payload consumed by `sensor.ws`. |
| `nucleares_installed_loops_url` | Endpoint that exposes the installed loop metadata for the `INSTALLED_LOOPS_JSON` sensor. |
| `nucleares_valve_panel_url` | Shared endpoint powering both `VALVE_PANEL_JSON` sensors. |
| `nucleares_weather_forecast_url` | Endpoint that surfaces the weather forecast payload for dashboard overlays. |

Keep these values only in `secrets.yaml`; the repository includes `secrets.yaml.example` as a template for new deployments.

## API Endpoint

All REST calls in `configuration.yaml` now pull their URLs from `secrets.yaml`. Edit the `nucleares_backend_*` entries in
`secrets.yaml` so they point at your own Nucleares backend before deploying. This keeps real hosts and tokens out of the
repository while still making the configuration drop-in ready.

## Configuration Map

```
HA_Dashboard/
|-- configuration.yaml        # Entry point that registers dashboards, sensors, and panels
|-- automations.yaml          # Operational logic (demand following, safety, logging)
|-- dashboard.yaml            # Lovelace layout for the Nucleares control room
|-- template_sensors.yaml     # Derived sensors for flow ratios, demand bands, alarms, etc.
|-- template.yaml             # Supplemental templates (legacy)
|-- helpers.json              # Definition of helper entities (loop selector, setpoints)
\-- .storage/                 # HA-managed storage (left untouched)
```

## Helpers

| Helper | Entity | Purpose |
| --- | --- | --- |
| Loop selector | `input_select.loop` (`_0_`, `_1_`, `_2_`) | Focuses automations and dashboard widgets on a specific coolant loop. |
| Target core temp | `input_number.target_core_temp` | Operator-entered reactivity setpoint used by the **Core Target Temp** automation. |
| Timestamp counter | `input_number.timestamp` | Bumped hourly for lightweight historian widgets or templated annotations. |

Update helpers through **Settings -> Devices & Services -> Helpers** if you need more loops or alternate temperature ranges.

## Automations

- **Demand Following** (`automations.yaml`): Adjusts pump speeds and steam valves through the exposed REST command so plant
  output stays within +/-5% of grid demand. Sends persistent notifications after each corrective action.
- **Core Target Temp**: Calculates a reactivity delta between the reactor core temperature and the operator's target, then
  nudges control rod positions to converge on the setpoint.
- **Hourly Timestamp**: Resets the `input_number.timestamp` counter at the top of every hour so dashboards can show elapsed
  time annotations without relying on history statistics.
- **Loop Flow Ratio Guard** *(new)*: Watches `sensor.loop*_flowratio` values and, if any loop's secondary/primary flow ratio
  drifts outside the safe 3.0-4.0 band for at least 10 seconds, automatically focuses the dashboard on that loop and raises
  a persistent notification with current pump capacities. Use it to triage coolant misconfiguration before the demand-following
  loop saturates.

Feel free to clone these patterns for other safety checks (for example, condenser temperature, boron dosing, generator breaker state).

## Operating Tips

- The REST sensors poll every 1-10 seconds. Keep an eye on Supervisor resource usage if you increase the scan intervals.
- When editing demand limits or flow windows, adjust both the template sensors (`template_sensors.yaml`) and the related
  automations so notifications remain accurate.
- The dashboard expects metric units. If you swap to imperial values in the simulator, mirror those adjustments in the
  template sensors and helper ranges.

## Screenshots

<img width="3614" height="1662" alt="Screenshot 2025-07-03 232332" src="https://github.com/user-attachments/assets/124928a2-92f2-42bc-8f7c-ae287e3b92c8" />
<img width="1769" height="641" alt="Screenshot 2025-07-04 000629" src="https://github.com/user-attachments/assets/24a93e44-aa48-4955-959f-66addcd9ff8c" />
<img width="1781" height="1287" alt="Screenshot 2025-07-04 002656" src="https://github.com/user-attachments/assets/a64e8358-1e2b-4d36-b3e5-e3a44683304f" />
<img width="1035" height="516" alt="Screenshot 2025-07-04 002841" src="https://github.com/user-attachments/assets/0773d206-5e17-49e4-8258-36329182e427" />
<img width="2444" height="1287" alt="Screenshot 2025-07-04 004920" src="https://github.com/user-attachments/assets/79e15bbc-3ff6-4fff-9bf4-68d3428ec002" />


