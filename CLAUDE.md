# CLAUDE.md — Dev onboarding repo

Tento repo hostuje **jeden Claude Code plugin `dev-onboarding`** ([plugins/dev-onboarding/](plugins/dev-onboarding/)) pro nové kolegy. Primární distribuce je přes **firemní Claude Team marketplace** (admin org-wide). Fallback cesty: lokální `claude plugin install` nebo copy-paste PROMPT.md v Claude Desktop.

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
5. **`claude-concepts`** — memory vs CLAUDE.md vs skills vs hooks. Kde co leží.
6. **`install-marketplace`** — volitelné, adaptivní (zeptá se z jaké firmy je kolega).
7. **`next-steps`** — klonuj první projekt.
8. **`troubleshoot`** — on-demand, dlouhodobě užitečný (SSH fails, git auth issues, plugin problems).

## Distribuce

- **Slevomat kolega (Team plan)**: admin nastavil plugin jako *Installed by default* v Team marketplace (GitHub sync z tohoto repa). Plugin je aktivní v Cowork/Code automaticky.
- **Jiná firma nebo individuál**: `claude plugin marketplace add https://github.com/AndreHeller/claude-code-onboarding.git` + `claude plugin install dev-onboarding`.
- **Claude Desktop / žádný plugin marketplace**: copy-paste [PROMPT.md](PROMPT.md) do chatu (fallback, stahuje `install-wsl/SKILL.md` přes URL).

## Lokální dev (pro autora)

```bash
claude plugin marketplace add file:///home/avatar/dev/claude-welcome
claude plugin install dev-onboarding
```

## Pravidla pro Claude

- **Tutoriál styl** — vysvětluj co a PROČ, krok po kroku, čekej na reakci uživatele.
- **Jazyk uživatele** — odpovídej jazykem kterým uživatel píše. Technické termíny v originále (SSH, git, rebase, atd.).
- **Bezpečnost** — neptej se na hesla, privátní klíče nevypisuj.
- **Permission dialogy** — uživatel bude vidět Allow/Deny, to je normální.
- **Neinstaluj nic bez vysvětlení** — u každého `apt install`, `pip install`, `ssh-keygen` řekni proč.
- **Funkční > existenční detekce** — pro stav prostředí použij funkční testy (`ssh -T`, `git config --get`, `gh auth status`), ne kontrolu souborů na standardních cestách.
