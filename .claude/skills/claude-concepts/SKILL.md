---
name: claude-concepts
description: "Vysvětluje Claude architekturu pro nového uživatele — rozdíl mezi conversation kontextem, memory, CLAUDE.md, plugins/skills, hooks, settings. Kde každá vrstva leží na disku, co je persistent a co ne, co modifikovat ručně a co nechat Claudovi. Kritické pro pochopení než začneš pracovat. Auto-invoke při 'jak funguje Claude', 'co je memory', 'kde je CLAUDE.md uložená', 'rozdíl mezi skill a CLAUDE.md'."
---

# Claude Code architektura pro uživatele

Než začneš pracovat na projektech, **pochop tyto koncepty**. Bez nich nebudeš vědět proč Claude někdy "zapomene" věc, jindy si pamatuje věci měsíce zpět, kdy sahat do `~/.claude/` a kdy do git repa. 15 minut teď ti ušetří hodiny budoucího frustrated guessingu.

## Šest vrstev kontextu pro Claude (od ephemerální po persistent)

Když Claude odpovídá na tvou zprávu, kombinuje **šest zdrojů informací**:

### 1. **Conversation context** (in-memory, ephemerální)

- **Co to je**: tvoje aktuální zprávy v chatu + Claudeovy odpovědi + tool call results.
- **Kde**: jen v paměti Claude Code procesu. Zavři VS Code → pryč.
- **Perzistence**: žádná (nebo jen session-scoped resume).
- **Užitečné pro**: jednorázové úkoly ("přepiš tento soubor", "fix tenhle bug").

**Typický příznak**: začneš novou session, Claude neví o předchozím chatu. Normální.

### 2. **Memory files** (`~/.claude/projects/.../memory/`)

- **Co to je**: markdown soubory s fakty o tobě, feedback od tebe, project context, reference na externí systémy.
- **Kde**: `~/.claude/projects/<sanitized-path>/memory/` — per-project, per-user.
  - Příklad: pro repo v `/home/avatar/dev/dbt` je memory v `/home/avatar/.claude/projects/-home-avatar-dev-dbt/memory/`
- **Perzistence**: **napříč sessions** pro tebe + tento projekt. Jiný user nebo jiný projekt má svoje separate memory.
- **Kdo píše**: Claude automaticky (když řekneš "zapamatuj si že mám rád tabulky" nebo on usoudí že je to důležité). Ty **můžeš** ručně editovat.
- **Co obsahuje**:
  - `user_*.md` — kdo jsi, tvé role, znalost, preference.
  - `feedback_*.md` — jak pracovat s tebou (např. "commit jen na vyžádání").
  - `project_*.md` — proč se co dělá, incidents, deadlines.
  - `reference_*.md` — kde najde co (Linear project, Slack channel, Grafana dashboard).
  - `MEMORY.md` — index (seznam všech memory souborů).

**Důležité**: memory **NENÍ sdílená s kolegy**. Je to **tvoje** soukromá historie v tomto projektu. Git s memory nemá nic společného (není commitnutá).

**Kdy editovat ručně**: když vidíš že si Claude pamatuje něco zastaralého nebo nesprávného — najdi soubor v `memory/`, uprav / smaž. Nebo prostě řekni Claudovi "zapomeň že mám rád tabulky" — smaže si sám.

### 3. **CLAUDE.md** (per-project, commitnutý)

- **Co to je**: markdown soubor v rootu git repa (nebo v parent adresářích). Automaticky načten každou session.
- **Kde**: `<repo>/CLAUDE.md`. Příklady:
  - `/home/avatar/dev/dbt/CLAUDE.md`
  - `/home/avatar/.claude/CLAUDE.md` — globální pro všechny tvé projekty (tvoje osobní preference).
- **Perzistence**: git-commitnuto → **sdílené s týmem** (kolegové vidí stejný CLAUDE.md).
- **Co obsahuje**:
  - Popis projektu (co to je, business kontext).
  - Architektura (kde co leží, jak věci spolu souvisí).
  - Konvence (naming, styling, workflow pravidla).
  - Kritické upozornění ("nikdy nemaž tabulku X", "build příkaz je Y, ne Z").
  - Seznam dostupných skills / pluginů v projektu.

**Kdo píše**: tým (ručně, přes git). Claude může navrhnout změny, ale commit děláš ty přes MR.

**Kdy editovat**: když projekt má nové pravidlo / konvenci. Např. "od teď všechny commits česky" → přidej do CLAUDE.md → git commit → kolegové po pull mají stejné pravidlo.

### 4. **Plugins & skills** (instalované, on-demand loaded)

- **Co to je**: balíky Claude capabilities — skills (expert knowledge), commands (slash commands), hooks (automatické reakce), agents (sub-Claude specialists), MCP servers (external tool integrations).
- **Kde**:
  - **Globální pluginy**: `~/.claude/plugins/cache/<marketplace>/<plugin>/<version>/`.
  - **Projekt-lokální skills**: `<repo>/.claude/skills/<skill-name>/SKILL.md`.
