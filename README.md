# Unity Game Text Localization Skill

Agent-neutral skill for extracting, translating, and back-filling text in Unity games. It works across Mono and IL2CPP builds and does not assume a fixed language pair, genre, localization system, or runtime hook.

## What it covers

- Build inventory and text-source discovery
- Localization tables, external data, serialized assets, AssetBundles, Addressables
- UI text, code strings, dynamic text, and text baked into media
- Corpus extraction with context, placeholders, and source locations
- Translation provider selection, glossaries, QA, and structural validation
- Native table patching, asset overlay, runtime hooks, font/UI adaptation
- Isolated-copy verification and reproducible manifests

## Agent compatibility

The skill uses the common `SKILL.md` format:

- Codex: place under `$CODEX_HOME/skills` or `~/.codex/skills`
- Claude Code: place under `~/.claude/skills` or a project skills directory
- Hermes / Ekko: import through the profile skill manager, or copy into the configured skills directory

Host-specific metadata is optional; unsupported fields can be ignored.

## Usage

Ask your agent:

```text
使用 unity-game-text-localization，从 <game-path> 提取文本并生成 <target-language> 本地化补丁
```

## Layout

```text
unity-game-text-localization/
|-- SKILL.md
`-- references/
    |-- workflow.md
    |-- text-sources.md
    |-- translation.md
    |-- backfill.md
    `-- pitfalls.md
```

## Safety

Only work on authorized copies. Do not bypass DRM, anti-cheat, encryption, or obfuscation. Keep the original game directory read-only and produce changes in an overlay or isolated copy.
