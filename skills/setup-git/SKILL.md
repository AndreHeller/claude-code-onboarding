---
name: setup-git
description: "ONBOARDING krok — první git config pro nového kolegu. Nastaví globální user.name, user.email, pull.rebase=true, pull.ff=only, init.defaultBranch=main dle týmových konvencí. Hloubkové vysvětlení rebase vs merge, lineární historie, fast-forward merge, per-repo identity (firemní vs osobní email). Auto-invoke JEN v onboarding kontextu: 'jsem nový kolega, nastav mi git', 'onboarding git config', 'dev onboarding git', 'welcome mě poslal na setup-git'. NEVOLAT při obecných dotazech o gitu nebo rebase — ty řeší obecné znalosti."
---

# Git konfigurace — Git konvence (best practices)

Po SSH máš infrastrukturu pro přístup k repům. Teď nastavíme **jak se git chová** — konvence co doporučujeme napříč projekty. Většinu z toho si nastavíš **jednou** a pak funguje automaticky.

Ale než začneme konfigurovat — **musíš pochopit PROČ**. Jinak to jsou jen magické příkazy co jednou nastavíš a za 3 měsíce, když něco selže, nebudeš vědět co se děje. Tak si to projdeme.

## Část 1: Kde git config žije

Git má **tři úrovně** konfigurace (od nejvyšší priority):

1. **Repository-specific** (`<repo>/.git/config`) — per-project. Override pro jedno konkrétní repo.
2. **User-level** (`~/.gitconfig`) — default pro tebe napříč všemi projekty. **Tady nastavujeme defaults.**
3. **System-wide** (`/etc/gitconfig`) — OS-level. Obvykle nesaháme.

**Důležité**: vyšší priorita přebíjí nižší. Pokud global říká `email = firemní` a repo-local říká `email = osobní`, **v tom repu vyhraje osobní**.

Příkazy:
- `git config --global <klíč> <hodnota>` → zapíše do `~/.gitconfig`.
- `git config <klíč> <hodnota>` → zapíše do aktuálního repa (`.git/config`).
- `git config --global --list` → zobrazí všechny globální nastavení.

## Část 2: Identita — user.name a user.email

Identita se objevuje v **každém commitu** — `git log`, GitHub/GitLab UI, blame, audit trail.

```bash
git config --global user.name "Tvé Jméno"
git config --global user.email "tvuj@email.cz"
```

**Důležité**: `user.email` musí být **registrovaný na GitHubu/GitLabu**. Jinak tvé commity budou "unverified" a nepropojí se s tvým profilem.

### Firemní vs osobní email — per-repo override

**Problém**: `user.email` je jeden, ale identity máš dvě. Ať do globalu dáš cokoli, **všechny** commity na stroji ponesou tutéž adresu — firemní i v osobních projektech, nebo osobní i ve firemních. Ani jedno nechceš.

**Řešení 1** — ruční override v konkrétním repu:

```bash
# V osobním repu:
cd ~/personal/muj-projekt
git config user.email "personal@email.com"    # bez --global!
```

Tohle repo bude mít osobní email, zbytek firemní. Ale musíš to udělat **ručně** pro každé osobní repo.

**Řešení 2** — automatické per-folder pravidlo (elegantnější):

Global drží jednu identitu jako default, `includeIf` ji přepíše ve vyjmenovaných
složkách. Jsou dvě možnosti, kterou identitu udělat defaultní — **vyber podle
toho, co ti víc sedí**, obě jsou v praxi používané.

**Varianta A — firemní jako default:**

```ini
# ~/.gitconfig
[user]
    name = Tvé Jméno
    email = your.name@company.com        # default = firemní

[includeIf "gitdir/i:~/personal/"]       # osobní složka přepíše
    path = ~/.gitconfig-personal
```

**Varianta B — osobní jako default:**

```ini
# ~/.gitconfig
[user]
    name = Tvé Jméno
    email = tvuj.osobni@email.cz         # default = osobní

[includeIf "gitdir/i:~/dev/firma/"]      # firemní složky přepíší
    path = ~/.gitconfig-firma
```

Přepisující soubor obsahuje jen ten jeden klíč:

```ini
# ~/.gitconfig-personal (resp. ~/.gitconfig-firma)
[user]
    email = ta-druha@adresa.cz
```

### Čím se varianty liší

Ne pohodlím — v obou případech je po setupu nulová manuální práce. Liší se tím,
**co se stane, když pravidlo nesedne**: překlep v cestě, klon do nepokryté
složky, nová složka, kterou nikdo do configu nedopsal.

| Default | Když pravidlo nesedne | Následek |
|---|---|---|
| A firemní | osobní repo dostane firemní email | firemní adresa zůstane v public historii cizího projektu |
| B osobní | firemní repo dostane osobní email | commit se nespáruje s pracovním účtem |

