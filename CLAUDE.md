# CLAUDE.md — Dev onboarding repo

Tento repo **JE** Claude Code plugin `dev-onboarding` (plugin manifest v [.claude-plugin/plugin.json](.claude-plugin/plugin.json), skills v [skills/](skills/)) pro nové kolegy. Současně slouží jako **marketplace shim** ([.claude-plugin/marketplace.json](.claude-plugin/marketplace.json)) pro individuální install. Primární distribuce je přes **firemní Claude Team marketplace** (admin org-wide, Cowork). Fallback cesty: lokální `claude plugin install` nebo copy-paste PROMPT.md v Claude Desktop.

## Jak Claude pozná, že má onboardit

Uživatel napíše něco jako:

> *"Jsem nový, proveď mě onboardingem."*
> *"Nastav mi dev prostředí."*
> *"Pokračuj onboardingem."*

Claude najde skill **`welcome`** v plugin `dev-onboarding` a aktivuje ho. Welcome je **router**:

1. Detekuje OS (`uname -s`, `$WSL_DISTRO_NAME`).
2. Funkčním testem zjistí stav SSH, git, gh, glab (ne existenční — různí kolegové mají klíče pojmenované jinak).
3. Ukáže dashboard ✓/✗ a nasměruje na další relevantní skill.

## Skills v plugin `dev-onboarding`

Pořadí v routeru welcome:

1. **`install-wsl`** — jen pro Windows uživatele v Cowork (Desktop). Ostatní skip.
2. **`setup-ssh`** — per-service SSH klíče pro GitHub + GitLab. Hloubkový tutoriál.
3. **`setup-git`** — user.name, user.email, pull.rebase=true, pull.ff=only. Vysvětlí proč.
4. **`install-gh-glab`** — GitHub CLI + GitLab CLI, auth login.
5. **`claude-concepts`** — memory vs CLAUDE.md vs skills vs hooks. Kde co leží na disku, co je persistent.
6. **`plugin-building-blocks`** — z čeho se plugin uvnitř skládá (skill / hook / command / agent / MCP / marketplace). Komplement ke `claude-concepts` — ten řeší storage pohled, tenhle architektonický.
7. **`install-claude-cli`** — prerekvizita pro marketplace: `claude` CLI binárka v terminálu (VS Code extension ji nevystavuje). Skip pokud `claude --version` prošlo v detekci.
8. **`install-marketplace`** — adaptivní (welcome router se aktivně zeptá na firmu, nenechává to na iniciativě uživatele).
9. **`next-steps`** — klonuj první projekt.
10. **`troubleshoot`** — on-demand, dlouhodobě užitečný (SSH fails, git auth issues, plugin problems).

## Distribuce

- **Slevomat kolega (Team plan)**: admin nastavil plugin jako *Installed by default* v Team marketplace (GitHub sync z tohoto repa). Plugin je aktivní v Cowork/Code automaticky.
- **Jiná firma nebo individuál**: `claude plugin marketplace add https://github.com/AndreHeller/claude-code-onboarding.git` + `claude plugin install dev-onboarding`.
- **Claude Desktop / žádný plugin marketplace**: copy-paste [PROMPT.md](PROMPT.md) do chatu (fallback, stahuje `install-wsl/SKILL.md` přes URL).

## Lokální dev (pro autora)

```bash
claude plugin marketplace add file:///home/avatar/dev/personal/claude-code-onboarding
claude plugin install dev-onboarding
```

## Pravidla pro Claude

- **Tutoriál styl** — vysvětluj co a PROČ, krok po kroku, čekej na reakci uživatele.
- **Jazyk uživatele** — odpovídej jazykem kterým uživatel píše. Technické termíny v originále (SSH, git, rebase, atd.).
- **Bezpečnost** — neptej se na hesla, privátní klíče nevypisuj.
- **Permission dialogy** — uživatel bude vidět Allow/Deny, to je normální.
- **Neinstaluj nic bez vysvětlení** — u každého `apt install`, `pip install`, `ssh-keygen` řekni proč.
- **Funkční > existenční detekce** — pro stav prostředí použij funkční testy (`ssh -T`, `git config --get`, `gh auth status`), ne kontrolu souborů na standardních cestách.

## Git workflow

**Archetyp:** Personal & self-maintained (= open-source na GitHub). Detail v plugin skillu **`bi:git-workflow`** → [`archetype-personal.md`](https://gitlab.com/slevomat/ai/claude-plugin-bi/-/blob/main/skills/git-workflow/references/archetype-personal.md).

**Per-repo specifika:**
- **Non-Slevomat remote** (= `AndreHeller/claude-code-onboarding` na GitHub) → plugin BI hook `check-git.sh` **silent-pass** (= hook validuje jen Slevomat repos, hobby/open-source repa nechává být).
- **Single owner** (André), self-merge OK, žádný hard reviewer requirement.
- **PR workflow přes `gh pr create`** (= GitHub remote, ne `glab`). Pro Slevomat kolegy je viditelnost přes Slevomat Team marketplace (admin GitHub sync), pro externí přes manual marketplace install.
- **Versioning:** Repo a plugin žijí v jednom (= plugin manifest je v rootu repa). Plugin má SemVer v [.claude-plugin/plugin.json](.claude-plugin/plugin.json). Release branch konvence (Slevomat marketplace): `release/v<major>` (= aktuálně `release/v1`).
