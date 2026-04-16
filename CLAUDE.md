# CLAUDE.md — Slevomat AI dev onboarding

Tento repo je **onboarding workspace** pro nové Slevomat AI kolegy. Otevři ho jako VS Code workspace a Claude Code extension tě provede setupem dev prostředí.

## Co tento workspace dělá

Provede tě od bodu "mám WSL + VS Code + Claude Code extension" až po "mám SSH, git, přístup k Slevomat pluginům, můžu klonovat projekty". Hloubkový tutoriál styl — každý krok vysvětlí co, proč, kde.

## Jak začít

Napiš v Claude Code chatu:

> *"Jsem nový Slevomat AI kolega po WSL setupu. Proveď mě onboardingem."*

Claude najde `welcome` skill a začne tě vést.

## Dostupné skills (v `.claude/skills/`)

Doporučené pořadí:

1. **`welcome`** — detekce stavu (SSH, git, gh, glab), rozcestník na zbylé skills.
2. **`setup-ssh`** — SSH klíč (ed25519), upload do GitLab + GitHub. Hloubkový tutoriál: co je SSH, rozdíl od hesla/tokenu/OAuth, kde klíče leží.
3. **`setup-git`** — git config dle Slevomat konvencí (pull.rebase, pull.ff, user.email). Proč lineární historie.
4. **`install-gh-glab`** — GitHub CLI + GitLab CLI instalace a login.
5. **`claude-concepts`** — vysvětlení Claude architektury: memory vs CLAUDE.md vs skills vs hooks. Kde co leží, co modifikovat.
6. **`install-slevomat-marketplace`** — registrace privátního Slevomat GitLab marketplace + instalace `bi` pluginu (org-wide git konvence, naming).
7. **`next-steps`** — "klonuj Slevomat projekt na kterém chceš pracovat. Každý projekt má vlastní CLAUDE.md a lokální skills."
8. **`troubleshoot`** — typické gotchy (SSH denied, git auth, Claude extension stav, WSL mount).

## Pravidla pro Claude

- **Tutoriál styl** — vysvětluj co a PROČ, krok po kroku, čekej na reakci uživatele.
- **Česky** — technické termíny v originále (SSH, git, rebase, atd.).
- **Bezpečnost** — neptej se na hesla, privátní klíče nevypisuj, soukromé tokeny nezobrazuj.
- **Permission dialogy** — uživatel bude vidět Allow/Deny dialogy, to je normální. V začátcích ať dává Allow (izolované WSL prostředí).
- **Neinstaluj nic bez vysvětlení** — u každého `apt install`, `pip install`, `ssh-keygen` řekni proč.
