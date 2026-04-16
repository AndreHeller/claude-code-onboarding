---
name: setup-git
description: "Git konfigurace dle Slevomat konvencí — user.name, user.email, pull.rebase=true, pull.ff=only, init.defaultBranch=main. Vysvětluje PROČ každá volba (zejména rebase vs merge = lineární historie, ff=only = pojistka proti override). Spouští se po setup-ssh. Auto-invoke když user řekne 'nastav mi git', 'git konvence', 'git config'."
---

# Git konfigurace — Slevomat konvence

Po SSH máš infrastruktura. Teď nastavíme **jak se git chová** — konvence co Slevomat používá napříč projekty pro čistou lineární historii.

## Část 1: Kde git config žije

Git má **tři úrovně** konfigurace (od nejvyšší priority):

1. **Repository-specific** (`<repo>/.git/config`) — per-project. Např. různá email adresa v open-source repu vs firemní.
2. **User-level** (`~/.gitconfig`) — default pro tebe napříč všemi projekty. **Tady nastavujeme Slevomat defaults.**
3. **System-wide** (`/etc/gitconfig`) — OS-level. Obvykle nesaháme.

Příkazy:
- `git config --global <klíč> <hodnota>` — zapíše do `~/.gitconfig`.
- `git config <klíč> <hodnota>` — zapíše do aktuálního repa (per-repo override).
- `git config --global --list` — zobrazí všechny globální nastavení.

## Část 2: Slevomat git konvence (5 pravidel)

### 1. `user.name` a `user.email`

Identita pro commits. Objeví se v `git log`, GitLab/GitHub UI, blame.

```bash
git config --global user.name "Tvé Jméno"
git config --global user.email "tvuj.email@slevomat.cz"
```

**Důležité**: `user.email` musí být **registrovaný v GitLabu/GitHubu**. Jinak tvé commity budou "unverified" a nepropojí se s tvým profilem. GitLab: Settings → Emails → Add. GitHub: Settings → Emails → Add.

### 2. `pull.rebase=true` (primární)

```bash
git config --global pull.rebase true
```

**Co dělá**: `git pull` bude dělat **rebase** místo **merge**.

**Rozdíl**:
- **Merge pull** (default Git): když máš lokální commity + remote má nové, Git vytvoří **merge commit** "Merge remote-tracking branch 'origin/main' into main". Historie vypadá jako rozvětvený graf.
- **Rebase pull**: tvé lokální commity se "přepíchnou" **nahoru** nad nové remote commity, žádný merge commit. Historie je **lineární** (rovná čára).

**Proč Slevomat chce lineární**: 
- Snazší čitelnost `git log --oneline`. 
- `git bisect` (binární vyhledávání chyby) funguje predikovatelně.
- `git revert` konkrétního commitu je trivial.
- Merge commits v historii jsou noise — popisují mechaniku gitu, ne business změny.

### 3. `pull.ff=only` (pojistka)

```bash
git config --global pull.ff only
```

**Co dělá**: pokud `git pull` má provést merge (což s `pull.rebase=true` by neměl, ale někdo může override přes `git pull --no-rebase`), povolí merge **jen** pokud je fast-forward (= nic k mergování, stačí posun pointeru).

**Proč**: fail-safe. Když někdo overridne rebase s `--no-rebase`, Git selže s:
```
fatal: Not possible to fast-forward, aborting.
```

místo aby tiše vytvořil merge commit. Vynutí **vědomé rozhodnutí** — rebase, merge, nebo abort.

### 4. `init.defaultBranch=main`

```bash
git config --global init.defaultBranch main
```

**Co dělá**: nový repo (`git init`) použije `main` jako default branch name (místo staršího `master`).

**Proč**: moderní standard. GitHub a GitLab default na `main`. Sjednocení.

### 5. (Volitelné) `core.editor` a `commit.gpgsign`

**Editor pro commit messages** (když `git commit` bez `-m`):
```bash
git config --global core.editor "code --wait"   # VS Code
# nebo "nano", "vim" podle preference
```

**GPG signing commitů** (advanced, pro audit trail):
```bash
git config --global commit.gpgsign true
git config --global user.signingkey <GPG_KEY_ID>
```

Pro Slevomat dev není signing povinný. Skippni.

## Část 3: Aplikace

Spustím všech 5 příkazů najednou (s tvým permission):

```bash
git config --global user.name "Tvé Jméno"
git config --global user.email "tvuj.email@slevomat.cz"
git config --global pull.rebase true
git config --global pull.ff only
git config --global init.defaultBranch main
```

**Dřív než spustím**, zeptám se tě na tvé jméno a email — ty dva potřebuji správně.

## Část 4: Ověření

```bash
git config --global --list | grep -E "user|pull|init"
```

Očekávaný výstup:
```
user.name=Tvé Jméno
user.email=tvuj.email@slevomat.cz
pull.rebase=true
pull.ff=only
init.defaultBranch=main
```

Pokud tam něco chybí, zopakuj `git config --global ...` pro ten klíč.

## Část 5: Kam git config ukládá data

```bash
cat ~/.gitconfig
```

Typicky:
```ini
[user]
	name = Tvé Jméno
	email = tvuj.email@slevomat.cz
[pull]
	rebase = true
	ff = only
[init]
	defaultBranch = main
```

Jednoduchý INI formát. Můžeš **editovat přímo**, ale `git config --global` je bezpečnější (syntax validation).

## Část 6: Hotovo, co dál

Máš Slevomat git konvence. Commity budou mít tvou identitu, historie zůstane čistá.

Pokračuj na **`/install-gh-glab`** — nainstalujeme GitHub CLI (`gh`) a GitLab CLI (`glab`) pro snadnější interakci s repa (issues, PRs, MRs bez browseru).
