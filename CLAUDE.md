# CLAUDE.md — Dev onboarding workspace

Tento repo je **onboarding workspace** pro nové kolegy. Otevři ho jako VS Code workspace a Claude Code extension tě provede setupem dev prostředí.

## Jak začít

Napiš v Claude Code chatu:

> *"Jsem nový, proveď mě onboardingem."*

Claude najde `welcome` skill a začne tě vést.

## Dostupné skills (v `.claude/skills/`)

Doporučené pořadí:

1. **`welcome`** — detekce stavu (SSH, git, gh, glab), rozcestník.
2. **`setup-ssh`** — SSH klíč (ed25519), upload do GitLab + GitHub. Hloubkový tutoriál: co je SSH, rozdíl od hesla/tokenu/OAuth.
3. **`setup-git`** — git config: pull.rebase, pull.ff, user.email. Proč lineární historie.
4. **`install-gh-glab`** — GitHub CLI + GitLab CLI instalace a login.
5. **`claude-concepts`** — Claude architektura: memory vs CLAUDE.md vs skills vs hooks. Kde co leží, co modifikovat.
6. **`install-marketplace`** — volitelné: registrace firemního privátního marketplace. Zeptá se z jaké jsi firmy a adaptuje instrukce.
7. **`next-steps`** — "klonuj projekt na kterém chceš pracovat."
8. **`troubleshoot`** — typické gotchy.

## Pravidla pro Claude

- **Tutoriál styl** — vysvětluj co a PROČ, krok po kroku, čekej na reakci uživatele.
- **Česky** — technické termíny v originále (SSH, git, rebase, atd.).
- **Bezpečnost** — neptej se na hesla, privátní klíče nevypisuj.
- **Permission dialogy** — uživatel bude vidět Allow/Deny, to je normální.
- **Neinstaluj nic bez vysvětlení** — u každého `apt install`, `pip install`, `ssh-keygen` řekni proč.
