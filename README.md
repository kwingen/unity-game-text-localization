# Unity Game Text Localization (Codex Skill)

A reusable Codex/Claude skill for extracting Japanese text from Unity games (including adult titles) and producing a playable Chinese localization. Distilled from a real end-to-end project.

## What it covers

- Unity asset structure analysis & text discovery (UnityPy + TypeTreeGenerator)
- Corpus extraction with dedup, SHA-256 IDs, and full source-location tracking
- Local Sakura (llama-server) translation with strict validation:
  - placeholder protection for rich-text/params/separators
  - line/order/multiplicity checks, prompt-leakage rejection, segment fallback
  - resumable append-only checkpoints bound to a glossary hash
- Independent asset overlay (source game dir stays read-only, byte-verified)
- BepInEx runtime loader: exact-match UGUI dictionary, UGUI/TMP font replacement, Utage font-only
- Isolated-copy smoke verification with in-game evidence

## Pitfalls captured

- Online strong models plan/analyze well but **adult content triggers their content filters** — do the translation with local Sakura instead.
- Glossary substring pollution (`ロード→读取` breaks `ダウンロード`) — longest-match-first, add long forms explicitly.
- Model "rambling" (fills the full token budget without EOS) burns minutes per retry — **curate the last ~10–20 stubborn entries by hand/agent instead of retrying**.
- Font style matching: identify the original font (e.g. rounded gothic) and pick a matching Chinese font (e.g. YouYuan); pre-check glyph coverage with GDI; **TMP runtime font creation only works via the 3-arg `CreateFontAsset(familyName, styleName, pointSize)` overload** (8-arg/DynamicOS return null in Player builds).

## Install

Put the folder in your skill directory:

```bash
# Linux/macOS
mkdir -p ~/.agents/skills
cp -r unity-game-text-localization ~/.agents/skills/

# or with $CODEX_HOME set
cp -r unity-game-text-localization "$CODEX_HOME/skills/"
```

Then ask your coding agent to use it, e.g.:

> 用 unity-game-text-localization 技能，从 <game-path> 提取日文文本并汉化

## Layout

```text
unity-game-text-localization/
|-- SKILL.md
`-- references/
    |-- workflow.md
    `-- pitfalls.md
```

## Notes

- The original game directory is always read-only; everything is produced in independent directories.
- Adult-content automation should only be run on targets the user explicitly authorizes.
