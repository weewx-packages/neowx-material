# NeoWX Material

NeoWX Material generates a weather website from WeeWX archive data, with
interactive ApexCharts, history tables, telemetry pages, translated labels and
light/dark themes. Optional browser MQTT updates use a WebSocket broker;
Open-Meteo forecasts use your station coordinates.

This repository mirrors
[seehase/neowx-material](https://github.com/seehase/neowx-material), based on the
original [Neoground skin](https://github.com/neoground/neowx-material).
The fixed catalog archive below supplies version 1.73.4 with its original
fonts and assets. Documentation is maintained separately from the source mirror.
The catalog declares WeeWX 5.x and Python 3.10 or newer.

## Install and activate

Use an existing WeeWX station with working archive reports. The base skin
uses the Python libraries already supplied with WeeWX. You do not need
Node.js to install the published assets.

Install the published archive with WeeWX 5's extension installer:

```sh
weectl extension install https://github.com/weewx-packages/catalog/releases/download/neowx-material/v1.73.4/neowx-material-1.73.4.zip
```

Use the `weectl` from your WeeWX environment. Add
`--config /path/to/weewx.conf` when the station configuration is not found
automatically. Package installations may require `sudo`; activate the WeeWX
virtual environment for pip installations. Restart your WeeWX service after
configuration, for example `sudo systemctl restart weewx` on systemd.
The installer does not restart it for you.

The installer copies `skins/neowx-material` and the search-list modules
`historygenerator`, `openmeteo`, `updatecheck`, `onlinecheck` and `panelorder`
into `bin/user`. It changes `StandardReport` to use this skin. Check the
report explicitly before restarting:

```ini
[StdReport]
    [[StandardReport]]
        skin = neowx-material
        enable = true
        lang = en
        HTML_ROOT = /var/www/html/weewx/neowx-material
```

Use a directory writable by WeeWX and served by your web server. Wait for the
next archive interval or run `weectl report run StandardReport`.
The station driver and archive schema determine which sensor values exist.
The skin does not provide sensor drivers.

## Configure the website

The installed [`skin.conf`](skins/neowx-material/skin.conf) contains the page,
card, chart, colour and forecast settings. Use report overrides in
`weewx.conf` where possible. To keep a larger set of skin customizations,
use the optional [configuration patcher](docs/CONFIG-PATCHER.md).
Run that tool yourself when needed; installation does not schedule updates.

- [MQTT setup](docs/MQTT.md) covers the separate WeeWX MQTT publisher, broker
  and browser WebSocket connection. The skin alone does not publish MQTT.
- [Telemetry](docs/TELEMETRY-CONFIG-GUIDE.md) and
  [voltage gauges](docs/VOLTAGE-BATTERY-GAUGE-UPDATE.md) describe battery fields.
- [Multi-axis charts](docs/MULTI-AXIS-USAGE-GUIDE.md) and the
  [quick reference](docs/MULTI-AXIS-QUICK-REFERENCE.md) cover chart axes.
- [Trend indicators](docs/TREND-INDICATORS.md) explains `trend_type` and
  `show_trend_on`.

Language files are in [`skins/neowx-material/lang`](skins/neowx-material/lang).
Set `lang` on the same report you enabled. Date formatting also depends on
installed system locales. A separate Skyfield almanac extension is optional;
its installation and ephemeris requirements belong to that project's guide.

Forecast retrieval, online checks and update checks can make network requests.
External forecasts and browser MQTT need working network services. Review
these options before publishing a station on an isolated network.

## Upgrade or remove

Back up `weewx.conf` and custom skin files. Install the desired fixed archive,
reapply your settings and restart WeeWX. The patcher can help restore skin
settings, but does not replace backups. Uninstall with
`weectl extension uninstall neowx-material`, restore the desired
`StandardReport` skin and restart.

## Build styles from source

From `skins/neowx-material`, install the dependencies in its `package.json`.
Run `npm run build-css` and `npm run build-minify-css`, or use the shipped
Yarn `build` script. These commands rebuild Sass styles, not the WeeWX reports.

## License and credits

The skin retains Neoground GmbH's [MIT license](LICENSE) and the work of
seehase and contributors. Preserve the notices in bundled third-party files.
Rubik fonts include their [SIL Open Font License](skins/neowx-material/fonts/OFL.txt).
Weather Icons includes icon/font notices; ApexCharts' bundled license is in
[`js/vendor/apexcharts/LICENSE`](skins/neowx-material/js/vendor/apexcharts/LICENSE).
Open-Meteo data has its own attribution requirements. The
[upstream demonstration](https://weewx.seehausen.org/) shows one configured site.

The skin installs a shared `user.historygenerator` module. Another skin using
the same module name can replace it. Review that overlap before installing
Niculskin and NeoWX Material into the same station.
