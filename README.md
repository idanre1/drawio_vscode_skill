# drawio-vscode-skill

`drawio-vscode-skill` is a VS Code-focused wrapper around the upstream [`drawio-skill`](https://github.com/Agents365-ai/drawio-skill).

It keeps the upstream Draw.io skill available as a git submodule for XML structure, shape references, style presets, helper scripts, and validation guidance, while changing the working assumption: diagrams are edited as `.drawio` XML and rendered by an active VS Code Draw.io extension instead of the draw.io desktop CLI.

## Skill Slugs

- This skill: `drawio-vscode-skill`
- Bundled upstream skill: `drawio-skill`

The names are intentionally almost the same. Use `drawio-vscode-skill` when VS Code is active and the user wants the VS Code Draw.io extension to render the diagram. Use `drawio-skill` when the desktop draw.io CLI workflow and image exports are desired.

## Requirements

- VS Code with an active workspace
- A Draw.io-compatible VS Code extension, such as [Draw.io Integration (`hediet.vscode-drawio`)](https://marketplace.visualstudio.com/items?itemName=hediet.vscode-drawio)
- Git submodule support

The draw.io desktop application and CLI are not required for this wrapper skill.

## Fresh Install

The GitHub repository is named `drawio_vscode_skill`, but the skill slug is `drawio-vscode-skill`. When installing directly into a skills directory, clone into a target folder named `drawio-vscode-skill` so the folder name matches `SKILL.md`.

Clone the repository into your skills path with submodules enabled:

```bash
git clone --recurse-submodules git@github.com:idanre1/drawio_vscode_skill.git ~/.claude/skills/drawio-vscode-skill
```

Do not install it as `~/.claude/skills/drawio_vscode_skill`; skill validators expect the folder name and slug to match.

If you prefer HTTPS:

```bash
git clone --recurse-submodules https://github.com/idanre1/drawio_vscode_skill.git ~/.claude/skills/drawio-vscode-skill
```

For Copilot skill locations, use the same pattern with your preferred skills directory, for example:

```bash
git clone --recurse-submodules git@github.com:idanre1/drawio_vscode_skill.git ~/.copilot/skills/drawio-vscode-skill
```

If you already cloned without submodules, initialize them afterward:

```bash
cd ~/.claude/skills/drawio-vscode-skill
git submodule update --init --recursive
```

After installation, both files should exist:

```text
~/.claude/skills/drawio-vscode-skill/SKILL.md
~/.claude/skills/drawio-vscode-skill/drawio-skill/skills/drawio-skill/SKILL.md
```

If `drawio-skill/` exists but is empty, the submodule was not initialized. Run:

```bash
cd ~/.claude/skills/drawio-vscode-skill
git submodule update --init --recursive
```

## Updating

Update this wrapper and the bundled upstream skill together:

```bash
cd ~/.claude/skills/drawio-vscode-skill
git pull --recurse-submodules
git submodule update --init --recursive
```

To move the upstream Draw.io skill submodule to its latest tracked remote commit:

```bash
cd ~/.claude/skills/drawio-vscode-skill
git submodule update --remote --merge drawio-skill
```

## Workflow

1. Load `drawio-vscode-skill` for `.drawio` work in VS Code.
2. Use the bundled `drawio-skill` submodule as the reference for Draw.io XML, shapes, styles, and validators.
3. Edit `.drawio` XML files directly.
4. Validate XML structurally.
5. Open or preview the `.drawio` file with the VS Code Draw.io extension.

This keeps diagrams editable in source control and avoids any dependency on the desktop draw.io CLI.