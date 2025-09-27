 ### pv-forecast-telegraf

**Fronius Gen24 + Grafana +Telegraf / Telegraf config and processors für Forecast.Solar daily PV forecast**

> Diese grafana telegraf config leuft auf einen [Unraid](https://unraid.net/) Server
>
> Grafan selber Leuft in einer VM über Home assistant

# Datenvisualisierung für Fronius

Haftungsausschluss: Dieses Projekt richtet sich an fortgeschrittene Nutzer, die Erfahrung mit Datenvisualisierung und Fronius-Systemen haben. Es wird keine Unterstützung oder Garantie geboten. Nutzung auf eigene Verantwortung.

Dieses README beschreibt, wie Daten von einem Fronius GEN24 4.0 Wechselrichter und einem Smart Meter TS 65A-3 in InfluxDB erfasst und mit Grafana visualisiert werden.

- Fronius Setup

+ Wechselrichter: Fronius GEN24 4.0 (Firmware >= ROW 1.38.7-1 Standt 27.09.2025)

+ Fronius JSON API (v1) aktiviert. Details siehe [Fronius Solar API Dokumentation](https://www.fronius.com/en/solar-energy/installers-partners/technical-data/all-products/system-monitoring/open-interfaces/fronius-solar-api-json-)
.

+ Smart Meter: TS 65A-3

+ Messort: 0 – Netzanschlusspunkt (Hauptzähler)

+ Speicher: BYD HVS 5.1

>Hinweis: Alle Grafana-Konfigurationen zu Minimal- und Maximalwerten sind auf dieses Setup abgestimmt.

# Telegraf, InfluxDB und Grafana Installation

Die Einrichtung des TIG-Stacks (Telegraf, InfluxDB, Grafana) kann über beliebige Anleitungen erfolgen, z. B. diese Suche
.

InfluxDB

- Es wurden folgende Buckets in InfluxDB angelegt:

+ inverter
  
+ home_assistant (optional)

+ Server (optional)

+ smarthome (optional)

+ pvforecast

+ Telegraf

> Hinweis: Telegraf importiert die Daten aus der Fronius JSON API und Forecast-Daten von forecast.solar, so wie weiter Daten vom Server 
. Die vollständige Konfiguration befindet sich in telegraf.conf nicht Gewünschte Optionen sind mit #  zu Deaktiviren
>
> so wie eine überwarung von Telegraf um fehler zu finden

<img width="1644" height="905" alt="Screenshot 2025-09-26 180816" src="https://github.com/user-attachments/assets/6e840aa8-0e49-45ca-a18d-5b1fcacfa0bb" />


Passe die InfluxDB-Ausgabe in Telegraf nach Bedarf an:
`[[outputs.influxdb_v2]]
    urls = ["http://127.0.0.1:8086"]
    token = "change_me"
    organization = "default"
`Hinweis: 

# Forecast

Passe ebenfalls die Links zu api.forecast.solar
 an, da in der Vorlage nur Platzhalter stehen. Auf dem Dashboard wird derzeit nur die Metrik Wattstunden pro Tag als Erwarteter Ertrag angezeigt.

[API Dokumentation für Schätzungen](https://forecast.solar/)

[Bestimmung des Azimuts](https://www.suncalc.org/#/50.7896,10.0052,9/2025.09.20/09:22/1/0)

# Energiepreise

Die Energiepreise werden benötigt, um Einsparungen zu berechnen. Die Genauigkeit ist auf tägliche Updates begrenzt. Die Preise werden per CSV über die InfluxDB-Importfunktion eingespielt. Beispiel CSV-Format:

```csv
#group,false,false,true,true,false,false,true,true,true
#datatype,string,long,dateTime:RFC3339,dateTime:RFC3339,dateTime:RFC3339,double,string,string,string
#default,mean,,,,,,,,
,result,table,_start,_stop,_time,_value,_field,_measurement,unit
,,0,2023-04-01T00:00:00+02:00,2023-04-01T00:00:00+02:00,2023-04-01T00:00:00+02:00,0.0786,sell,electricity,Cent/kWh
,,0,2023-07-01T00:00:00+02:00,2023-07-01T00:00:00+02:00,2023-07-01T00:00:00+02:00,0.0786,sell,electricity,Cent/kWh
,,0,2021-10-02T00:00:00+02:00,2021-10-02T00:00:00+02:00,2021-10-02T00:00:00+02:00,00.40,buy,electricity,Cent/kWh
```
<img width="2140" height="765" alt="energy_cost_dashboard" src="https://github.com/user-attachments/assets/ac5877b3-93fc-429f-bae8-c65ebc184bd1" />



>Hinweis: Influxdb Line Protocol habe ich genutzt da ich feste preiße nutze bzw. habe eine Datei so wie ein script für die automation ist in Arbeit
>
>Eine umrechnung ist nicht Erfoderlich aus man gibt sell und buy in € an und nicht wie ich in cent zb 12.14€
>dann ist bei der kosten berechnung diesen code 
>
>```SELECT last("sell")/100 as sellEuroWh, last("buy")/100 as buyEuroWh FROM "autogen"."electricity" GROUP BY time(1d) fill(previous) tz('${tz:raw}')```
>
>durch ```SELECT last("sell")/10000 as sellEuroWh, last("buy")/10000 as buyEuroWh FROM "autogen"."electricity" GROUP BY time(1d) fill(previous) tz('${tz:raw}')```
>zu erstezen!

[Grafana](https://grafana.com/)

[Plugins](https://grafana.com/grafana/plugins/)



**Installiere folgende Grafana-Plugins:**



[Sun and Moon by fetzerch](https://grafana.com/grafana/plugins/fetzerch-sunandmoon-datasource/)

[Infinity by Sriramajeyam Sugumaran](https://grafana.com/docs/plugins/yesoreyeram-infinity-datasource/latest/)


# Datenquellen

Konfiguriere folgende Datenquellen:

InfluxDB

+ inverter
  
+ home_assistant (optional)

+ Server (optional)

+ smarthome (optional)

+ pvforecast

+ Telegraf


> **_NOTE:_** Query Language muss auf InfluxQL gesetzt werden.
Verwende für den Zugriff den Header Authorization mit dem Wert Token YOUR_TOKEN. Ersetze YOUR_TOKEN durch dein InfluxDB-Token. (InfluxDB v2 API Dokumentation)

<img width="838" height="1188" alt="datasource_settings" src="https://github.com/user-attachments/assets/9acef4c3-4d3c-4170-8812-ffa9280d845c" />



>Hinweis: Eventuell erscheint die Meldung „Database not found“. Mappe dann InfluxDB v2 Buckets zu v1-Datenbanken. Siehe Setting up InfluxDB v2 (Flux) with InfluxQL in Grafana
 für Details. Bucket-IDs erhält man mit influx bucket list.

# Weitere Quellen

Infinity Datasource (keine weitere Konfiguration notwändig)

Sun and Moon (Latitude/Longitude anpassen)

>Hinweis: Die IP des  GEN24 4.0  Power Flow-Panel und Battery & Grid-Panel an .
<img width="2177" height="446" alt="power_flow_panel_settings" src="https://github.com/user-attachments/assets/142ac868-9ed5-41a1-ab02-a3eb8dad2802" />




Die Dashboards können aus folgenden Dateien importiert werden:



Import in Grafana:

Grafana öffnen.

Navigiere zu Dashboards > Neu > Import.

Datei hochladen.

Standardmäßig zeigt das Dashboard den aktuellen Tag und aktualisiert alle 5 Sekunden. Kurze Refresh-Zeit, weil das Power Flow-Panel Werte direkt aus der Fronius JSON API abruft.



Dashboard Screenshots


Credits
Grafana

Grafana

Basierend auf: [Powerwall Dashboard by jasonacox](https://github.com/jasonacox/Powerwall-Dashboard)

[Telegraf ](https://github.com/influxdata/telegraf)

[Influx](https://www.influxdata.com/lp/influxdb-database-de/?cq_con=173719674632&cq_term=influxdb&cq_med=&cq_plac=&cq_net=g&cq_plt=gp&gad_campaignid=22102419547)



# Basierend auf:

[chpro](url)

[Fronius-to-Influx by szymi-](url)

[Solar Panel Monitoring with Telegraf](url)

[How I Created a Telegraf Plugin to Monitor Solar Panels](url)

Readme Verbesserungen

