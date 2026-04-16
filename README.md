# claude-code-onboarding

**Open-source** Claude Code onboarding pro dev týmy. Provede nového kolegu kompletním setupem dev prostředí — od Windows/WSL install po SSH, git, CLI nástroje a Claude Code koncepty.

## Komu je to určeno

- **Dev / BI inženýr** na Windows — potřebuješ WSL + VS Code + Claude Code extension + SSH + git. ✅
- **Existující Linux/macOS dev** — přeskoč WSL fázi, začni rovnou fází 2 (SSH, git, CLI).
- **Tvůj tým** — forkni, uprav skills na vlastní konvence, přidej vlastní marketplace krok.

## Jak začít

### Fáze 1: Windows → WSL + VS Code + Claude Code extension

1. Otevři **Claude Desktop** chat (z [claude.ai/download](https://claude.ai/download)).
2. Copy-paste prompt z **[PROMPT.md](PROMPT.md)**.
3. Claude tě provede WSL instalací krok za krokem.
4. Na konci dostaneš instrukce pro fázi 2.

### Fáze 2: SSH + git + CLI + Claude koncepty

```bash
cd ~/dev
git clone https://github.com/AndreHeller/claude-code-onboarding.git
code claude-code-onboarding
```

V Claude Code extension napiš: *"Jsem nový, proveď mě onboardingem."*

Claude přečte CLAUDE.md + `.claude/skills/` z tohoto workspace a provede tě:
- **SSH klíče** (ed25519, per-service pro GitLab + GitHub, hloubkový tutoriál co je SSH/token/OAuth)
- **Git config** (pull.rebase, pull.ff, user.email — s vysvětlením proč lineární historie)
- **GitHub CLI + GitLab CLI** (instalace, login)
- **Claude koncepty** (memory vs CLAUDE.md vs skills — kde co leží, co modifikovat)
- **Firemní marketplace** (volitelné — zeptá se z jaké jsi firmy a adaptuje se)
- **Handoff** na konkrétní projekt (klonuj, jeho CLAUDE.md přebírá)

### Fáze 3: Tvůj projekt

Po fázi 2 klonuješ svůj projekt. Pokud má `CLAUDE.md` a `.claude/skills/`, Claude tě tam provede project-specific setupem.

## Struktura

```
claude-code-onboarding/
├── CLAUDE.md                          ← instrukce pro Claude Code v tomto workspace
├── PROMPT.md                          ← copy-paste prompt pro Claude Desktop (fáze 1)
├── .claude/skills/                    ← 8 skills pro fázi 2
│   ├── welcome/                       ← entry point, detekce stavu
│   ├── setup-ssh/                     ← SSH klíč + tutoriál auth metod
│   ├── setup-git/                     ← git config + rebase/ff/lineární historie
│   ├── install-gh-glab/               ← GitHub CLI + GitLab CLI
│   ├── claude-concepts/               ← memory vs CLAUDE.md vs skills
│   ├── install-marketplace/            ← volitelné: firemní plugin marketplace
│   ├── next-steps/                    ← handoff na tvůj projekt
│   └── troubleshoot/                  ← typické gotchy
└── plugins/wsl-onboarding/            ← zdroj pro PROMPT.md (fáze 1)
```

## Přizpůsobení pro tvůj tým

1. **Forkni** tento repo.
2. **Uprav skills** — `setup-git` (tvoje konvence), `setup-ssh` (tvoje služby).
3. **Uprav `install-marketplace`** — přidej vlastní firemní marketplace URL a pluginy.
4. **Uprav `next-steps`** — tvoje projekty místo vlastních příkladů.

## Autor

André Heller — [mail@andreheller.cz](mailto:mail@andreheller.cz) · [GitHub](https://github.com/AndreHeller)
