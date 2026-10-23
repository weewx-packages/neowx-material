# Live updates with MQTT

NeoWX Material receives live readings in visitors' browsers over MQTT
WebSockets. It requires a separate WeeWX publisher and a broker with both a
TCP MQTT listener for WeeWX and a WebSocket listener for browsers.

The browser's default message format is a JSON object. Each payload needs
`dateTime` in Unix seconds unless you explicitly disable timestamp checks.
Individual scalar messages such as `weewx/outTemp 15.4` do not supply that object.

## Install the publisher

The fixed archive in this example is Rich Bell's MQTTPublish 1.2.0. Its
configuration is `[MQTTPublish]`, separate from Matthew Wall's older
`[StdRESTful][[MQTT]]` publisher configuration.

Use the Python environment running WeeWX:

```sh
python -m pip install paho-mqtt
weectl extension install https://github.com/weewx-packages/catalog/releases/download/mqtt-publish/v1.2.0/mqtt-publish-1.2.0.tar.gz
```

Add `--config /path/to/weewx.conf` when needed. Package installations may
require `sudo`; activate WeeWX's virtual environment for pip installations.
The installer registers `user.mqttpublish.MQTTPublish` in `restful_services`.
Its service and example topic start disabled. The skin is installed separately
as described in the [README](../README.md#install-and-activate).

## Configure the broker

Use an existing broker, or configure Mosquitto using its
[documentation](https://mosquitto.org/documentation/). The examples below assume
TCP MQTT on port 1883 and WebSockets on port 9001. Use your broker's actual
host, ports, WebSocket path and TLS settings.

Create a publisher account allowed to write `weewx/loop` and a browser account
allowed to read it. Browsers receive the configured username and password, so
use a read-only account there. An HTTPS weather page needs a secure WebSocket
endpoint with a certificate accepted by the browser.

## Configure WeeWX

Merge this into the installer-created section in `weewx.conf`. Replace the
host and credentials. Keep the existing engine services and any other topics:

```ini
[MQTTPublish]
    enable = true
    host = localhost
    port = 1883
    username = weewx
    password = YOUR_PUBLISHER_PASSWORD
    [[topics]]
        [[[weewx/loop]]]
            publish = true
            binding = loop
            type = json
            unit_system = METRIC
            append_unit_label = false
            retain = true
```

This publishes one JSON object per LOOP packet to `weewx/loop`. Field names
remain `dateTime`, `outTemp`, `barometer`, etc. The publisher converts to
METRIC units, so temperature is Celsius, speed is km/h and rain is cm.
A LOOP packet may contain only a subset of observations.

Restart WeeWX after applying its configuration:

```sh
sudo systemctl restart weewx
```

## Configure the skin

In the installed `skin.conf`, merge the following into `[Extras][[MQTT]]`.
Change the broker hostname to one reachable from visitors' browsers, not just
from the WeeWX host:

```ini
[Extras]
    [[MQTT]]
        enabled = true
        debug = false
        message_format = json
        message_timestamp_field = dateTime
        host = mqtt.example.com
        port = 9001
        topic = "weewx/loop"
        use_ssl = true
        websocket_path = /mqtt
        username = weewx-readonly
        password = YOUR_BROWSER_PASSWORD
        flash_on_update = true
        flash_color = "#00ff00"
        [[[SensorMapping]]]
            outTemp = "outTemp", "°C", 1
            inTemp = "inTemp", "°C", 1
            outHumidity = "outHumidity", "%", 0
            barometer = "barometer", " hPa", 1
            windSpeed = "windSpeed", " km/h", 1
            windGust = "windGust", " km/h", 1
            windDir = "windDir", "°", 0
            rain = "dayRain", " cm", 1
            rainRate = "rainRate", " cm/h", 2
            UV = "UV", "", 1
```

These mappings match the publisher configuration above. Override the remaining
shipped mappings similarly if you display more sensors. The shipped defaults
expect names with unit suffixes, such as `outTemp_C`, from other publisher
configurations. A mapping contains a payload field name, display unit and
number of decimal places. It does not convert the numeric value. Changing a
`cm` label to `mm` alone would display the wrong amount.

The `rain` card expects a daily total here. Map it only if your publisher
supplies `dayRain`; an individual LOOP packet's `rain` is not that total.
MQTTPublish's optional aggregate plugin can supply totals. See the
[publisher guide](https://weewx-mqtt.github.io/publish/) for its configuration.

Keep the skin's other MQTT options unless you need to change them. Use
`use_ssl = false` only for a plain WebSocket listener on an HTTP site.
Save the skin configuration, then wait for the next archive report or run:

```sh
weectl report run StandardReport
```

Use your report's actual name if it differs. Reload the page to load the
updated browser configuration. The [configuration patcher](CONFIG-PATCHER.md)
can reapply these settings after a skin update.

## Check the payload and browser

For a plain TCP listener, inspect messages with the Mosquitto client:

```sh
mosquitto_sub -h localhost -p 1883 -u weewx-readonly -P 'YOUR_BROWSER_PASSWORD' -t 'weewx/loop' -v
```

A message should contain an object similar to:

```json
{"dateTime": 1792713600, "outTemp": 15.4, "outHumidity": 65, "barometer": 1013.25}
```

Use the broker's TLS client options for a TLS listener. `mosquitto_sub`'s
ordinary TCP invocation does not test a WebSocket listener on port 9001.
Check that endpoint in the browser's developer console instead.

If the browser connects but values do not change, check the topic, JSON field
names and `dateTime`. The default timestamp check accepts readings newer than
the rendered page's timestamp. Set `debug = true` in the skin, regenerate the
report and reload to see detailed browser messages. Check the WeeWX log for
publisher errors if no messages arrive at the broker.

## Upgrade

Back up the publisher configuration and skin customizations before installing
a newer fixed archive. Recheck the payload names and units after changing
publisher versions or configurations. Upgrading the skin does not install or
configure a publisher or broker.
