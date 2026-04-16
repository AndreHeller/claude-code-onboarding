---
name: setup-git
description: "Git konfigurace dle Slevomat konvencí — user.name, user.email (per-repo identity pro firemní vs osobní), pull.rebase=true, pull.ff=only, init.defaultBranch=main. Hloubkové vysvětlení rebase vs merge, lineární vs nelineární historie, fast-forward merge, proč obě pojistky. Spouští se po setup-ssh. Auto-invoke při 'nastav mi git', 'git konvence', 'git config', 'co je rebase'."
---

# Git konfigurace — Slevomat konvence

Po SSH máš infrastrukturu pro přístup k repům. Teď nastavíme **jak se git chová** — konvence co Slevomat používá napříč projekty. Většinu z toho si nastavíš **jednou** a pak funguje automaticky.

Ale než začneme konfigurovat — **musíš pochopit PROČ**. Jinak to jsou jen magické příkazy co jednou nastavíš a za 3 měsíce, když něco selže, nebudeš vědět co se děje. Tak si to projdeme.

## Část 1: Kde git config žije

Git má **tři úrovně** konfigurace (od nejvyšší priority):

1. **Repository-specific** (`<repo>/.git/config`) — per-project. Override pro jedno konkrétní repo.
2. **User-level** (`~/.gitconfig`) — default pro tebe napříč všemi projekty. **Tady nastavujeme Slevomat defaults.**
3. **System-wide** (`/etc/gitconfig`) — OS-level. Obvykle nesaháme.

**Důležité**: vyšší priorita přebíjí nižší. Pokud global říká `email = firemní` a repo-local říká `email = osobní`, **v tom repu vyhraje osobní**.

Příkazy:
- `git config --global <klíč> <hodnota>` → zapíše do `~/.gitconfig`.
- `git config <klíč> <hodnota>` → zapíše do aktuálního repa (`.git/config`).
- `git config --global --list` → zobrazí všechny globální nastavení.

## Část 2: Identita — user.name a user.email

Identita se objevuje v **každém commitu** — `git log`, GitLab/GitHub UI, blame, audit trail.

```bash
git config --global user.name "Tvé Jméno"
git config --global user.email "tvuj.email@slevomat.cz"
```

**Důležité**: `user.email` musí být **registrovaný v GitLabu/GitHubu**. Jinak tvé commity budou "unverified" a nepropojí se s tvým profilem.

### Firemní vs osobní email — per-repo override

**Problém**: nastavíš global `andre.heller@slevomat.cz`. Teď **všechny** commity na tomhle stroji — i do osobních projektů na GitHubu — budou podepsané firemním emailem. To nechceš.

**Řešení 1** — ruční override v konkrétním repu:

```bash
# V osobním repu:
cd ~/personal/muj-projekt
git config user.email "mail@andreheller.cz"    # bez --global!
```

Tohle repo bude mít osobní email, zbytek firemní. Ale musíš to udělat **ručně** pro každé osobní repo.

**Řešení 2** — automatické per-folder pravidlo (elegantnější):

```ini
# ~/.gitconfig

[user]
    name = André Heller
    email = andre.heller@slevomat.cz    # default = firemní

# Pokud repo je v ~/personal/, použij osobní email
[includeIf "gitdir:~/personal/"]
    path = ~/.gitconfig-personal
```

```ini
# ~/.gitconfig-personal (vytvoř ručně)
[user]
    email = mail@andreheller.cz
```

Výsledek: **vše v `~/dev/` (firemní)** → automaticky Slevomat email. **Vše v `~/personal/`** → automaticky osobní email. Nula manuální práce po setupu.

**Pro Slevomat kolegu**: global = firemní (`@slevomat.cz`). Per-repo nebo conditional include = osobní. Většina práce je firemní, tak firemní jako default dává smysl.

## Část 3: Co je lineární vs nelineární historie (a proč na tom záleží)

Než nastavíme `pull.rebase` a `pull.ff`, musíš pochopit **proč**. Tohle je nejdůležitější koncept v Slevomat git workflow.

### Nelineární historie (merge commits)

Představ si: ty a kolega oba pracujete na `main`. Oba máte lokální commit. Ty pustíš `git pull` (default chování = merge):

```
         kolega commitnul
              ↓
main:  A ── B ── C (remote)
              \
               D (tvůj lokální commit)
```

Git vytvoří **merge commit** `M`:

```
main:  A ── B ── C ── M
              \      /
               D ───┘
```

