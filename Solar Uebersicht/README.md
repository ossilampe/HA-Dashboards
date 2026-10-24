# Solar Übersicht – Home-Assistant-Dashboard im Victron-Stil

Dashboard für eine Victron-Anlage mit Pylontech-Speicher und Lynx-Verteilern – Energiefluss, MPPT, MultiPlus, Akkus und Sicherungen, jeweils mit eigener Handy-Ansicht.

![Pylontech und Lynx (V2)](V2/docs/desktop-4-pylontech-lynx.png)

## Versionen

| Version | Stand | Was ist drin |
|---|---|---|
| [**V2**](V2) (aktuell) | 24.10.2026 | Pylontech-Module mit SOC, Spannung, Strom, Zellspannung und Temperatur je Modul (über ESPHome), Mindest-SoC als Schieberegler mit Sicherheitsabfrage, neue Handy-Ansicht der Module |
| [**V1**](V1) | 18.10.2026 | Erste Version: Module nur mit SOC, Mindest-SoC über −/+ |

Beide Versionen sind vollständig – einfach den Ordner der gewünschten Version öffnen und der Anleitung dort folgen. V1 ist zusätzlich als Git-Tag [`solar-uebersicht-v1`](https://github.com/ossilampe/HA-Dashboards/tree/solar-uebersicht-v1/Solar%20Uebersicht) gesichert.

## Zugehörige ESPHome-Projekte

- [Pylontech Batterie System 2](https://github.com/ossilampe/ESP-Home-Projekte/tree/main/Pylontech%20Batterie%20System%202) – liest die einzelnen Akkumodule über die Konsole aus (für V2 nötig)
- [Lynx Monitor](https://github.com/ossilampe/ESP-Home-Projekte/tree/main/Lynx%20Monitor) – meldet den Zustand der MEGA-Sicherungen in den Lynx Distributoren
