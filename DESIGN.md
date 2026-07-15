# Design System — agenttop

Gewählt am 2026-07-15 via /design-consultation: 2 Runden Vergleichs-Board
(Runde 1: Stille Konsole / Werkbank / Phosphor → „more like B, white background";
Runde 2: drei helle B-Riffs → **B1 · Werkbank Hell**).
Mockups + approved.json: `~/.gstack/projects/patzaa-Agenttop/designs/design-system-20260715/`

## Product Context
- **What this is:** htop-artiges curses-TUI — die Zentrale der eigenen launchd-Agenten-Flotte (Master-Detail, Health-Klassifikation, ↵ öffnet Claude Code auf dem Agenten) + Claude-Spend.
- **Who it's for:** Dan, Single-User, täglicher Blick.
- **Space:** Terminal-Tools (htop, btop, lazygit, k9s, charm.sh-Welle).
- **Project type:** Terminal-TUI, Python curses. Kein Web.

## Merksatz (jede Entscheidung dient diesem Satz)
**„Ruhe — nichts schreit."** Der eine kranke Agent trägt das Bild; alles Gesunde
tritt zurück. Farbe ist ein Budget, das der ABWEICHUNG gehört.

## Aesthetic Direction
- **Direction:** Industrial/Utilitarian, hell („Werkbank Hell") — k9s-Disziplin auf Papier.
- **Decoration level:** minimal — das Zeichenraster ist die Dekoration.
- **Mood:** aufgeräumte Werkbank; professionell, still, sofort lesbar.

## Farbe (der Kern — alles andere folgt)
**Ansatz:** restrained. Farbe existiert an genau zwei Orten: Semantik-Glyphen/-Wörtern
und der EINEN Strukturfarbe. Fließtext bleibt achromatisch (die k9s-Lektion:
„Primärfarbe für 1–2 Dinge, Rest achromatisch").

Der Hintergrund ist IMMER das Terminal-Default (`use_default_colors`) — das Design
definiert deshalb ZWEI Paletten derselben Familie; der Zwilling greift auf dunklen
Terminals (Erkennung: nicht nötig — Truecolor-Werte sind Zielwerte fürs Terminal-Theme,
curses nutzt die 256c-Indexe, die auf beiden lesbar gewählt sind, Auswahl per
Terminal-Theme des Nutzers):

| Token | hell (primär) | dunkel (Zwilling) | 256c | 16c-Degrade | Verwendung |
|---|---|---|---|---|---|
| ink | `#2E3338` | `#C6CCD2` | 236 / 251 | default fg | 80 % des Inhalts |
| dim | `#8A9099` | `#707880` | 245 | bright-black | Metadaten, Schedule, Hints |
| faint | `#C6CBD1` | `#3B444F` | 251 / 238 | bright-black | Trennlinien, Log-Rauschen |
| accent | `#3E6E96` | `#7AA2C7` | 24 / 67 | blue | Panel-Titel, ⚡-Badge, Selektionskante `▌` |
| fail | `#C0453E` | `#E88388` | 131 | red | ✗ + FAILING/BLOCKED (+bold) |
| warn | `#A8752A` | `#ECC48D` | 136 | yellow | ⚠ + stale, Fix-Zeile |
| ok | `#4F7D3C` | `#A8CC8C` | 65 | green | ● ↻ Glyphen — NUR Glyphen, nie Text |
| sel-bg | `#E8EEF4` | `#1B2430` | 254 / 234 | reverse | Selektionsband |

Regeln:
- Zustand trägt IMMER Glyph + Farbe + Textwort — nie Farbe allein (16-Farben- und
  Farbfehlsicht-fest). Glyphen single-width: `✗ ⚠ ↻ ○ ● ?` — nie Emoji (double-width
  schert Spalten).
- `ok`-Farbe nur am Glyphen; die Zeile selbst bleibt ink/dim. Gesund darf grün
  blinken, aber nicht grün REDEN.
- Rot/bold ist FAILING vorbehalten. Kommt Rot ins Bild, ist es die Nachricht.

## Typografie (im Terminal = Gewichts-Disziplin, keine Fontwahl)
- **Font:** der des Nutzers (Terminal-Monospace). Das Design schreibt keinen vor.
- **Hierarchie:** bold = Programmname, Panel-Gegenstand, Aktions-Primärtaste ·
  normal = Inhalt · dim = Metadaten/Hints · bold+Farbe = Zustandswörter.
- Niemals underline/italic/blink (Terminal-Lotterie).

## Spacing (das Zeichenraster)
- Basiseinheit: 1 Zelle. Spalten tabellarisch, links ausgerichtet, feste Breiten.
- Dichte: comfortable — eine Leerzeile zwischen Detail-Blöcken, Label-Spalte 9 Zeichen.
- Genau EINE vertikale Trennlinie `│` (Master↔Detail), horizontale `─` nur unter
  Header und über Footer. Keine Panel-Rahmen (Rahmen sind Gewicht, nicht Struktur).

## Layout
- Master-Detail, feste Positionen (räumliches Gedächtnis = Navigation): links
  Flotte worst-first, rechts Detail (State/Schedule/Ursache/Fix/Log-Tail/Aktionen).
- Statuszeile: links Tasten (max. 5), rechts Zähler (`✗ 1 · ⚠ 1 · ● 9`).
- Selektion: sel-bg-Band + `▌`-Kante in accent, Zustandsfarben bleiben sichtbar.
- < 130 Spalten: einspaltig, Detail per Enter (bestehendes Verhalten).

## Motion
- Keine. Refresh-Tick 2 s ersetzt jede Animation; Zustandswechsel wechseln Farbe/Glyph
  hart. Ein TUI, das zappelt, widerspricht dem Merksatz.

## Decisions Log
| Date | Decision | Rationale |
|------|----------|-----------|
| 2026-07-15 | B1 „Werkbank Hell" gewählt (2 Board-Runden) | „more like B + white background is usually my default"; Merksatz „Ruhe — nichts schreit" |
| 2026-07-15 | ok-Farbe nur an Glyphen, nie an Text | Farbbudget gehört der Abweichung; k9s-Restraint |
| 2026-07-15 | Kein Emoji, nur single-width-Glyphen | double-width schert curses-Spalten (live erlebt) |
| 2026-07-15 | Heller Primär + dunkler Zwilling derselben Familie | bg ist Terminal-Default; beide Themes teilen Struktur + Glyphen |