První se opravuje přepisem historie, druhý jedním `git config user.email`.
Varianta A je zvyk u lidí, kteří na stroji dělají téměř jen firemní práci;
varianta B u těch, kdo mají osobních projektů srovnatelně. **Zeptej se kolegy,
kterou chce**, nepředpokládej.

Pokud osobní projekty na stroji vůbec nemáš, `includeIf` nepotřebuješ — stačí
global s firemní adresou.

### `gitdir/i:` versus `gitdir:`

Používej **`gitdir/i:`**, tedy case-insensitive variantu. Na macOS a Windows je
filesystem case-insensitive, takže `cd ~/dev` i `cd ~/Dev` fungují — ale git
porovnává pattern **textově**. Když má člověk složku `Dev` a v configu `~/dev`,
pravidlo se nikdy nechytne a **nikdo si toho nevšimne**: commity tiše dostanou
default identitu.

Na Linuxu je FS case-sensitive, tam je `gitdir:` přesnější a `/i` by teoreticky
chytilo i jinou složku, kdyby existovaly `~/dev` a `~/Dev` zvlášť. Krajní
případ; pro onboarding je `/i` bezpečnější volba.

## Část 3: Co je lineární vs nelineární historie (a proč na tom záleží)

Než nastavíme `pull.rebase` a `pull.ff`, musíš pochopit **proč**. Tohle je nejdůležitější koncept pro čistý git workflow.

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

### Proč chceme lineární

- **Čitelnost**: `git log --oneline` je čistý příběh projektu.
- **Bisect**: binární vyhledávání chyby ("`git bisect` — který commit to rozbil?") funguje přesně, nemusí přeskakovat merge commits.
- **Revert**: `git revert <hash>` pro libovolný commit je single command, žádný `-m 1` parent confusion.
- **Blame**: `git blame soubor.sql` ukazuje skutečného autora řádku, ne merge commit.
- **Code review**: PR diff na GitHubu ukazuje přesně co kolega změnil, bez merge noise.

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
- **Který email má být default** — a jestli budeš mít na stroji i druhou
  identitu. Pokud ano, vyber variantu podle Části 2 (firemní nebo osobní jako
  default); global dostane tu defaultní a druhá se doplní přes `includeIf`
  v Části 10. Adresa musí být registrovaná na účtu, ke kterému se mají commity
  párovat.

> **Než zapíšeš**: pokud `welcome` ohlásil spravované dotfiles (nebo
> `[ -L ~/.gitconfig ]` vrátí true), postupuj podle
> [managed-dotfiles.md](../welcome/references/managed-dotfiles.md) — zápis do
> cesty v home by upravil trackovaný soubor v config repu.

```bash
git config --global user.name "Tvé Jméno"
git config --global user.email "tvuj@email.cz"      # ta defaultni
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
user.email=tvuj@email.cz
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
    email = tvuj@email.cz
[pull]
    rebase = true
    ff = only
[init]
    defaultBranch = main
```

Jednoduchý INI formát. Můžeš **editovat přímo** (VS Code, nano), ale `git config --global` je bezpečnější (syntax validation).

### Per-repo identity (volitelné — pro pokročilé)

Pokud máš na stejném stroji **firemní i osobní projekty** a chceš **automaticky** přepínat email:

Vyber variantu podle Části 2 (firemní nebo osobní jako default) a přidej
odpovídající pravidlo. Příklad pro firemní default:

```ini
# ~/.gitconfig — přidej na konec:
[includeIf "gitdir/i:~/personal/"]
    path = ~/.gitconfig-personal
```

```bash
# Vytvoř ~/.gitconfig-personal:
cat > ~/.gitconfig-personal << 'EOF'
[user]
    email = tvuj.osobni@email.cz
EOF
```

Výsledek: vše v `~/dev/` → default identita. Vše v `~/personal/` → osobní email.
Nula manuální práce.

### Ověř, že pravidlo skutečně zabírá

`includeIf` je snadné napsat tak, že se nikdy nechytne (překlep v cestě, chybějící
lomítko na konci, case mismatch). Konfigurace přitom vypadá správně a git
nehlásí nic. **Netvrď, že to funguje, dokud to neuvidíš** — otestuj to
dočasnými repy:

```bash
for d in ~/dev ~/personal; do
    T="$d/_identity_test_$$"
    mkdir -p "$T" && git -C "$T" init -q \
        && printf '%-20s -> %s\n' "$d" "$(git -C "$T" config user.email)"
    rm -rf "$T"
done
```

Každá cesta musí vypsat email, který u ní očekáváš. Když obě vypíší tentýž,
pravidlo nesedí — zkontroluj cestu v `includeIf` (včetně koncového lomítka)
a jestli má být `gitdir/i:`.

## Část 11: Hotovo, co dál

Máš git konvence nastavené. Commity budou mít tvou identitu, `git pull` bude rebasovat (lineární historie), fast-forward pojistka chrání před accidental merge commits.

Pokračuj na **`/install-gh-glab`** — nainstalujeme GitHub CLI (`gh`) a GitLab CLI (`glab`) pro snadnější interakci s repa.
