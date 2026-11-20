# Home Assistant App: eBUSd (Weishaupt CRC)

The upstream eBUSd app of [LukasGrebe/ha-addons](https://github.com/LukasGrebe/ha-addons) with
the ebusd build [v26.1-weishaupt.1](https://github.com/aghulas/ebusd/releases/tag/v26.1-weishaupt.1)
of aghulas/ebusd, which computes the run-time checksum needed to write settings of Weishaupt
controllers (WTC, Kromschröder WCM). Needed only for writing; it cannot load the online
definitions, so give it local ones (`--configpath`). See the Documentation tab.

## What this app does

- Runs ebusd as a Home Assistant supervised app (container).
- Exposes ebusd's own command-line options as HA configuration fields.
- Auto-configures the MQTT connection to the HA MQTT broker.
- Seeds `mqtt-hassio.cfg` (MQTT–Home Assistant integration config) into the app config folder on first start.
- Optionally stores every eBUS telegram ebusd receives in an SQLite database.

## Installation

See the [repository readme](https://github.com/aghulas/ha-addons/tree/weishaupt#how-to-install) for how to add this repository to Home Assistant (store address `https://github.com/aghulas/ha-addons#weishaupt`).

## Versioning

This app version follows the upstream app (here `26.1.10`), with the suffix `-weishaupt.N` for this build. The upstream app version mirrors [john30/ebusd releases](https://github.com/john30/ebusd/releases) — `MAJOR.MINOR` matches ebusd, `.PATCH` is reserved for app-specific changes.

For the changelog, see the upstream [ebusd releases page](https://github.com/john30/ebusd/releases).

## Support

**This build, this app and the Weishaupt definitions** — use the [issues of aghulas/weishaupt-ebus](https://github.com/aghulas/weishaupt-ebus/issues).

**Configuration and device issues** — this app only runs ebusd; CSV message definition files are not managed here.
Please see the official [ebusd project](https://ebusd.eu) and [community](https://github.com/john30/ebusd/discussions).

**Issues specific to running ebusd via HA Supervisor that also occur with the stock eBUSd app** — use [Issues](https://github.com/LukasGrebe/ha-addons/issues) or [Pull Requests](https://github.com/LukasGrebe/ha-addons/pulls).
