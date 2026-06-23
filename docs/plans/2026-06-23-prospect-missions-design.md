# Prospect Missions Tweak — Design

Datum: 2026-06-23

## Problem

Die generische Mission **"Bezug aus lokalen Regionen"** (`GM_Find_Resources`,
Spieltext-Seite 30132) ist seit X4 9.0 sehr schwer abzuschließen. Spieler finden
Ressourcen-Fundorte mit der scheinbar geforderten Sternzahl, bekommen aber
dauerhaft "nicht genügend Ressourcen" gemeldet.

## Ursachenanalyse (Vanilla-Scripts)

- `md/gm_find_resources.xml` generiert die Mission. In `Check_Space` wird die
  geforderte Bewertung auf `$Space.bestyieldrating.{ware}` gesetzt — das
  **absolute Maximum der Ressource im gesamten Sektor**.
- `md/rml_find_resources.xml` prüft den Erfolg (Cue `CheckMissionStatus`):
  ```
  $Probe.bestyieldrating.{$ware} ge $ResourceYieldRatingList.{$i}
  ```
  Verglichen wird der **exakte Rating-Wert** (0–15-Skala).
- `md/lib_generic.xml` (`GenerateRatingStarsText`): Sterne = `floor(Rating/3)`,
  d.h. **1 voller Stern = 3 Rating-Punkte**, max. 5 Sterne (Rating 15).
  Die Sonden-UI ("Sammelrate ★★★★") rundet visuell; der Vergleich nicht.
  Beispiel: Sonde Rating 11 (≈3⅔ Sterne, sieht aus wie 4) vs. gefordert 12 →
  "nicht genügend", obwohl beides wie ★★★★ aussieht.
- Zusätzlich (im Vanilla-Code als TODO vermerkt, `rml_find_resources.xml:122`):
  nur das **Starten einer neuen Sonde** löst die Prüfung aus; bereits platzierte
  Sonden werden ignoriert.

## Lösung

Kleiner Mod (`prospect_missions`), der `rml_find_resources.xml` per `<diff>`
patcht. Wirkt auch auf **bereits laufende** Missionen, da MD-Action-Logik beim
Laden neu eingelesen wird (reine Generierungs-Änderungen würden nur neue
Missionen betreffen).

1. **Toleranz im Erfolgsvergleich** (konfigurierbar, Default -2 Rating-Punkte):
   ```
   $Probe.bestyieldrating.{$ware} ge ($ResourceYieldRatingList.{$i} + $Toleranz)
   ```
2. **Vorhandene Sonden mitprüfen** (konfigurierbar, Default an): zusätzlicher
   Loop-Cue (alle 300 s) scannt spielereigene `class.resourceprobe` im
   Zielsektor und wendet dieselbe Bedingung an.

## Konfiguration

- Eigenes MD-Script `prospect_missions.xml`:
  - `PM_Init` (auf `event_game_loaded`) setzt Defaults `global.$pm_tolerance = -2`,
    `global.$pm_check_existing = true` und stellt persistierte Spieler-Auswahl
    aus `player.entity.$pm_saved_*` wieder her (dn-Muster).
  - Optionales Optionsmenü via `sn_mod_support_apis` (Simple Menu API):
    Slider "Nachsicht" (0–6 → intern 0…-6) + Checkbox "Vorhandene Sonden".
- Der Patch liest jeden Wert defensiv mit Inline-Default
  (`if global.$pm_tolerance? then … else -2`) → funktioniert auch ohne `PM_Init`
  und **ohne SMAPI**.

## Dateistruktur (Stil an `dn_distribution_network` angelehnt)

```
prospect_missions/
  content.xml                  id, version, optional dep SMAPI, save="false"
  md/prospect_missions.xml     Config-Init + SMAPI-Optionsmenü
  md/rml_find_resources.xml    <diff>: Vergleich lockern + Loop für vorhandene Sonden
  t/0001.xml | -l044 | -l049   Lokalisierung (Seite 55801)
  README.md
```

## Save-Sicherheit

`save="false"`, nur `<diff>` auf Actions/Conditions + eigenes Script. Mod jederzeit
entfernbar; Vanilla-Verhalten kehrt zurück.

## Bewusst NICHT geändert

Generierungslogik (`Check_Space`, `ObjectiveRadius`) — beträfe nur neue Missionen,
nicht die laufende; der Vergleichs-Fix ist die eigentliche Ursache.
