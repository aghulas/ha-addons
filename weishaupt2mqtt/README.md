# Weishaupt2MQTT

An MQTT bridge for Weishaupt WTC condensing boilers and their Kromschröder WCM controllers
(system manager, heating modules, room units). One add-on holds ebusd, the
[Weishaupt definitions](https://github.com/aghulas/ebusd-configuration/releases) and the bridge:
it publishes decoded values, weekly programs and fault memory as JSON on MQTT, and accepts guarded
write commands, off by default.

Unofficial, not affiliated with Weishaupt, Kromschröder or the ebusd project. Checked on one
installation. How to set it up: the **Documentation** tab. Everything else (topics, writing,
undoing, limits): <https://github.com/aghulas/weishaupt2mqtt>.
