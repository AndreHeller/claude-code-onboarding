---
name: install-marketplace
description: "ONBOARDING volitelný krok — registrace firemního privátního Claude Code plugin marketplace. Adaptivní: zeptá se uživatele z jaké je firmy. Pro Slevomat kolegy nainstaluje BI plugin z privátního GitHub marketplace. Pro ostatní vysvětlí jak firemní marketplace obecně funguje a kolega si nastaví svůj. Auto-invoke JEN v onboarding kontextu: 'jsem nový kolega, nastav firemní marketplace', 'onboarding marketplace', 'welcome mě poslal na install-marketplace'. NEVOLAT při obecných dotazech o Claude pluginech nebo marketplaces."
---

# Firemní plugin marketplace (volitelné)

Pokud tvůj tým má **privátní Claude Code plugin marketplace** (s org-wide konvencemi, naming, git workflow, atd.), tady ho nainstalujeme.

## Prerequisites

- **Funkční `claude` CLI v terminálu** — plugin příkazy (`claude plugin marketplace add ...`) fungují **jen v CLI binárce**, ne v chat panelu VS Code extension. Ověř:

  ```bash
  claude --version
  ```

  Pokud `command not found` → spusť nejdřív skill **`install-claude-cli`** a pak se sem vrať.

- **SSH key registrovaný v GitHubu** (pro Slevomat marketplace — repo se klonuje přes SSH) — viz skill `setup-ssh`.

> **⚠ Důležité**: příkazy `claude plugin ...` v této skill **NESPOUŠTĚJ v Claude Code chat panelu** — panel nemá plugin slash commandy. Všechno patří do **bash terminálu** (Ctrl+backtick ve VS Code, nebo Ubuntu ze Start menu).

## Krok 1: Z jaké jsi firmy?

**Zeptej se uživatele**: *"Pracuješ pro konkrétní firmu, která má vlastní Claude plugin marketplace? Pokud ano, řekni mi název firmy."*

### Pokud odpověď = **Slevomat**

Slevomat má privátní marketplace na GitHubu s pluginem `bi` (git workflow konvence, naming patterns, auto-update check).

**Prerequisites pro Slevomat**: přijatá pozvánka do GitHub organizace `slevomat` + SSH key registrovaný v GitHubu (viz `setup-ssh`) + `claude` CLI v terminálu (viz `install-claude-cli`).

Instalace přes Claude Code CLI v terminálu:

```bash
# 1. Registrace marketplace
claude plugin marketplace add git@github.com:slevomat/claude-marketplace.git
```

**Jméno marketplace není jméno repa.** Registruješ URL
`slevomat/claude-marketplace`, ale marketplace se jmenuje **`slevomat-ai`** —
jméno se bere z pole `name` v jeho `marketplace.json`. Příkaz `add` ho na konci
vypíše (`Successfully added marketplace: slevomat-ai`); kdykoli později ho
zjistíš přes `claude plugin marketplace list`. Instalační syntax je
`<plugin>@<jméno-marketplace>`, takže překlep to není.

```bash
# 2. Co katalog nabizi a co uz je nactene
claude plugin marketplace list
claude plugin list
```

**Neinstaluj naslepo.** Katalog obsahuje víc pluginů a u některých dvě varianty:
`x` (HTTPS origin, pro Cowork / claude.ai) a `x-cli` (SSH origin, pro instalaci
z terminálu). Část pluginů navíc může být **už načtená z desktopové aplikace**,
která si je spravuje ve vlastním úložišti. Když takový plugin nainstaluješ ještě
přes CLI, vznikne **druhá spravovaná kopie** — obě se aktualizují nezávisle a
časem se rozejdou.

Postup: porovnej katalog s tím, co už je načtené, a instaluj **jen to, co
chybí**. Když si kolega o nějaký plugin řekne a už ho má odjinud, řekni mu to
a domluvte se, které místo je nadále to hlavní.

```bash
# 3. Instalace bi pluginu (pokud jeste neni)
claude plugin install bi@slevomat-ai
```

Po restartu VS Code jsou skills z `bi` pluginu dostupné. **Vypiš, co plugin
reálně nabízí**, místo abys spoléhal na seznam v této dokumentaci — sada skillů
se mění a natvrdo psaný výčet zastarává:

```bash
ls ~/.claude/plugins/cache/slevomat-ai/bi/*/skills/
```

Ověření: napiš Claude *"jaké git konvence používáme?"* — měl by citovat ze
skillu `git-workflow`.

### Pokud odpověď = **jiná firma** nebo **žádná**

Firemní marketplace je způsob jak tým distribuuje org-wide Claude Code pluginy. Funguje takto:

1. **Tým vytvoří git repo** s `.claude-plugin/marketplace.json` (katalog pluginů).
2. **Kolegové zaregistrují**: `claude plugin marketplace add <git-url>`.
3. **Nainstalují plugin**: `claude plugin install <plugin>@<marketplace>`.
4. **Auto-update** — plugin může mít SessionStart hook pro detekci nových verzí.

Pokud tvůj tým marketplace nemá, **přeskoč tento krok** — pokračuj na `next-steps`.

## Hotovo, co dál

Pokračuj na **`next-steps`** — klonuj projekt na kterém chceš pracovat.