`M` je "merge commit" — technický artefakt, neobsahuje žádnou business změnu. Jeho message je:

```
Merge remote-tracking branch 'origin/main' into main
```

**Problém**: tohle se opakuje při **každém** `git pull` kdy máš lokální commity. Za měsíc tvůj `git log` vypadá takto:

```
a1b2c3d Merge remote-tracking branch 'origin/main' into main
f4e5d6a Přidat voucher cancel logiku
b7c8d9e Merge remote-tracking branch 'origin/main' into main
1a2b3c4 Merge remote-tracking branch 'origin/main' into main
e5f6a7b Fix: product_vertical_travel CZ/Non-CZ detekce
8c9d0e1 Merge remote-tracking branch 'origin/main' into main
```

**Polovina historie je noise.** Merge commits nepopisují business změny, jen mechaniku gitu. `git bisect` (binární vyhledávání chyby) musí přeskakovat merge commits. `git revert` merge commitu je komplikovaný (musíš specifikovat `-m 1` parent). `git blame` ukazuje merge commit místo skutečného autora.

### Lineární historie (rebase)

Stejná situace, ale místo merge uděláš **rebase**:

```
         kolega commitnul
              ↓
main:  A ── B ── C (remote)
              \
               D (tvůj lokální commit)
```

Rebase **přesune** tvůj commit `D` **nad** remote commity:

```
main:  A ── B ── C ── D' (tvůj commit, ale nový hash)
```

Žádný merge commit. Historie je **rovná čára**. `D'` má nový hash (protože parent se změnil z `B` na `C`), ale obsah (diff) je identický.

`git log` za měsíc:

```
a1b2c3d Přidat voucher cancel logiku
f4e5d6a Fix: product_vertical_travel CZ/Non-CZ detekce
b7c8d9e Refactor: stg_vouchers CTE naming
1a2b3c4 Nový model: fact_deal_visits
```

**Čistě business commity.** Žádný noise. `git bisect` funguje predikovatelně. `git revert` je triviální. `git blame` ukazuje skutečného autora.

### Proč Slevomat chce lineární

- **Čitelnost**: `git log --oneline` je čistý příběh projektu.
- **Bisect**: binární vyhledávání chyby ("`git bisect` — který commit to rozbil?") funguje přesně, nemusí přeskakovat merge commits.
- **Revert**: `git revert <hash>` pro libovolný commit je single command, žádný `-m 1` parent confusion.
- **Blame**: `git blame soubor.sql` ukazuje skutečného autora řádku, ne merge commit.
- **Code review**: MR diff v GitLabu ukazuje přesně co kolega změnil, bez merge noise.

## Část 4: `pull.rebase=true` — jak to funguje

```bash
git config --global pull.rebase true
```

**Co to dělá**: změní chování `git pull` z `fetch + merge` na `fetch + rebase`.

**Bez tohoto nastavení** (default Git):

```bash
git pull
# = git fetch origin + git merge origin/main
# → vytvoří merge commit pokud máš lokální commity
```

**S tímto nastavením**:

```bash
git pull
# = git fetch origin + git rebase origin/main
# → přesune tvé lokální commity nahoru nad remote
# → žádný merge commit
```

### Co se stane při konfliktu

Rebase může narazit na **konflikt** — tvůj commit mění stejný řádek co remote commit. Git se zastaví:

```
CONFLICT (content): Merge conflict in models/stg_products.sql
error: could not apply d1e2f3a... Tvůj commit message
```

**Fix**:
1. Otevři soubor s konfliktem, vyřeš (git označuje `<<<<<<< HEAD` / `=======` / `>>>>>>> commit`).
2. `git add <soubor>` — označ jako vyřešený.
3. `git rebase --continue` — pokračuj v rebase.

**Nebo abort** (vrátit se před rebase):
```bash
git rebase --abort
```

Konflikty při rebase jsou **identické** s konflikty při merge — stejná práce, stejný výsledek. Jediný rozdíl je `--continue` místo `git commit`.

### Kdy rebase NEDĚLAT

- **Na sdílené větvi kam jiný kolega pushuje** — rebase přepisuje commit hashe. Pokud kolega má staré hashe, jeho push selže. Proto rebasujeme **jen lokální nepushnuté commity** (což `git pull --rebase` dělá automaticky správně).
- **Na main po push** — proto máme `--force-with-lease` jako pojistku (vysvětleno v `git-workflow` skillu).

`pull.rebase=true` je bezpečný — rebasuje **jen** tvé lokální nepushnuté commity nad nové remote commity. Sdílenou historii nemění.

