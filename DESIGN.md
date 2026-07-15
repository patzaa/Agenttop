# Design System — agenttop

Gewählt am 2026-07-15 via /design-consultation, 2 Runden Vergleichs-Board
(Runde 1: Stille Konsole / Werkbank / Phosphor → „more like B, white background";
Runde 2: drei helle B-Riffs). Erst B1 festgeschrieben, dann auf Dans Zuruf
revidiert auf **B2 · Papier — „Notion im Terminal"**.
Mockups + approved.json: `~/.gstack/projects/patzaa-Agenttop/designs/design-system-20260715/`

## Product Context
- **What this is:** htop-artiges curses-TUI — die Zentrale der eigenen launchd-Agenten-Flotte (Master-Detail, Health-Klassifikation, ↵ öffnet Claude Code auf dem Agenten) + Claude-Spend.
- **Who it's for:** Dan, Single-User, täglicher Blick.
- **Space:** Terminal-Tools (htop, btop, lazygit, k9s, charm.sh-Welle).
- **Project type:** Terminal-TUI, Python curses. Kein Web.

## Merksatz (jede Entscheidung dient diesem Satz)
**„Ruhe — nichts schreit."** Der eine kranke Agent trägt das Bild; alles Gesunde
tritt zurück. Farbe ist ein Budget, das ausschließlich der ABWEICHUNG gehört.

## Aesthetic Direction
- **Direction:** B2 · Papier — Notion im Terminal. Warmes Weiß, near-monochrome,
  Farbe nur für „braucht dich".
- **Decoration level:** minimal — das Zeichenraster ist die Dekoration.
- **Mood:** ein ruhiges Blatt Papier, auf dem genau eine Stelle rot angestrichen ist.

## Farbe (der Kern — alles andere folgt)
**Ansatz:** restrained, radikaler als Werkbank: **kein Grün, keine Strukturfarbe.**
Gesund/erholend/transient sind gedimmte Glyphen. Es gibt genau zwei Chroma:
fail-Rot und warn-Bernstein — beide gedämpft (nie #FF0000, „sieht billig aus").

Der Hintergrund ist IMMER das Terminal-Default (`use_default_colors`). Zielwerte
fürs Terminal-Theme (hell primär, dunkler Zwilling = dieselbe These auf dunkel);
curses selbst nutzt die 256c-Indexe, die auf BEIDEN lesbar gewählt sind:

| Token | hell (primär) | dunkel (Zwilling) | 256c | 16c-Degrade | Verwendung |
|---|---|---|---|---|---|
| ink | `#40403C` | `#B8B8B8` | default fg | default | 80 % des Inhalts |
| dim | `#98968E` | `#6E6E6E` | 245 | default (stumm!) | Metadaten, Hints, gesunde Glyphen |
| faint | `#D4D2C8` | `#4A4A4A` | 250 | bright-black | Trennlinien, Log-Rauschen |
| fail | `#B5433C` | `#E88388` | 131 | red | ✗ + FAILING/BLOCKED (+bold) |
| warn | `#9C7226` | `#D7BA7D` | 136 | yellow | ⚠ + stale, Fix-Zeile |
| sel | `#F0EEE6`-Band | `#1F1F1F`-Band | A_REVERSE | A_REVERSE | Selektion (Terminal-ehrlich) |

Regeln:
- **Grün heißt lebendig, grau heißt AUS** (Dan-Override 2026-07-15): healthy und
  recovering = weiches Grün (65 / `#5F875F`-Klasse, k9s-Stil — nie Neongrün).
  Grau (dim) ist für **disabled** (per `x` bewusst abgeschaltet) und transient
  reserviert — ein grauer gesunder Agent sähe aus wie ein abgeschalteter.
- Zustand trägt IMMER Glyph + Farbe + Textwort — nie Farbe allein (16-Farben- und
  Farbfehlsicht-fest). Glyphen single-width: `✗ ⚠ ↻ ○ ● -` — nie Emoji.
- Rot/bold ist FAILING/DEAD vorbehalten. Kommt Rot ins Bild, ist es die Nachricht.
- Fokus trägt das GEWICHT (bold vs. dim), nicht die Farbe.

## Typografie (im Terminal = Gewichts-Disziplin, keine Fontwahl)
- **Font:** der des Nutzers (Terminal-Monospace). Das Design schreibt keinen vor.
- **Hierarchie:** bold = Programmname, fokussierter Panel-Titel, Aktions-Primärtaste ·
  normal = Inhalt · dim = Metadaten/Hints/gesunde Glyphen · bold+Farbe = Zustandswörter.
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
- Selektion: A_REVERSE (auf jedem Terminal-Theme korrekt); Zustandsfarbe der
  Zeile bleibt am Glyph erkennbar.
- < 130 Spalten: einspaltig, Detail per Enter (bestehendes Verhalten).

## Motion
- Keine. Refresh-Tick 2 s ersetzt jede Animation; Zustandswechsel wechseln Farbe/Glyph
  hart. Ein TUI, das zappelt, widerspricht dem Merksatz.

## Decisions Log
| Date | Decision | Rationale |
|------|----------|-----------|
| 2026-07-15 | B1 „Werkbank Hell" gewählt (2 Board-Runden) | „more like B + white background is usually my default" |
| 2026-07-15 | **Revision: B1 → B2 „Papier"** (Dans Zuruf „use design B the notion for terminal") | Notion-These konsequent: kein Grün, keine Strukturfarbe — reinste Form des Merksatzes |
| 2026-07-15 | ok-Zustände ganz ohne Chroma (dim statt grün) | Farbbudget gehört der Abweichung; Grün wäre noch ein Schrei |
| 2026-07-15 | **Override: healthy = weiches GRÜN, grau = INAKTIV** (Dan, am Live-Screenshot) | „Gray looks like it's inactive" — mit dem neuen `x`-Toggle (an/aus) braucht grau die Bedeutung „bewusst aus"; neuer State `disabled` (`-`, dim, sortiert ganz unten) |
| 2026-07-15 | Fokus = Gewicht (bold/dim), nicht Farbe | Cyan-Titel widersprach der These; Gewicht ist terminal-ehrlich |
| 2026-07-15 | Selektion = A_REVERSE statt Theme-Band | bg ist Terminal-Default — ein festes Band bricht auf dem jeweils anderen Theme |
| 2026-07-15 | Kein Emoji, nur single-width-Glyphen | double-width schert curses-Spalten (live erlebt) |
