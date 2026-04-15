---
name: install-slevomat-marketplace
description: "Registrace privátního Slevomat GitLab marketplace (slevomat/ai/claude-marketplace) do settings.json a instalace org-wide bi pluginu (git workflow konvence, naming conventions, audit checks, update-check hook). Předpokládá dokončený setup-ssh (SSH key je registrovaný v GitLabu — marketplace klonuje přes SSH). Auto-invoke při 'nainstaluj slevomat plugin', 'bi plugin', 'marketplace'."
---

# Privátní Slevomat marketplace + bi plugin

Máš SSH, git, gh/glab. Teď ti dáme přístup k **Slevomat-wide Claude pluginům**. Tohle je privátní marketplace — jen členové Slevomat organizace (s SSH key registrovaným v GitLabu) mají přístup.

## Co je v Slevomat marketplace

Aktuálně jeden plugin: **`bi`** — org-wide pro BI engineers:
- **`git-workflow`** skill — Slevomat git konvence (feature branch, rebase, MR, commit messages v čs, protected main) + integrovaný audit git configu.
- **`naming-conventions`** skill — Slevomat BI naming patterns pro sloupce a tabulky (prefix, camelCase, FK na konci, stg_/dim_/fact_/mart_).
- **`update-check`** skill + SessionStart hook — 1× za 24h kontrola nové verze pluginu (fail-silent).

Budoucí pluginy (TODO):
- `data-consumer` — pro analytiky bez dev skills.
- `devops` — CI/CD, deploy tooling.
- `shared-utils` — společné utility napříč teams.

## Prerequisites

- **Dokončený `/setup-ssh`** — marketplace repo je na privátním GitLabu, klonuje se přes SSH.
- **Ověřené SSH spojení na GitLab**:
  ```bash
  ssh -T git@gitlab.com
  # Welcome to GitLab, @<tvoje_username>!
  ```

Pokud SSH test selže, **vrať se na `/setup-ssh`** — nejdřív fix, pak pokračuj sem.

## Krok 1: Registrace marketplace

Přidej marketplace do svých Claude settings:

```bash
# V Claude Code (slash command v chat panelu):
/plugin marketplace add git@gitlab.com:slevomat/ai/claude-marketplace.git
```

Co se stane:
- Claude Code naklonuje marketplace repo přes SSH do `~/.claude/plugins/marketplaces/slevomat-ai/`.
- Přečte `.claude-plugin/marketplace.json` — uvidí že tam je plugin `bi`.
- Přidá entry do `~/.claude/settings.json` pod `extraKnownMarketplaces.slevomat-ai`.

Ověř:
```bash
claude plugin marketplace list
```

Měl bys vidět `slevomat-ai` v seznamu.

**Pokud registrace selže s permission denied**: SSH key není v GitLabu, nebo ssh-agent ho nemá. Zkus znovu `ssh -T git@gitlab.com`, vyřeš auth, pak opakuj.

## Krok 2: Install bi plugin

```bash
# V Claude Code:
/plugin install bi@slevomat-ai
```

Claude Code:
1. Přečte `marketplace.json` v slevomat-ai marketplace.
2. Najde plugin `bi`, vidí `source: git@gitlab.com:slevomat/ai/claude-plugin-bi.git, ref: release/v1`.
3. Naklonuje plugin repo (SSH) do `~/.claude/plugins/cache/slevomat-ai/bi/<version>/`.
4. Načte skills + hooks.
5. Plugin aktivovaný.

Ověř:
```bash
claude plugin list
```

Měl bys vidět `bi@slevomat-ai` v installed.

## Krok 3: Vyzkoušej si skills

Plugin má 3 skills dostupné pod namespace `slevomat-ai:bi:`:

### git-workflow

```
/slevomat-ai:bi:git-workflow
```

Claude ti vysvětlí Slevomat git konvence (feature branch naming, rebase, MR, commit messages v čs s body, protected branches). Referenční mód.

V **audit módu** zkontroluje tvůj aktuální git config:

```
/slevomat-ai:bi:git-workflow audit
```

Claude spustí checky `pull.rebase`, `pull.ff`, `user.email`, remote origin — vydá verdikt ✅/⚠/❌ pro každý a navede na fixy.

### naming-conventions

```
/slevomat-ai:bi:naming-conventions
```

Claude ti ukáže Slevomat BI naming patterns pro sloupce a tabulky. Hodí se když napíšeš nový dbt model — Claude podle tohoto skill auto-invoke a připomene pravidla (prefix product_/voucher_, FK na konci, atd.).

### update-check

Běží **automaticky** každé 24h při start VS Code (SessionStart hook). Pokud je nová verze plugin v marketplace, vypíše:

```
📦 slevomat-ai:bi update dostupný: 1.0.0 → 1.1.0
   Spusť /plugin update bi@slevomat-ai — nebo řekni Claudovi 'updatuj plugin'.
```

Pro manuální check:
```
/slevomat-ai:bi:update-check
```

## Krok 4: Jak update

Když chceš nejnovější verzi:

```
/plugin update bi@slevomat-ai
```

Claude Code přetahuje git pull v lokálním plugin clone, načte nové skills. Pokud verze major bumpla (v1 → v2), může vyžadovat explicit opt-in kvůli breaking changes.

## Kam bi plugin leží na disku

- **Marketplace metadata**: `~/.claude/plugins/marketplaces/slevomat-ai/`
- **Plugin soubory**: `~/.claude/plugins/cache/slevomat-ai/bi/<version>/`
  - `.claude-plugin/plugin.json` — metadata, version.
  - `skills/git-workflow/SKILL.md`
  - `skills/naming-conventions/SKILL.md`
  - `skills/update-check/SKILL.md`
  - `scripts/check-updates.sh`
  - `hooks/hooks.json` — SessionStart hook config.

**Nesahej do těchto adresářů ručně** — jsou spravované Claude Code. Pokud chceš přispět / opravit plugin, klonuj source repo (`git clone git@gitlab.com:slevomat/ai/claude-plugin-bi.git`), udělej změnu tam, MR → po merge další `/plugin update` u kolegů.

## Hotovo, co dál

Máš Slevomat org-wide pluginy. Claude už ví Slevomat konvence a začne ti na ně auto-upozornovat při kódování.

Pokračuj na **`/next-steps`** — klonování konkrétního projektu a přechod na jeho project-specific onboarding.