- **Perzistence**: plugins jsou instalované přes `/plugin install`, verzované přes marketplace. Projekt-lokální jsou commitnuté do git.
- **Spouští**: buď **automaticky** (Claude detekuje relevant situation podle `description` ve SKILL.md frontmatter), nebo **manuálně** (slash command `/setup-ssh`).
- **Co obsahuje**: specializované znalosti — `setup-ssh` ví jak SSH klíče generovat, `naming-conventions` ví Slevomat BI naming patterns.

**Kdo píše**: autor pluginu (centrálně) + repo owner (pro lokální skills).

**Kdy editovat**: nikdy ručně v `~/.claude/plugins/cache/` — to je stažená kopie. Editovat ve zdrojovém git repu pluginu, pushnout, ostatní dostanou přes `/plugin update`.

### 5. **Hooks** (`.claude/settings.json` nebo `~/.claude/settings.json`)

- **Co to je**: automatické reakce na eventy (PreToolUse, PostToolUse, SessionStart, atd.).
- **Kde**:
  - **Per-user**: `~/.claude/settings.json`.
  - **Per-project**: `<repo>/.claude/settings.json` (committed) nebo `<repo>/.claude/settings.local.json` (per-user, ignored).
- **Perzistence**: per-user nebo per-project dle scope.
- **Příklady**: auto-lint po Write souboru, notifikace v Slack při dokončení dlouhého úkolu, auto-check updates plugin.

**Kdo píše**: ručně (ty) nebo přes `update-config` skill.

**Kdy editovat**: když chceš deterministickou auto-akci. Pro on-demand logiku (conditional, model-driven) použij skill místo hook.

### 6. **Permissions** (`.claude/settings.json` → `permissions` blok)

- **Co to je**: allow/deny rules co Claude smí dělat (Bash příkazy, Read / Write / Edit files, atd.).
- **Kde**: `permissions` klíč v settings.json (per-user nebo per-project).
- **Perzistence**: dle scope.
- **Typy**: `allow` (auto-Allow bez dialog), `deny` (auto-reject), `ask` (dialog).

**Kdo píše**: ty, ale Claude se ptá "Always allow Write for this project?" — klikni a uloží do settings.json.

## Kde to všechno leží na disku (souhrn)

```
~/.claude/                               ← Per-user globální
├── CLAUDE.md                            ← Tvoje globální preference (napříč projekty)
├── settings.json                        ← Permissions, marketplaces, hooks (per-user)
├── settings.local.json                  ← Per-user override (nesdílet)
├── keybindings.json
├── projects/
│   └── <sanitized-path>/
│       └── memory/                      ← Per-project memory (tvoje, soukromé)
│           ├── MEMORY.md                ← Index
│           ├── user_*.md
│           ├── feedback_*.md
│           ├── project_*.md
│           └── reference_*.md
├── plugins/
│   ├── known_marketplaces.json
│   └── cache/<marketplace>/<plugin>/<version>/   ← Stažené pluginy
└── skills/                              ← Tvoje osobní globální skills

<repo>/                                  ← Per-project, git-commitnuté
├── CLAUDE.md                            ← Sdílené s týmem
├── .claude/
│   ├── settings.json                    ← Team settings (permissions, hooks)
│   ├── skills/<name>/SKILL.md           ← Project-local skills (např. onboarding-dbt)
│   ├── commands/<cmd>.md                ← Project-local slash commands
│   └── agents/<agent>.md                ← Project-local sub-agents
```

## Kdy co modifikovat (cheat sheet)

| Chceš | Edituj |
|---|---|
| Naučit Claude něco o tobě | řekni mu zprávou "zapamatuj si X" → automaticky do `~/.claude/projects/.../memory/` |
| Změnit konvenci projektu | `<repo>/CLAUDE.md` → git commit |
| Přidat nový slash command pro projekt | `<repo>/.claude/commands/<cmd>.md` → git commit |
| Aktualizovat osobní bash snippety | `~/.claude/CLAUDE.md` (tvoje osobní) — ne do projektu |
| Dát Claudovi trvalý auto-allow pro `git *` | `<repo>/.claude/settings.json` → `permissions.allow` |
| Vypnout auto-update check | Settings → Auto-update toggle (plugin-specific) |

## Nikdy nesahej (Claude tam sáhne za tebe)

- `~/.claude/plugins/cache/` — stažené kopie pluginů, přepíše se při `/plugin update`.
- `~/.claude/plugins/known_marketplaces.json` — Claude spravuje.
- `<repo>/.claude/env-check.json` (pokud v projektu je) — per-user audit cache.
- Claude conversation logs — persistent jen pro tvůj stroj.

## Hotovo, co dál

Rozumíš Claude architektuře. Teď pokračuj na **`/install-slevomat-marketplace`** — registrujeme privátní Slevomat GitLab marketplace a nainstalujeme `bi` plugin (org-wide git konvence, naming).
