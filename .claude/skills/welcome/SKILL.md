---
name: welcome
description: "Entry point onboardingu po WSL setupu. Detekuje aktuální stav prostředí (SSH, git, gh, glab, firemní marketplace) a navádí na zbývající skills v doporučeném pořadí. Auto-invoke když uživatel řekne 'jsem nový po WSL', 'pokračuj onboardingem', 'co dál', nebo zadá '/welcome'. První krok co Claude v dev-onboarding pluginu udělá."
---

# Vítej v dev onboardingu — fáze 2 🚀

Předpokládám že už máš za sebou **WSL setup** (přes `wsl-onboarding/install-wsl` skill v Claude Desktop). Pokud ne, nejdřív se vrať k tomu — bez WSL nepokračujeme.

Tento plugin tě dovede k bodu kdy:
1. Máš funkční SSH klíče (přístup do GitLab + GitHub).
2. Máš správnou git konfiguraci dle týmových konvencí.
3. Máš nainstalované GitHub CLI (`gh`) + GitLab CLI (`glab`).
4. Rozumíš základním Claude konceptům — kde co leží (memory, CLAUDE.md, skills), na co sahat a na co ne.
5. Máš zaregistrovaný privátní firemní marketplace + nainstalovaný firemní plugin (pokud máš marketplace).
6. Víš jak klonovat konkrétní projekt a kde najdeš jeho specifické instrukce.

## Krok 1: Detekce aktuálního stavu

Spustím tyhle příkazy a zjistím co už máš (s tvým permissions). Některé mohou selhat — nevadí, jen mi tím řekneš co chybí:

```bash
# Kdo jsem
whoami
pwd

# SSH klíče
ls -la ~/.ssh/

# Git config
git config --global --list | grep -E "user|pull|init" || echo "(žádný git config)"

# GitHub CLI
command -v gh && gh --version | head -1 || echo "(gh nenainstalovaný)"

# GitLab CLI
command -v glab && glab --version | head -1 || echo "(glab nenainstalovaný)"

# Claude plugin marketplace
claude plugin marketplace list 2>/dev/null || echo "(žádné marketplaces zaregistrované)"
claude plugin list 2>/dev/null || echo "(žádné plugins nainstalované)"
```

Podle výstupu poznám co už máš a co chybí, pak ti navrhnu **doporučený sled skills**.

## Doporučený sled (pro úplného nováčka)

V tomto pořadí, krok po kroku — neskáčeme dopředu:

1. **`/setup-ssh`** — SSH klíče (asi 10 minut včetně upload do GitLab + GitHub). Vysvětlím **co SSH klíč je**, jak se liší od hesla / access tokenu / OAuth, kde soubory leží.

2. **`/setup-git`** — git konfigurace dle týmových konvencí: user.name, user.email, pull.rebase, pull.ff. Vysvětlím **proč** každé pravidlo (zvlášť rebase vs merge — kazí lineární historii).

3. **`/install-gh-glab`** — GitHub CLI (`gh`) + GitLab CLI (`glab`). Pokud tvůj tým používá obě platformy.

4. **`/claude-concepts`** — **DŮLEŽITÉ pro práci s Claude**: rozdíl mezi memory, CLAUDE.md, skills. Kde to leží na disku, co modifikovat ručně, co nechat Claudovi. Bez tohoto budeš v Claude flow tápat.

5. **`/install-marketplace — volitelné: registrace firemního privátního marketplace.

6. **`/next-steps`** — "teď si klonuj projekt na kterém budeš pracovat (dbt, Streamlit, atd.). Každý projekt má vlastní `CLAUDE.md` a často lokální skills v `.claude/skills/`, které tě povedou v jeho specifikách."

## Pokud jsi advanced uživatel

Pokud už máš SSH + git nastavené (jiný projekt, předchozí setup), můžeš:
- **Přeskočit** některé skills (ukaž mi výstup detekce, doporučím co skipnout).
- **Začít rovnou** na `/claude-concepts` (je důležité pro každého kdo Claude používá poprvé).
- **Skočit** na `/install-marketplace` pokud chceš jen firemní marketplace setup.

## Troubleshooting

Pokud kdykoli něco selže (`/setup-ssh` nepojde, `gh auth login` zaseká), použij **`/troubleshoot`** — projdu typické gotchy a navedu na fix.

## Začni — pošli mi výstup detekce

Spusť detekční bash blok výše a pošli mi výstupy. Nebo prostě řekni *"jsem fresh nováček, nemám nic"* — začneme od `/setup-ssh`.
