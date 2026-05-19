---
name: next-steps
description: "ONBOARDING finální krok — navede nového kolegu na klonování prvního projektu, přepnutí VS Code workspace, a předání na project-specific CLAUDE.md + lokální skills. Adaptivně detekuje, zda má kolega nainstalovaný firemní plugin, a pokud ano, nasměruje na jeho project-specific onboarding skills. Auto-invoke JEN v onboarding kontextu: 'dokončil jsem onboarding, co dál', 'welcome mě poslal na next-steps', 'klonuj mi první projekt po onboardingu'. NEVOLAT při obecném 'co dál' — to může být cokoliv."
---

# Next steps — klonuj projekt a začni pracovat

Gratuluji — máš dokončený **obecný dev setup**:

- ✅ WSL2 + Ubuntu + Unix user (pokud jsi Windows)
- ✅ VS Code + Remote-WSL + Claude Code extension
- ✅ Claude Code login
- ✅ SSH klíče (GitHub / GitLab)
- ✅ Git konfigurace (pull.rebase, pull.ff, user.email)
- ✅ GitHub CLI + GitLab CLI
- ✅ Claude koncepty (memory vs CLAUDE.md vs skills)
- ✅ Firemní marketplace + plugin (pokud máš)

Teď si **klonuj první projekt** na kterém chceš pracovat.

## Univerzální pattern pro každý projekt

Platí pro jakýkoliv dev projekt (web, data pipeline, ML, cokoliv):

### 1. Klonuj repo do ~/dev/

**Potřebuješ SSH clone URL tvého konkrétního projektu.** Níže jsou komentované příklady — **nekopíruj je doslova**, nahraď svým URL.

```bash
cd ~/dev

# Příklady SSH URL (formát — nahraď org/group/repo tím správným):
# git clone git@github.com:<org>/<repo>.git                     # GitHub obecně
# git clone git@gitlab.com:<group>/<subgroup>/<projekt>.git     # GitLab obecně
# git clone git@gitlab.com:slevomat/bi/dbt.git                  # příklad: Slevomat dbt repo

cd <nazev-slozky>   # složka co git clone vytvořil (obvykle = název repa)
```

**Kde najít správné URL pro tvůj projekt:**

- **GitHub**: otevři repo v browseru → zelené tlačítko **`Code`** → záložka **`SSH`** → zkopíruj (formát `git@github.com:org/repo.git`).
- **GitLab**: otevři repo → modré tlačítko **`Clone`** → sekce **`Clone with SSH`** → zkopíruj (formát `git@gitlab.com:group/project.git`).
- **Nevíš, který projekt je ten správný**: **zeptej se team-leada nebo buddyho**. Každý tým má 1-3 primární repa pro onboarding nového člena (typicky dbt projekt, BI aplikace, nebo core service) — task ticket ti řekne, na kterém budeš pracovat.

**Proč SSH a ne HTTPS**: máš `setup-ssh` za sebou, klíče fungují. HTTPS by tě nutilo zadávat token při každém `git push` nebo konfigurovat credential helper. SSH je pro dev práci ergonomicky lepší.

**Pokud clone selže s `Permission denied (publickey)`**: SSH key není registrovaný na té platformě, nebo není v ssh-agent. Spusť skill **`troubleshoot`**.

### 2. Otevři ve VS Code

```bash
code .
```

Otevře se **nové VS Code okno** přesně s tímto workspace. Claude Code extension načte projektová pravidla z `CLAUDE.md` automaticky.

### 3. Přečti CLAUDE.md

```bash
cat CLAUDE.md
```

Nebo otevři v editoru. **Pozorně**. Tam bývá business kontext, architektura, týmové konvence, "co nesahat". 15 minut čtení ušetří hodiny debugování později.

### 4. Hledej project-specific onboarding skill

```bash
ls .claude/skills/
```

Typická jména:
- `onboarding-*` — project setup průvodce (virtualenv, env credentials, DB access, smoke test)
- `audit-*` — kontrola stavu prostředí pro tento projekt
- Další project-specific expertise

Pokud existuje onboarding skill, napiš Claudovi:

> *"Jsem nový v tomto projektu, proveď mě setupem."*

Claude skill auto-invoke a provede tě. Pokud skill neexistuje, postupuj podle `CLAUDE.md` ručně.

### 5. Pokud něco chybí, napiš autorovi

Pokud projekt nemá `CLAUDE.md` nebo onboarding skill, **napiš autorovi repa** (viz `git log | head -5` pro poslední autory, nebo README). Feedback je vítaný — další kolega bude mít lepší experience.

## Máš firemní plugin? Zjistíme to

```bash
claude plugin list 2>&1 | grep -v "^$" | head -20
```

- **Pokud vidíš firemní plugin** (např. `bi@slevomat-ai`, `<firma>-skills@<firma>-marketplace`): firemní plugin má typicky **project-specific onboarding skills** nebo org-wide doporučení co klonovat. Projdi jeho popis:
  ```bash
  claude plugin list --verbose 2>&1 | head -40
  ```
  A napiš Claudovi: *"Mám firemní plugin `<název>`, co s ním dál?"* — adaptivně ti poradí.

- **Pokud nemáš žádný firemní plugin**: OK, pokračuj univerzálním patternem výše. Pokud tě zajímá registrace firemního marketplace, spusť **`install-marketplace`** skill.

## Trust layers — jak Claude kombinuje info

Když v projektu pracuješ, Claude v každé zprávě kombinuje:

1. **Tvoje aktuální zpráva** (co chceš).
2. **Conversation history** (co už jsme probrali).
3. **Memory files** (`~/.claude/projects/.../memory/` — tvoje personal persistent poznatky).
4. **Projektový `CLAUDE.md`** (načten automaticky, sdílený s týmem).
5. **Lokální skills** v projektu (`.claude/skills/`, on-demand).
6. **Nainstalované pluginy** (firemní + osobní, skills load per description match).

Claude pak plánuje akci a (s tvým permission) spouští tools.

## Co když se ztratíš

- **`troubleshoot`** skill v tomto pluginu pokrývá typické infra problémy (SSH, git, WSL).
- **Project-specific audit skill** (pokud projekt má — např. `audit-<projekt>-env`) — kontrola specifického projektu.
- **Firemní plugin** může mít obecné git/review/deployment skills.
- **Prostě napiš Claudovi** co vidíš + co očekáváš. Většinu problémů diagnostikuje sám.

## Finální rada

**Nespěchej.** Dev setup co jsi právě prošel je jednorázová investice — jednou provedeš správně, slouží roky. Projekty se střídají, infrastruktura zůstává.

Pokud se v některé části onboardingu zasekl nebo je něco matoucí, **napiš autorovi onboardingu** (viz `README.md` tohoto pluginu). Feedback dělá další iteraci lepší.

Hodně štěstí. 🚀
