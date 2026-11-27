# Weishaupt2MQTT

This add-on reads a Weishaupt WTC / Kromschröder WCM heating system over eBUS and publishes it on
MQTT under `weishaupt/…`. It bundles ebusd (the build `v26.1-weishaupt.2`, not reachable from
outside the add-on), the Weishaupt definitions `weishaupt-1.3.1` and the bridge
[weishaupt2mqtt](https://github.com/aghulas/weishaupt2mqtt), whose README is the full manual:
topics, writing, undoing, limits. This page covers the add-on only.

It creates no Home Assistant entity by itself. Entities come from the integration
[ha-weishaupt](https://github.com/aghulas/ha-weishaupt) in its Weishaupt2MQTT mode, or from MQTT
sensors you declare.

Unofficial. Checked on one installation: two WTC-GB 250 boilers in cascade, a WCM-KA 3.0 system
manager, two WCM-EM heating modules, two WCM-FS room units, a network eBUS adapter, Home Assistant
OS on a 64-bit ARM board with the Mosquitto broker add-on.

## Before you start

- An eBUS adapter reachable **over the network**. The add-on gets no serial device.
- The **Mosquitto broker** add-on and the MQTT integration.
- No other ebusd on the same adapter: stop the eBUSd add-on if it uses it.
- If you want the integration ha-weishaupt: version 1.4.1 creates its entry only against an ebusd
  add-on. Make that entry **before** you replace the ebusd add-on by this one, then switch it
  (*⋮ → Reconfigure → Weishaupt2MQTT (MQTT bridge)*). The manual, *Use it from Home Assistant*,
  gives the order.

## Set up

1. **Configuration** tab: set `ebus_device` to your adapter in ebusd's form, for example
   `ens:<address>:9999` for an enhanced-protocol network adapter. Leave the rest for a first start;
   the broker's address and credentials are found by themselves.
2. Start the add-on. It was built on your machine at install; the first start needs no download.
3. **Log** tab. About half a minute after the start:

   ```text
   Supervisor token detected, checking for MQTT service...
   Waiting for internal ebusd on 127.0.0.1:8888...
   … INFO weishaupt2mqtt.bridge: Connected to ebusd (26.1.p20261120), definitions 1.3.1, circuits=[…], checksum=present
   ```

   `circuits=` lists the device codes found (`bc1`, `bc2`, `ka`, `em1`–`em3`, `fs1`–`fs3`).
   `circuits=['broadcast']` alone with `checksum=unknown` means no device was found; lines
   `[bus error] unable to open …` every few seconds mean ebusd does not reach the adapter.
   `ERROR: EBUS_DEVICE is required unless EBUSD_EXTERNAL=true` means step 1 was skipped.
4. *Settings → Devices & services → MQTT → Configure → Listen to a topic*, `weishaupt/#`:
   `weishaupt/bridge/state` reads `online`, `weishaupt/bridge/info` lists the circuits, and one
   `weishaupt/<circuit>/state` per device follows. The first full pass over all settings and
   programs took about a quarter of an hour on the test installation.

`online` only says that the bridge is connected to the broker. To know that values are fresh, look
at `"_updated"` in `weishaupt/ka/state`.

## Options

| Option | Default | Meaning |
|---|---|---|
| `ebus_device` | *(empty)* | The adapter, in ebusd's `--device` form. Required. |
| `ebusd_address` | `31` | ebusd's own bus address. Keep `31`. |
| `ebusd_external` | `false` | `true`: use an ebusd outside the add-on (`ebusd_host`, `ebusd_port`) with these definitions loaded, instead of the bundled one. |
| `ebusd_host`, `ebusd_port` | `127.0.0.1`, `8888` | Leave as is with the bundled ebusd. |
| `mqtt_host`, `mqtt_port` | `core-mosquitto`, `1883` | The broker. No TLS. |
| `mqtt_username`, `mqtt_password` | *(empty)* | Empty: taken from the Mosquitto broker add-on. |
| `base_topic` | `weishaupt` | First level of every topic. |
| `writes_enabled` | `false` | Start value of the user-level write switch: room and hot-water setpoints, weekly programs. |
| `installer_writes_enabled` | `false` | Start value of the installer-level write switch (heating curve, limits, holiday dates, cascade…). Acts only with `writes_enabled` on; each command must also carry `"confirm": true`. Counts only until the first change at run time (below). |
| `allow_stock_ebusd` | `false` | Only with `ebusd_external` and an official ebusd. No effect with the bundled one. |
| `max_writes_per_hour` | `30` | Hourly write budget, 1 to 200. |
| `clock_sync_enabled` | `false` | **Leave off in 0.1.3**: switched on, the bridge stops publishing new values after its first pass. |
| `log_level` | `INFO` | `DEBUG`, `INFO`, `WARNING`, `ERROR`. |

The two write switches can be changed while the add-on runs (the **Writes** switch of the
integration, or a message on `weishaupt/bridge/config/set`). The add-on keeps the last state in
its data folder, and **that state wins over the two options at the next start**:
`weishaupt/bridge/info` shows what is in force. Switching the integration's **Writes** off also
switches the installer level off; to switch it on again, publish
`{"installer_writes_enabled": true}` on `weishaupt/bridge/config/set` (*Developer tools → Actions
→ `mqtt.publish`*).

## Writing, in short

Off by default. The boiler automats are never written. Each write is checked against the range of
the setting, read from the device before, written, and read back; at most `max_writes_per_hour`
per hour. Do not write while the vendor's service tool is connected to the bus. The manual gives
the topics, the payloads and how to restore a value.

## Stop, update, go back

- Stopping the add-on changes nothing on the heating system. `weishaupt/bridge/state` turns
  `offline`; the other topics keep their last values in the broker.
- An update rebuilds the add-on, definitions included. The store offers only the latest version:
  make a backup of the add-on before an update, and restore it to go back.
- Back to a plain ebusd: stop this add-on, start the eBUSd add-on, and in the integration
  *⋮ → Reconfigure → ebusd (the list, and which one to use)*.

Problems: <https://github.com/aghulas/weishaupt-ebus/issues>, with the payload of
`weishaupt/bridge/info` and the log around the problem (replace the adapter's address).
