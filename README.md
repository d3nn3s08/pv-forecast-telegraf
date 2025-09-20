# pv-forecast-telegraf
Fronius Gen24 + Grafana +Telegraf / Telegraf config and processors for Forecast.Solar daily PV forecast
Datenvisualisierung für Fronius

Haftungsausschluss: Dieses Projekt richtet sich an fortgeschrittene Nutzer, die Erfahrung mit Datenvisualisierung und Fronius-Systemen haben. Es wird keine Unterstützung oder Garantie geboten. Nutzung auf eigene Verantwortung.

Dieses README beschreibt, wie Daten von einem Fronius GEN24 4.0 Wechselrichter und einem Smart Meter TS 65A-3 in InfluxDB erfasst und mit Grafana visualisiert werden.

Fronius Setup

Wechselrichter: Fronius GEN24 4.0 (Firmware >= ROW 1.36.6-1)

Fronius JSON API (v1) aktiviert. Details siehe [Fronius Solar API Dokumentation]([url](https://www.fronius.com/en/solar-energy/installers-partners/technical-data/all-products/system-monitoring/open-interfaces/fronius-solar-api-json-))
.

Smart Meter: TS 65A-3

Messort: 0 – Netzanschlusspunkt (Hauptzähler)

Speicher: BYD HVS 5.1

Hinweis: Alle Grafana-Konfigurationen zu Minimal- und Maximalwerten sind auf dieses Setup abgestimmt.

Telegraf, InfluxDB und Grafana Installation

Die Einrichtung des TIG-Stacks (Telegraf, InfluxDB, Grafana) kann über beliebige Anleitungen erfolgen, z. B. diese Suche
.

InfluxDB

Es wurden folgende Buckets in InfluxDB angelegt:

inverter

pvforecast

energyprices

Datenimport

Telegraf importiert die Daten aus der Fronius JSON API und Forecast-Daten von forecast.solar
. Die vollständige Konfiguration befindet sich in telegraf.conf
.

Energiedaten

Passe die InfluxDB-Ausgabe in Telegraf nach Bedarf an:
`[[outputs.influxdb_v2]]
    urls = ["http://127.0.0.1:8086"]
    token = "change_me"
    organization = "default"
`Hinweis: Die IP des Symo GEN24 4.0 wird in meinem Netzwerk über den Hostnamen inverter aufgelöst. Entweder du richtest dein Netzwerk genauso ein oder änderst die IP in der Konfiguration.

Forecast

Passe ebenfalls die Links zu api.forecast.solar
 an, da in der Vorlage nur Platzhalter stehen. Auf dem Dashboard wird derzeit nur die Metrik Wattstunden pro Tag als Erwarteter Ertrag angezeigt.

API Dokumentation für Schätzungen

Bestimmung des Azimuts

Energiepreise

Die Energiepreise werden benötigt, um Einsparungen zu berechnen. Die Genauigkeit ist auf tägliche Updates begrenzt. Die Preise werden per CSV über die InfluxDB-Importfunktion eingespielt. Beispiel CSV-Format:

`#group,false,false,true,true,false,false,true,true,true
#datatype,string,long,dateTime:RFC3339,dateTime:RFC3339,dateTime:RFC3339,double,string,string,string
#default,mean,,,,,,,,
,result,table,_start,_stop,_time,_value,_field,_measurement,unit
,,0,2023-04-01T00:00:00+02:00,2023-04-01T00:00:00+02:00,2023-04-01T00:00:00+02:00,14.457,sell,electricity,Cent/kWh
,,0,2023-07-01T00:00:00+02:00,2023-07-01T00:00:00+02:00,2023-07-01T00:00:00+02:00,13.691,sell,electricity,Cent/kWh
,,0,2021-10-02T00:00:00+02:00,2021-10-02T00:00:00+02:00,2021-10-02T00:00:00+02:00,21.211,buy,electricity,Cent/kWh
`
Grafana
Plugins

Installiere folgende Grafana-Plugins:

Infinity by Sriramajeyam Sugumaran
Sun and Moon by fetzerch
[Infinity by Sriramajeyam Sugumaran](https://grafana.com/docs/plugins/yesoreyeram-infinity-datasource/latest/)

[Sun and Moon by fetzerch](https://github.com/fetzerch/grafana-sunandmoon-datasource)

Datenquellen

Konfiguriere folgende Datenquellen:

InfluxDB

InfluxDB auf Bucket energyprices

InfluxDB auf Bucket inverter

InfluxDB auf Bucket pvforecast

Hinweis: Query Language muss auf InfluxQL gesetzt werden.
Hinweis: Verwende für den Zugriff den Header Authorization mit dem Wert Token YOUR_TOKEN. Ersetze YOUR_TOKEN durch dein InfluxDB-Token. (InfluxDB v2 API Dokumentation
)

Hinweis: Eventuell erscheint die Meldung „Database not found“. Mappe dann InfluxDB v2 Buckets zu v1-Datenbanken. Siehe Setting up InfluxDB v2 (Flux) with InfluxQL in Grafana
 für Details. Bucket-IDs erhält man mit influx bucket list.

Weitere Quellen

Infinity Datasource (keine weitere Konfiguration)

Sun and Moon (Latitude/Longitude anpassen)

Hinweis: Die IP des Symo GEN24 4.0 wird in meinem Netzwerk über inverter aufgelöst. Passe ggf. Power Flow-Panel und Battery & Grid-Panel an deine IP an.

Dashboard

Die Dashboards können aus folgenden Dateien importiert werden:


[adfasdasda](url)
[dasdasdasda](url)

Import in Grafana:

Grafana öffnen.

Navigiere zu Dashboards > Neu > Import.

Datei hochladen.

Standardmäßig zeigt das Dashboard den aktuellen Tag und aktualisiert alle 5 Sekunden. Kurze Refresh-Zeit, weil das Power Flow-Panel Werte direkt aus der Fronius JSON API abruft.



Dashboard Screenshots


Credits
Grafana

Grafana

Basierend auf: [Powerwall Dashboard by jasonacox]([url](https://github.com/jasonacox/Powerwall-Dashboard))

Telegraf und Influx

Telegraf

[InfluxDB](url)

Basierend auf:

[chpro](url)

[Fronius-to-Influx by szymi-](url)

S[olar Panel Monitoring with Telegraf](url)

H[ow I Created a Telegraf Plugin to Monitor Solar Panels](url)

Readme Verbesserungen

Diese README-Version wurde von ChatGPT für bessere Lesbarkeit und klare Formatierung überarbeitet.
