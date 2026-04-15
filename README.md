# claude-welcome

**Public** Claude onboarding katalog pro **Slevomat AI**. Dva pluginy + standalone prompt pro Claude Desktop. Žádný auth, žádný GITLAB_TOKEN — vše veřejně přístupné.

## Architektura

```
claude-welcome/
├── PROMPT.md                     ← copy-paste prompt pro Claude Desktop (start here)
├── .claude-plugin/
│   └── marketplace.json          ← katalog dvou pluginů
└── plugins/
    ├── wsl-onboarding/           ← Windows → WSL + VS Code + Claude Code ext
    │   └── skills/install-wsl/
    └── dev-onboarding/           ← Po WSL: SSH + git + Claude konceptů + Slevomat MP
        └── skills/welcome/, setup-ssh/, setup-git/, ... (TODO)
```

## Komu je to určeno

- **Dev / BI inženýr** — chystáš se na dbt / Snowflake / Streamlit projekty ve Slevomat. ✅ Jsi v cílovce.
- **Analytik / data konzument** — máš jen číst hotové marty, nechceš programovat? Tohle pro tebe není (konzumentský onboarding je TODO).
- **Product / PM / marketing** — neprogramuješ? Standardní Claude Cowork bez specifických Slevomat pluginů.

## Jak začít

### Pokud máš jen Claude Desktop (Windows app z claude.ai/download)

1. Otevři Claude Desktop chat.
2. Copy-paste obsah z **[PROMPT.md](PROMPT.md)** do chatu.
3. Claude si stáhne `wsl-onboarding/install-wsl` skill z tohoto repa a provede tě WSL setupem.
4. Po dokončení tě naveže na `dev-onboarding` plugin v Claude Code extension.

### Pokud už máš WSL + Claude Code extension v VS Code (přeskoč WSL setup)

V Claude Code chat panelu napiš:

```
Nainstaluj plugin dev-onboarding@claude-welcome z https://github.com/AndreHeller/claude-welcome
a spusť skill welcome — provede mě SSH, git, Slevomat marketplace setupem.
```

(Skills v `dev-onboarding` jsou aktuálně rozpracované — TODO.)

## Plánovaná struktura

### Plugin `wsl-onboarding` (existuje, alpha)

Jediný skill `install-wsl` obsahující kompletní hloubkový tutoriál:
- WSL2 + Ubuntu (latest LTS) install
- Unix user setup (přes Win+S → Ubuntu, ne `wsl` z PowerShellu — kvůli mountu)
- VS Code Remote-WSL extension
- Claude Code extension v WSL kontextu (NE Local)
- Login Slevomat účtem (Claude.ai subscription, copy-paste token, async race condition)
- Smoke test (hello.py)
- Handoff na `dev-onboarding`

Spouští se z **Claude Desktop** přes fetch markdown URL (viz [PROMPT.md](PROMPT.md)). NE přes `/plugin install` — Claude Desktop nemá per-user `/plugin` slash command, plugin install je jen v admin Cowork katalogu (rollout TBD).

### Plugin `dev-onboarding` (TODO)

Skills:
- `welcome` — entry point, detekce stavu, rozcestník.
- `setup-ssh` — ed25519 klíč, upload do GitLab + GitHub.
- `setup-git` — user config, pull.rebase, pull.ff, init.defaultBranch.
- `install-gh-glab` — GitHub CLI + GitLab CLI.
- `claude-concepts` — vysvětlení memory vs CLAUDE.md vs skills, kde to leží, co modifikovat a co ne.
- `install-slevomat-marketplace` — registrace privátního `slevomat/ai/claude-marketplace` do settings.json + install `bi` plugin (org-wide git konvence + naming).
- `next-steps` — "klonuj projekt na kterém pracuješ. Každý Slevomat projekt má vlastní CLAUDE.md a lokální skills, které tě povedou v jeho specifikách."
- `troubleshoot` — typické gotchy.

Spouští se v **Claude Code extension v WSL** přes `/plugin install dev-onboarding@claude-welcome`.

### Co tento repo NEobsahuje

- **Project-specific onboarding** (dbt, Snowflake setup, Streamlit deploy, atd.) — to je v `.claude/skills/` v každém Slevomat projektu samostatně.
- **Slevomat-wide konvence** (git workflow, naming) — to je v privátním `slevomat/ai/claude-plugin-bi` (instaluje se přes `dev-onboarding/install-slevomat-marketplace`).

## Pro Claude.ai admin: budoucí deploy do Cowork org

Až bude `dev-onboarding` plugin kompletní a otestovaný, plánovaný rollout do Slevomat Cowork org marketplace:

1. Claude.ai → **Admin Settings → Claude Code → Managed settings → Plugins**.
2. **GitHub sync** → `AndreHeller/claude-welcome`.
3. Visibility: `Installed by default` pro dev kolegy (nebo `Available for install`).
4. Kolegové v Slevomat Claude org uvidí pluginy v Claude Desktop **Browse plugins**.

To vyžaduje Anthropic Enterprise/Team plán + admin práva. Pro per-user (mimo Cowork) zůstává **fetch URL** flow (PROMPT.md).

## Development

Pro lokální test pluginu (před push):

```bash
git clone git@github.com:AndreHeller/claude-welcome.git
cd claude-welcome
claude plugin validate .
# nebo v Claude Code:
# /plugin marketplace add ./claude-welcome
# /plugin install wsl-onboarding@claude-welcome
```

## Kontakt

- Owner: André Heller — `andre.heller@slevomat.cz`
- GitHub: [AndreHeller/claude-welcome](https://github.com/AndreHeller/claude-welcome)

## Licence

Internal use (Slevomat). Public visibility na GitHubu kvůli fetch URL přístupu — neobsahuje žádné credentials, jen tutoriály.