## Část 5: `pull.ff=only` — záložní pojistka

```bash
git config --global pull.ff only
```

### Co je fast-forward merge

**Fast-forward** = speciální případ merge kdy **není co mergovat**. Tvůj `main` je prostě pozadu za remote — stačí posunout pointer:

```
Tvůj stav:     A ── B         (main)
Remote stav:   A ── B ── C ── D  (origin/main)

Fast-forward:  A ── B ── C ── D  (main = origin/main)
```

Žádný merge commit, žádný rebase — prostě posun. **Toto je vždy bezpečné.**

**Non-fast-forward** = tvůj branch a remote se **rozvětvily** (oba mají commity co ten druhý nemá). Tady Git musí buď merge (vytvoří merge commit) nebo rebase.

### Co `pull.ff=only` dělá

Říká Gitu: **pokud `git pull` by vyžadoval non-fast-forward merge, SELŽI**. Nevytvoř tiše merge commit.

```
fatal: Not possible to fast-forward, aborting.
```

### Proč obě pojistky (defense in depth)

**`pull.rebase=true`** je primární — řídí jak se `git pull` chová. Místo merge dělá rebase → tvoje commity nahoru, žádný merge commit.

**`pull.ff=only`** je záložní pojistka pro edge case:

1. Kolega spustí `git pull --no-rebase` (explicitní override).
2. Nebo má repo-local `.git/config` s `pull.rebase=false` (per-repo override).
3. Bez `pull.ff=only` by Git **tiše vytvořil merge commit** → rozbije lineární historii.
4. S `pull.ff=only` Git **selže** s explicit chybou → kolega musí vědomě rozhodnout (rebase? merge? abort?).

**Analogie**: `pull.rebase=true` je zámek na dveřích. `pull.ff=only` je alarm — i pokud někdo zámek obejde, alarm zařve.

## Část 6: `init.defaultBranch=main`

```bash
git config --global init.defaultBranch main
```

**Co dělá**: nový repo (`git init`) použije `main` jako default branch name (místo staršího `master`).

**Proč**: moderní standard. GitHub a GitLab default na `main` od ~2020. Sjednocení, žádný mismatch "u mě je master, u tebe main".

## Část 7: (Volitelné) `core.editor`

**Editor pro commit messages** (když `git commit` bez `-m` flag):

```bash
git config --global core.editor "code --wait"   # VS Code
# nebo "nano" (jednoduchý terminálový editor)
# nebo "vim" (pokud vim znáš)
```

`--wait` flag u VS Code = terminál čeká až zavřeš editor tab, pak pokračuje s commitem. Bez něj Git okamžitě commitne s prázdnou zprávou.

## Část 8: Aplikace

Spustím všech 5 příkazů najednou (s tvým permission). **Dřív než spustím**, zeptám se tě na:
- **Tvé jméno** (jak se chceš zobrazovat v commitech).
- **Firemní email** (registrovaný v GitLab + GitHub).

```bash
git config --global user.name "Tvé Jméno"
git config --global user.email "tvuj.email@slevomat.cz"
git config --global pull.rebase true
git config --global pull.ff only
git config --global init.defaultBranch main
```

## Část 9: Ověření

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

## Část 10: Kde to leží na disku

```bash
cat ~/.gitconfig
```

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

Jednoduchý INI formát. Můžeš **editovat přímo** (VS Code, nano), ale `git config --global` je bezpečnější (syntax validation).

### Per-repo identity (volitelné — pro pokročilé)

Pokud máš na stejném stroji **firemní i osobní projekty** a chceš **automaticky** přepínat email:

```ini
# ~/.gitconfig — přidej na konec:
[includeIf "gitdir:~/personal/"]
    path = ~/.gitconfig-personal
```

```bash
# Vytvoř ~/.gitconfig-personal:
cat > ~/.gitconfig-personal << 'EOF'
[user]
    email = tvuj.osobni@email.cz
EOF
```

Výsledek: vše v `~/dev/` → automaticky `@slevomat.cz`. Vše v `~/personal/` → automaticky osobní email. Nula manuální práce.

## Část 11: Hotovo, co dál

Máš Slevomat git konvence. Commity budou mít tvou identitu, `git pull` bude rebasovat (lineární historie), fast-forward pojistka chrání před accidental merge commits.

Pokračuj na **`/install-gh-glab`** — nainstalujeme GitHub CLI (`gh`) a GitLab CLI (`glab`) pro snadnější interakci s repa.
