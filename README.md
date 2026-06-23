# Prospect Missions Tweak

Macht die generischen **"Bezug aus lokalen Regionen"**-Missionen
(`GM_Find_Resources`) in X4: Foundations 9.x wieder spielbar.

## Problem

Die Mission verlangt eine Ressource mit z.B. ★★★★. Man findet Fundorte, die in
der Sonden-Anzeige genau diese Sternzahl haben — die Mission meldet aber
weiterhin *"nicht genügend Ressourcen"*.

Ursache: Intern wird auf einer **0–15-Rating-Skala** verglichen
(1 voller Stern = 3 Punkte), die UI rundet die Sterne aber. Eine Sonde mit
Rating 11 sieht aus wie 4 Sterne, liegt aber unter dem geforderten Rating 12.
Außerdem wertet das Spiel nur **neu gestartete** Sonden, nicht bereits
platzierte.

## Was der Mod tut

- **Toleranz** im Erfolgsvergleich (Default **-2** Rating-Punkte ≈ ⅔ Stern):
  eine Sonde besteht, wenn ihr Rating bis zu 2 Punkte unter dem geforderten
  liegt.
- **Bereits platzierte** spielereigene Ressourcensonden im Zielsektor werden
  ebenfalls geprüft (Loop alle 300 s, abschaltbar).
- Wirkt auch auf **bereits laufende** Missionen.

## Konfiguration

Mit installiertem [`sn_mod_support_apis`](https://steamcommunity.com/sharedfiles/filedetails/?id=2042901274)
(Simple Menu API) erscheint unter **Einstellungen → Erweiterungen →
Prospect Missions**:

- *Nachsicht* (0 = streng/Vanilla … 6 = sehr leicht)
- *Bereits platzierte Sonden mitzählen* (an/aus)

Ohne SMAPI laufen die Defaults (Toleranz -2, vorhandene Sonden an).

## Save-Sicherheit

`save="false"` — der Mod ist jederzeit entfernbar, der Spielstand lädt dann mit
Vanilla-Verhalten weiter. Es werden nur Spiel-Scripts per `<diff>` gepatcht.

## Technik

Siehe [`docs/plans/2026-06-23-prospect-missions-design.md`](docs/plans/2026-06-23-prospect-missions-design.md).
