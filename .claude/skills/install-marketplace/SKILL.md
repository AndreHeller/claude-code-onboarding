---
name: install-marketplace
description: "Volitelný krok — registrace firemního privátního Claude Code plugin marketplace. Zeptá se uživatele z jaké je firmy a adaptuje instrukce. Pro Slevomat kolegy: nainstaluje bi plugin z privátního GitLab marketplace. Pro ostatní: vysvětlí jak firemní marketplace funguje obecně. Auto-invoke při 'marketplace', 'firemní plugin', 'org-wide konvence'."
---

# Firemní plugin marketplace (volitelné)

Pokud tvůj tým má **privátní Claude Code plugin marketplace** (s org-wide konvencemi, naming, git workflow, atd.), tady ho nainstalujeme.

## Krok 1: Z jaké jsi firmy?

**Zeptej se uživatele**: *"Pracuješ pro konkrétní firmu, která má vlastní Claude plugin marketplace? Pokud ano, řekni mi název firmy."*

### Pokud odpověď = **Slevomat**

Slevomat má privátní marketplace na GitLabu s pluginem `bi` (git workflow konvence, naming patterns, auto-update check).

**Prerequisites**: SSH key registrovaný v GitLabu (viz skill `setup-ssh`).

Instalace přes Claude Code CLI v terminálu:

```bash
# 1. Registrace Slevomat marketplace
claude plugin marketplace add git@gitlab.com:slevomat/ai/claude-marketplace.git

# 2. Instalace bi pluginu
claude plugin install bi@slevomat-ai
```

Po restartu VS Code jsou skills z bi pluginu dostupné:
- **`git-workflow`** — Slevomat git konvence + git config audit.
- **`naming-conventions`** — Slevomat BI naming patterns.
- **`update-check`** — auto-detekce aktualizací (1× za 24h).

Ověření: napiš Claude *"jaké git konvence používáme?"* — měl by citovat z `git-workflow` skill.

### Pokud odpověď = **jiná firma** nebo **žádná**

Firemní marketplace je způsob jak tým distribuuje org-wide Claude Code pluginy. Funguje takto:

1. **Tým vytvoří git repo** s `.claude-plugin/marketplace.json` (katalog pluginů).
2. **Kolegové zaregistrují**: `claude plugin marketplace add <git-url>`.
3. **Nainstalují plugin**: `claude plugin install <plugin>@<marketplace>`.
4. **Auto-update** — plugin může mít SessionStart hook pro detekci nových verzí.

Pokud tvůj tým marketplace nemá, **přeskoč tento krok** — pokračuj na `next-steps`.

## Hotovo, co dál

Pokračuj na **`next-steps`** — klonuj projekt na kterém chceš pracovat.
