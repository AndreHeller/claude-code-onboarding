---
name: next-steps
description: "ONBOARDING finální krok — navede nového kolegu na klonování konkrétního projektu (dbt, Streamlit, atd.), přepnutí VS Code workspace, a předání na project-specific CLAUDE.md + lokální skills. Auto-invoke JEN v onboarding kontextu: 'dokončil jsem onboarding, co dál', 'welcome mě poslal na next-steps', 'klonuj mi první projekt po onboardingu'. NEVOLAT při obecném 'co dál' — to může být cokoliv."
---

# Next steps — klonuj projekt a začni pracovat

Gratuluji — máš dokončený **obecný dev setup** pro dev:

- ✅ WSL2 + Ubuntu + Unix user
- ✅ VS Code + Remote-WSL + Claude Code extension v WSL
- ✅ Claude Code login (Claude.ai subscription (Team/Enterprise))
- ✅ SSH klíče (GitLab + GitHub)
- ✅ Git konfigurace (pull.rebase, pull.ff, user.email)
- ✅ GitHub CLI + GitLab CLI
- ✅ Claude koncepty (memory vs CLAUDE.md vs skills)
- ✅ Firemní marketplace + org-wide plugin (pokud máš)

Teď si **klonuj projekt** na kterém chceš pracovat. Každý projekt má vlastní `CLAUDE.md` a často lokální skills v `.claude/skills/`, které tě povedou v jeho specifikách.

## Nejčastější příklady projektů

### dbt (hlavní BI projekt)

**Repo**: `git@gitlab.com:your-org/your-project.git`

**Co to je**: dbt + Snowflake + Streamlit PoC pro migrace Keboola transformací. 16 staging modelů, 6 core (dim/fact), 1 mart.

**Setup**:
```bash
cd ~/dev
git clone git@gitlab.com:your-org/your-project.git
cd dbt
code .    # otevře VS Code v novém okně s tímto workspace
```

**Co bude dál**: v novém VS Code okně se načte projektový `CLAUDE.md` + lokální `.claude/skills/onboarding-dbt` skill. Napiš Claudovi:

> *"Jsem nový kolega v dbt projektu, proveď mě onboardingem."*

Claude auto-invoke `/onboarding-dbt` skill, který tě dovede:
- Python venv + `pip install dbt-core dbt-snowflake`.
- Snowflake keypair generace + upload public klíče Andrému (nebo sám přes Snowflake UI).
- `~/.dbt/profiles.yml` s per-user schema (`DEV_<JMENO>`).
- `dbt debug` ověření connection.
- První `dbt run -s stg_products` smoke test.

Čas: ~20-30 min (většinou čekání na pip install a Snowflake Workspaces access od admina).

### (Budoucí) další projekty

- `streamlit/business-drivers` — Streamlit dashboard (dbt mart reporting).


Pro každý platí **stejný pattern**:
1. Klonuj do `~/dev/`.
2. Otevři ve VS Code (`code .`).
3. Přečti `CLAUDE.md` (Claude to udělá automaticky).
4. Hledej `.claude/skills/onboarding-*` skill — projekt-specific průvodce.
5. Pokud skill existuje, pusť ho. Pokud ne, napiš Andrému že by se hodil.

## Jak pracovat v novém projektu — flow template

Po klonování projektu pattern vypadá takto:

### 1. Přečti si CLAUDE.md

```bash
cat CLAUDE.md
```

Nebo otevři v VS Code a přečti. **Pozorně**. Tam je business kontext, architektura, konvence, "co nesahat". Investuj 15 minut do čtení — ušetří hodiny debugging později.

### 2. Lokální skills (pokud jsou)

```bash
ls .claude/skills/
```

Typicky:
- `onboarding-*` — setup tohoto projektu.
- `audit-*` — kontrola stavu prostředí pro tento projekt.
- Další project-specific expertise.

### 3. Jdi podle doporučeného flow

CLAUDE.md by měl obsahovat sekci "Jak začít" nebo "Vývojový cyklus". Jdi dle ní.

### 4. Pozvi org-wide skills když potřeba

`bi` plugin je **vždy dostupný** (nainstalovaný globally). Pokud máš dotaz k git workflow:

```
org-plugin:git-workflow
```

Nebo naming:

```
org-plugin:naming-conventions
```

Org-wide věci jsou tam, projekt-specific v `.claude/skills/`.

## Trust layers (jak Claude kombinuje info)

Když v projektu pracuješ, Claude v každé zprávě **kombinuje**:

1. **Tvoje aktuální zpráva** (co chceš).
2. **Conversation history** (co už jsme v této session probrali).
3. **Memory files** (`~/.claude/projects/.../memory/` — tvoje projekt-specific knowledge).
4. **CLAUDE.md** projektu (načten automaticky, sdílený s týmem).
5. **Lokální skills** v projektu (loaded on-demand).
6. **Global pluginy** (bi, atd.) — skills loaded per popis match.

Claude pak plánuje akci a (s tvým permission) spouští tools.

## Co když se ztratíš

Kdykoli si nejsi jistý:
- `/troubleshoot` — tento plugin (dev-onboarding) má troubleshoot skill pro infra problémy.
- `org-plugin:git-workflow audit` — kontrola git nastavení.
- Projekt-specific audit skill (pokud existuje, např. `/audit-dbt-env`).
- Napiš Claudovi co vidíš + co očekáváš — většinu problémů diagnostikuje sám.

## Finální rada

**Nespěchej.** Dev setup co jsi právě prošel je jednorázová investice — jednou provedeš správně, slouží roky. Projekty se střídají, ale infrastructure zůstává.

Pokud se v některé části tohoto onboardingu zasekl nebo je něco matoucí, **napiš autorovi onboardingu (viz README). Feedback je vítaný — příští kolega bude mít lepší experience.

Hodně štěstí. 🚀
