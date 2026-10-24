# Solar Übersicht V2 – Home-Assistant-Dashboard im Victron-Stil

Ein Dashboard für Home Assistant, das eine Victron-Anlage mit Pylontech-Speicher übersichtlich darstellt – angelehnt an den Look von Victron VRM. Alle Grafiken (Energiefluss, MPPT, MultiPlus, Akku-Stapel, Lynx-Verteiler) werden live als SVG gezeichnet; es werden keine Bilddateien benötigt.

Jede Karte gibt es zweimal: eine **Desktop-Variante** (ab 768 px Breite) und eine eigene **Handy-Variante**, die Home Assistant automatisch je nach Bildschirmbreite zeigt.

> Die erste Version liegt unverändert in [`../V1`](../V1) und lässt sich weiterhin installieren.

## Neu in V2

| Bereich | Änderung |
|---|---|
| **Pylontech-Module** | Jedes der 10 Module zeigt jetzt **SOC, Spannung, Strom, Zellspannung min/max und Temperatur** – mit Spaltenköpfen und kleiner Modulnummer. Die Typbeschriftung „Pylontech US5000/US2000“ ist entfallen; rechts zeigt ein Punkt, ob das Modul Daten liefert (grün) oder nicht (grau). |
| **Datenquelle Module** | Die Einzelwerte kommen von einem eigenen ESPHome-Gerät an der Konsole des Master-Akkus: [Pylontech Batterie System 2](https://github.com/ossilampe/ESP-Home-Projekte/tree/main/Pylontech%20Batterie%20System%202). |
| **Mindest-SoC** | Der „Limit“-Balken ist jetzt ein **echter Schieberegler** (Vorlage `vic_soclim`). Beim Loslassen fragt er nach: „Mindest-SoC (ESS) von 30 % auf 45 % ändern?“ – bei *Abbrechen* springt er zurück. Die kleinen −/+-Felder aus V1 sind entfallen. |
| **Handy** | Die Module stehen untereinander über die volle Breite (vorher zwei schmale Spalten). |

## Energiefluss in Bewegung

![Animierter Energiefluss](docs/energiefluss-demo.svg)

*Demo mit Beispielwerten (sonniger Mittag): Die Punkte zeigen die Flussrichtung, und je mehr Leistung fließt, desto schneller laufen sie.*

<details>
<summary>Handy-Ansicht</summary>

<img src="docs/energiefluss-demo-mobil.svg" width="400" alt="Animierter Energiefluss, Handy-Ansicht">

</details>

## Ansichten

### Normale Ansicht (Desktop / Tablet)

| | |
|---|---|
| ![Energiefluss und Heute](docs/desktop-1-energiefluss.png) | ![MPPT](docs/desktop-2-mppt.png) |
| ![MultiPlus und Steuerung](docs/desktop-3-multiplus.png) | ![Pylontech und Lynx](docs/desktop-4-pylontech-lynx.png) |

### Handy

| | | | | |
|---|---|---|---|---|
| ![Energiefluss](docs/mobil-1-energiefluss.png) | ![MPPT](docs/mobil-2-mppt.png) | ![MultiPlus](docs/mobil-3-multiplus.png) | ![Pylontech](docs/mobil-4-pylontech.png) | ![Lynx](docs/mobil-5-lynx.png) |

*Die Bilder sind aus den Kartenvorlagen dieses Dashboards mit echten Werten der laufenden Anlage gerendert (24.10.2026).*

## So hängt alles zusammen

```mermaid
flowchart LR
    subgraph Anlage["Victron-Anlage"]
        PV["PV-Module"] --> MPPT["3× MPPT"]
        MPPT --> BUS[("DC-Bus 48 V")]
        MP["3× MultiPlus-II<br/>L1 · L2 · L3"] <--> BUS
        LYNX["3× Lynx Distributor<br/>je 4 MEGA-Sicherungen"] --- BUS
        PYL["Pylontech-Stapel<br/>2× US5000 · 8× US2000"] <--> LYNX
        CERBO["Cerbo GX"]
        PYL -- "CAN (BMS)" --> CERBO
        MPPT -. "VE.Can / VE.Direct" .-> CERBO
        MP -. "VE.Bus" .-> CERBO
    end

    PYL -- "Konsole RS232" --> ESP1["ESP<br/>Pylontech Batterie System 2"]
    LYNX -- "I²C (RJ10)" --> ESP2["ESP<br/>Lynx Monitor"]

    CERBO -- "MQTT / Modbus" --> HA["Home Assistant"]
    ESP1 -- "ESPHome-API" --> HA
    ESP2 -- "ESPHome-API" --> HA

    HA --> DASH["Dashboard<br/>Solar Übersicht V2"]
    DASH -- "Mindest-SoC · Betriebsart ·<br/>Ladestrom · Force charge" --> HA
    HA -- "number / select / switch" --> CERBO
```

### Mindest-SoC einstellen

```mermaid
sequenceDiagram
    actor Du
    participant R as Regler (vic_soclim)
    participant HA as Home Assistant
    participant GX as Cerbo GX
    Du->>R: Balken ziehen – Anzeige läuft mit (5-%-Schritte)
    Du->>R: loslassen
    R->>Du: „Mindest-SoC (ESS) von 30 % auf 45 % ändern?“
    alt OK
        R->>HA: number.set_value (45)
        HA->>GX: ESS Mindest-SoC = 45 %
        GX-->>HA: neuer Wert
        HA-->>R: Regler zeigt 45 %
    else Abbrechen
        R-->>Du: springt auf 30 % zurück, nichts wird gesendet
    end
```

## Was das Dashboard zeigt

| Bereich | Inhalt |
|---|---|
| **Energiefluss** | Netz, MultiPlus, AC-Verbraucher, Batterie, Solar und DC-Verbraucher. Bewegte Punkte zeigen die Flussrichtung, die Geschwindigkeit richtet sich nach der Leistung. |
| **Heute** | Solar erzeugt, eingespeist, aus dem Netz, Gesamtverbrauch, in den / aus dem Akku – dazu Eigenverbrauch, Autarkie und die Zählerstände Bezug / Einspeisung. |
| **Solarladeregler (MPPT)** | Je Laderegler: Zustand, PV-Leistung, PV- und Batteriespannung/-strom, Ertrag und Maximum heute, Leistung der 4 Tracker. Alarmglocke (grün / orange / rot) und Ein/Aus-Knopf. |
| **MultiPlus-II** | Je Phase L1–L3: AC-Eingang, AC-Ausgang, Wechselrichterleistung, Frequenz, DC-Bus, Scheinleistung und Alarmglocke. |
| **Steuerung MultiPlus-System** | Betriebsart Ein / Nur Laden / Nur WR / Aus, „Aus dem Netz laden“ (ESS Force charge), max. Ladestrom (DVCC) mit −/+. |
| **Pylontech Batteriespeicher** | 10 Module (2× US5000, 8× US2000) mit SOC, Spannung, Strom, Zellspannung min/max und Temperatur je Modul; SOC-Balken, Mindest-SoC (ESS) per Schieberegler mit Abfrage, Leistung, Spannung, Strom, Temperatur, SoH, Kapazität, Zellwerte, Lade-/Entladegrenzen, Restlaufzeit. |
| **Lynx** | 3 Lynx Distributor mit je 4 MEGA-Sicherungen und Beschriftung. LED grün = OK, rot blinkend = defekt. |

Alle Schaltflächen und der Mindest-SoC-Regler fragen vor dem Schalten nach.

## Voraussetzungen

- Home Assistant 2026.x mit einem **Sections**-Dashboard
- [**button-card**](https://github.com/custom-cards/button-card) (über HACS installieren)
- Victron-Daten in Home Assistant – hier über die Integrationen **victron_mqtt** (`gx_device_*`, `multiplus_ii_1_*`) und **Victron (Modbus)** (`victron_*`)
- Pylontech-Gesamtwerte über den Cerbo (hier: `pylontech_battery_id_512_*`)
- **Neu:** Einzelwerte der Module über das ESPHome-Gerät [Pylontech Batterie System 2](https://github.com/ossilampe/ESP-Home-Projekte/tree/main/Pylontech%20Batterie%20System%202) (`sensor.pylontech_batterie_system_2_battery1…10_*`)
- Lynx-Sicherungsstatus über das ESPHome-Gerät [Lynx Monitor](https://github.com/ossilampe/ESP-Home-Projekte/tree/main/Lynx%20Monitor) (`binary_sensor.lynx_monitor_lynx_a…c_fuse_1…4_broken`)

## Installation

1. **button-card** über HACS installieren und den Browser neu laden.
2. **Hilfswert anlegen:** Einstellungen → Geräte & Dienste → Helfer → Helfer erstellen → **Zahlenwert**
   - Name: `Solar Ziel Ladestrom` (ergibt `input_number.solar_ziel_ladestrom`)
   - Minimum `-1`, Maximum `1000`, Schrittweite `1`, Einheit `A`, Anzeige: Eingabefeld
3. **Automationen anlegen:** Den Inhalt von [`automations.yaml`](automations.yaml) übernehmen – entweder in die eigene `automations.yaml` einfügen oder jede Automation im Editor über „In YAML bearbeiten“ einfügen (dann die `id`-Zeile weglassen).
4. **ESPHome-Geräte einrichten:** [Pylontech Batterie System 2](https://github.com/ossilampe/ESP-Home-Projekte/tree/main/Pylontech%20Batterie%20System%202) und [Lynx Monitor](https://github.com/ossilampe/ESP-Home-Projekte/tree/main/Lynx%20Monitor). Ohne den Pylontech-ESP zeigen die Modulzeilen „--“, der Rest funktioniert trotzdem.
5. **Dashboard anlegen:** Einstellungen → Dashboards → Dashboard hinzufügen → „Neues Dashboard von Grund auf“. Dashboard öffnen → ✏️ → ⋮ → **Raw-Konfigurationseditor** → den kompletten Inhalt von [`solar-uebersicht.yaml`](solar-uebersicht.yaml) einfügen und speichern.
6. **Entitäten anpassen:** Die Entitäts-IDs passen zu dieser Anlage. Bei einer anderen Anlage im Raw-Editor mit Suchen & Ersetzen anpassen (siehe unten).

### Von V1 auf V2 umsteigen

- Im Raw-Konfigurationseditor den kompletten Inhalt durch [`solar-uebersicht.yaml`](solar-uebersicht.yaml) aus V2 ersetzen.
- Hilfswert und Automationen bleiben gleich.
- ESPHome-Gerät **Pylontech Batterie System 2** einrichten bzw. auf den aktuellen Stand flashen (Akku 1–10 mit Temperatur und Zellspannung).
- Danach die Seite neu laden (am Handy die App einmal ganz schließen).

## Wie es funktioniert

- **Vorlagen statt Einzelkarten:** Unter `button_card_templates` liegen die Bausteine. `vic_base` enthält die Hilfsfunktionen (Formatierung, SVG-Text, Rechtecke, Gerätegrafiken `MPPT`, `MULTI`, `MODULE`, Alarmglocke `BELL`, Ein/Aus-Symbol `PWR`). Die übrigen Vorlagen (`vic_flow`, `vic_energy`, `vic_mppt`, `vic_multi`, `vic_mpctl`, `vic_pylon`, `vic_lynx`, `vic_soclim`) bauen daraus jeweils eine Karte.
- **Desktop / Handy:** Jede Karte steht zweimal im Dashboard, einmal mit `variables: {m: true}`. Über `visibility` mit `screen`-Bedingung (`min-width: 768px` / `max-width: 767px`) zeigt Home Assistant die passende.
- **Schaltflächen:** Die SVG-Grafik selbst ist nicht anklickbar. Über jedem Knopf liegt eine unsichtbare button-card (`custom_fields` mit Präfix `t_`, Vorlage `vic_tap`), die per Prozentangabe genau auf das gezeichnete Symbol gesetzt ist.
- **Modulzeilen:** `MODULE` in `vic_base` zeichnet eine Zeile mit Nummer und fünf Werten; `vic_pylon` holt die Werte je Modul aus `sensor.pylontech_batterie_system_2_battery<N>_*` (SOC, Spannung, Strom, Zelle min/max, Temperatur).
- **Mindest-SoC-Regler:** `vic_soclim` ist eine eigene kleine button-card im Feld `t_lim` über dem Limit-Balken. Sie enthält ein unsichtbares `<input type="range">` über dem gezeichneten Balken, aktualisiert die Anzeige beim Ziehen und ruft nach der Abfrage `number.set_value` auf. Weil sie nur auf `number.gx_device_ess_mindest_soc_grenze` reagiert (`triggers_update`), unterbrechen die ständig aktualisierten Akkuwerte das Ziehen nicht.
- **Geräte pro Karte:** Die MPPT-Karten bekommen `variables: {id: "0" | "1" | "226", name: …}`, die MultiPlus-Karten `variables: {ph: "l1" | "l2" | "l3", dev: "0" | "1" | "2", name: …}`.
- **Sicherungsnamen:** In `vic_lynx` unter `variables.names` (je Lynx A/B/C eine Liste mit 4 Namen, max. ca. 12 Zeichen).

> **Stolperfalle button-card:** Eine button-card, deren `tap_action`, `hold_action` und `double_tap_action` alle `none` sind, schaltet Maus- und Touch-Eingaben ab. Damit der Schieberegler darin bedienbar ist, setzt `vic_soclim` ausdrücklich `pointer-events: auto`.

### Warum ein Hilfswert für den Ladestrom?

Der Cerbo meldet einen geänderten Ladestrom erst nach einigen Sekunden zurück und ignoriert gelegentlich einen Schreibbefehl. Deshalb ändern −/+ zuerst sofort `input_number.solar_ziel_ladestrom` – die Anzeige reagiert direkt und ist orange, bis der Cerbo bestätigt.

- **„Solar: Ziel-Ladestrom an Cerbo übertragen“** schreibt den Wert 1 s nach dem letzten Tippen an den Cerbo und wiederholt es bis zu 3-mal, falls er nicht übernommen wird.
- **„Solar: Ziel-Ladestrom mit Cerbo abgleichen“** zieht den Hilfswert nach, wenn der Ladestrom woanders (VRM, Cerbo-Display) geändert wird.

## Verwendete Entitäten (Auswahl)

| Zweck | Entitäten |
|---|---|
| Netz / Verbrauch | `sensor.gx_device_netz_leistung_l1…l3`, `sensor.gx_device_verbrauch_leistung_l1…l3` |
| PV / Batterie gesamt | `sensor.victron_system_dc_pv_power`, `sensor.victron_system_battery_power/_soc/_voltage/_current/_state/_time_to_go` |
| MPPT | `sensor.victron_solarcharger_*_{0,1,226}`, `select.victron_solarcharger_mode_{0,1,226}` |
| MultiPlus | `sensor.multiplus_ii_1_wechselrichter_*_l1…l3`, `sensor.multiplus_ii_1_ve_bus_gerat_{0,1,2}_inverter_leistung`, `sensor.victron_vebus_state_227`, `sensor.victron_vebus_alarm_*_227` |
| Steuerung | `select.victron_vebus_mode_227`, `switch.ess_force_charge`, `number.gx_device_ess_max_ladestrom`, `input_number.solar_ziel_ladestrom` |
| Tageswerte | `sensor.gx_device_solar_erzeugung_taglich`, `sensor.vrm_today_grid_energy_in/_out`, `sensor.vrm_today_consumption`, `sensor.gx_device_batterie_in`, `sensor.gx_device_baterie_out` |
| Zählerstände | `sensor.stromleser_ebz_e_inop`, `sensor.stromleser_ebz_e_outop` |
| Pylontech gesamt | `sensor.pylontech_battery_id_512_*`, `number.gx_device_ess_mindest_soc_grenze` |
| Pylontech je Modul | `sensor.pylontech_batterie_system_2_battery{1…10}_state_of_charge/_voltage/_current/_temperatur/_zelle_min_spannung/_zelle_max_spannung` |
| Lynx | `binary_sensor.lynx_monitor_lynx_{a,b,c}_fuse_{1…4}_broken` |

## Dateien

```
solar-uebersicht.yaml   Dashboard (für den Raw-Konfigurationseditor)
automations.yaml        die zwei Automationen für den Ladestrom
docs/                   Bilder Desktop und Handy, animierter Energiefluss
```
