# agenttop

Ein selbstständiges Python-curses-TUI (eine Datei: `agenttop`), htop-artig:
links die launchd-Agenten-Flotte (Fleet-Mode, Master-Detail, Health-Klassifikation,
↵ öffnet Claude Code auf dem gewählten Agenten), rechts/Badge der Claude-Spend.
`~/.local/bin/agenttop` ist ein Symlink auf DIESEN Working Tree — der ausgecheckte
Branch ist das Live-Tool.

## Design System
Always read DESIGN.md before making any visual or UI decisions.
All color tokens (curses pairs + 256c indexes), glyphs, weight hierarchy, spacing,
and aesthetic direction are defined there. Do not deviate without explicit user
approval. In QA mode, flag any code that doesn't match DESIGN.md.

Kernregeln in Kürze (B2 „Papier"): Zustand = Glyph+Farbe+Text (nie Farbe allein) ·
single-width-Glyphen, nie Emoji (double-width schert Spalten) · GESUND IST STUMM
(dim, kein Grün) · nur zwei Chroma: fail-Rot + warn-Bernstein · Fokus = bold,
nicht Farbe · bg = Terminal-Default.

## Arbeiten am Code
- Pure-Logik headless testen: `python3 -c "from importlib.machinery import
  SourceFileLoader; m = SourceFileLoader('agenttop','agenttop').load_module(); …"`
- TUI-Smoke ohne Sichtkontakt: `(sleep 4; printf 'q') | script -q /dev/null ./agenttop`
  — Traceback im Output = Fail.
- `--once` / `--once-claude` sind die nicht-interaktiven Modi.
- ClaudeIndex v2 speichert KEINE Roh-Events (Minuten-Buckets 48 h + Tages/Stunden-
  Aggregate 31 d) — niemals eine Roh-Event-Liste wieder einführen; sie war der
  1,4-GB/4-s-Performance-Bug.
