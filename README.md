# ha-addons — eBUSd with the run-time checksum for Weishaupt

This branch (`weishaupt`) of a fork of [LukasGrebe/ha-addons](https://github.com/LukasGrebe/ha-addons)
publishes one add-on, **eBUSd (Weishaupt CRC)**: the upstream eBUSd add-on with the ebusd build
[aghulas/ebusd v26.1-weishaupt.1](https://github.com/aghulas/ebusd/releases/tag/v26.1-weishaupt.1),
which computes the checksum that Weishaupt / Kromschröder WCM controllers expect in a register
write (john30/ebusd#1825, not merged yet). Everything else is the upstream add-on, with its
documentation ([`ebusd/DOCS.md`](ebusd/DOCS.md)).

You need it only to **write** settings with the
[Weishaupt definitions](https://github.com/aghulas/ebusd-configuration/releases) (or the Home
Assistant integration [ha-weishaupt](https://github.com/aghulas/ha-weishaupt)). For reading, the
stock add-on is enough.

## How to install

1. Add-on store → ⋮ → **Repositories** → add `https://github.com/aghulas/ha-addons#weishaupt`
   (the `#weishaupt` selects this branch).
2. Reload, install **eBUSd (Weishaupt CRC)**.
3. Stop and disable the stock **eBUSd** if you run it: one adapter, one add-on.
4. Configure it as the stock add-on (adapter, options). Its configuration folder is its own,
   `/addon_configs/<prefix>_ebusd/`: copy your definitions and `mqtt-hassio.cfg` there.
5. Check which ebusd runs: `ebusctl info` inside the add-on (SSH add-on, `docker exec <container>
   ebusctl info`); the first line reads `version: ebusd 26.1.p20261029`. The definitions' notice
   gives a check that sends nothing to the bus.

Unofficial, not affiliated with Weishaupt or Kromschröder. Updates follow upstream releases.


## [ebusd](https://github.com/LukasGrebe/ha-addons/tree/main/ebusd) 

This Add-on runs [ebusd](http://ebusd.eu), a daemon for handling communication with eBUS devices connected to a 2-wire bus system (“energy bus” used by numerous heating systems), in [Home Assistant OS](https://www.home-assistant.io/installation/raspberrypi). You can simply plugin a ebus hardware interface into your RaspberryPi, and install the addon to run ebusd.


# Thanks ...

Thank you to

- https://github.com/m-reuter
- https://github.com/john30
- https://github.com/ludeeus
- Https://github.com/tim-devel

And thanks to you for your interest in this project! If it is of any use to you, you can buy me a coffee:
https://www.paypal.com/paypalme/lukasgrebe
