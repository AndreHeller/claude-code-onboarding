# claude-code-onboarding

**Open-source** Claude Code plugin `dev-onboarding` pro dev týmy. Jeden plugin provede nového kolegu **kompletním** setupem — od Windows/WSL instalace přes SSH, git, CLI nástroje, Claude koncepty, volitelně firemní plugin marketplace, až po klonování prvního projektu.

## Komu je to určeno

- **Dev / BI inženýr na Windows** — plugin obsahuje `install-wsl` skill (WSL2 + Ubuntu + VS Code Remote-WSL + Claude Code extension).
- **Existující Linux / macOS dev** — plugin router přeskočí WSL a začne rovnou SSH/git/CLI fází.
- **Firma** — admin nainstaluje plugin org-wide přes Claude Team marketplace. Forkni, uprav skills na vlastní konvence.

## Tři cesty instalace

### 1. Team / Enterprise plan (doporučeno pro firmy)

Admin v [Claude Team marketplace](https://support.claude.com/en/articles/13837433-manage-claude-cowork-plugins-for-your-organization) nastaví tento repo jako GitHub sync source a plugin `dev-onboarding` označí jako *Installed by default*. Noví kolegové ho mají v Cowork / Claude Code automaticky.

### 2. Individuální instalace přes marketplace

```bash
claude plugin marketplace add https://github.com/AndreHeller/claude-code-onboarding.git
claude plugin install dev-onboarding
```

Pak napíše v Claude Code (nebo Cowork): *"Jsem nový, proveď mě onboardingem."*

### 3. Fallback pro Claude Desktop bez pluginu (Windows nováček)

Kolega, co ještě nemá Claude Code (typicky Windows bez WSL), má **Claude Desktop**. Plugin tam neexistuje, ale:

1. Otevře **Claude Desktop** chat ([claude.ai/download](https://claude.ai/download)).
2. Copy-paste prompt z **[PROMPT.md](PROMPT.md)**.
3. Claude Desktop fetchne `install-wsl` tutoriál přes URL a provede WSL instalací.
4. Po WSL instalaci kolega pokračuje cestou 1 nebo 2 (plugin v Claude Code/Cowork).

## Co plugin udělá

`welcome` skill funguje jako **router**:

1. Detekce OS (`uname -s`, `$WSL_DISTRO_NAME`) + funkční testy (ne existenční — různí kolegové mají klíče pojmenované jinak):
   - `ssh -T git@github.com` / `ssh -T git@gitlab.com`
   - `git config --get user.email`, `gh auth status`, `glab auth status`
2. Dashboard **✓ máš / ✗ chybí**.
3. Nasměrování na relevantní další skill:

| Skill | Co dělá |
|---|---|
| `install-wsl` | jen Windows v Cowork — WSL2 + Ubuntu + VS Code + Claude Code |
| `setup-ssh` | per-service SSH klíče pro GitHub + GitLab, hloubkový tutoriál |
| `setup-git` | user.name/email, pull.rebase=true, pull.ff=only + proč |
| `install-gh-glab` | GitHub CLI + GitLab CLI, auth login |
| `claude-concepts` | memory vs CLAUDE.md vs skills vs hooks |
| `install-marketplace` | volitelné, adaptivní (Slevomat BI / generic) |
| `next-steps` | klonuj první projekt, přepni workspace |
| `troubleshoot` | on-demand — SSH/git fails, plugin problems, WSL gotchy |

## Struktura repa

```
claude-code-onboarding/
├── .claude-plugin/
│   └── marketplace.json                    ← pro lokální dev (autor) / GitHub sync source
├── CLAUDE.md                               ← instrukce pro Claude v tomto workspace
├── PROMPT.md                               ← fallback copy-paste prompt pro Claude Desktop
└── plugins/
    └── dev-onboarding/
        ├── .claude-plugin/plugin.json
        └── skills/
            ├── welcome/                    ← router (OS + funkční stav detekce)
            ├── install-wsl/
            ├── setup-ssh/
            ├── setup-git/
            ├── install-gh-glab/
            ├── claude-concepts/
            ├── install-marketplace/
            ├── next-steps/
            └── troubleshoot/
```

## Přizpůsobení pro tvůj tým

1. **Forkni** tento repo.
2. **Uprav skills** — `setup-git` (týmové konvence), `setup-ssh` (používané služby).
3. **Uprav `install-marketplace`** — přidej vlastní firemní marketplace URL a pluginy (adaptivní skill už má rozšiřitelnou strukturu).
4. **Uprav `next-steps`** — odkazy na tvoje projekty.
5. **Publikuj** — Team admin přidá tvůj fork jako GitHub sync source.

## Lokální dev (pro autora/přispěvatele)

```bash
claude plugin marketplace add file:///home/avatar/dev/claude-welcome
claude plugin install dev-onboarding
claude plugin validate plugins/dev-onboarding
```

## Autor

André Heller — [mail@andreheller.cz](mailto:mail@andreheller.cz) · [GitHub](https://github.com/AndreHeller)
