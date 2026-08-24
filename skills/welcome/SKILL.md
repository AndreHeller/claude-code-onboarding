---
name: welcome
description: "Entry point dev onboardingu — router pro nového kolegu. Detekuje OS (Windows/WSL/Linux/macOS) a stav dev prostředí (SSH, git, gh, glab, firemní marketplace), ukáže dashboard ✓/✗ a nasměruje na relevantní další skill. Auto-invoke když uživatel napíše 'jsem nový', 'proveď mě onboardingem', 'nastav mi dev prostředí', 'pokračuj onboardingem', 'co dál po WSL', nebo zadá '/welcome'. První krok co Claude v dev-onboarding pluginu udělá."
---

# Vítej v dev onboardingu 🚀

Jsem průvodce nastavením tvého dev prostředí. Povedu tě krok po kroku — vysvětlím **co** a **proč**, ne jen "spusť tohle".

## Krok 0: Které hostingy tým používá

**Zeptej se dřív, než pustíš detekci** (jedna otázka, ne domněnka):

> *"Kde žijí repa, na kterých budeš pracovat — GitHub, GitLab, nebo oba?"*

GitLab **není** default. Když ho tým nepoužívá, vynech GitLab řádky z detekce
i z dashboardu úplně — jinak kolega uvidí `✗ GitLab SSH` a `✗ glab` a bude
řešit neexistující problém. V `install-gh-glab` pak přeskoč sekci `glab`.

Když odpověď neznáš (kolega si není jistý), testuj obojí, ale GitLab výsledek
označ jako `— GitLab: nepoužíváš? přeskoč`, ne jako `✗`.

## Krok 1: Detekce prostředí

Nejdřív zjistím **kde jsi** a **co už máš**. Spustím tyhle příkazy (každý uvidíš v permission dialogu — schval je):

```bash
# 1. OS detection — deterministic
uname -s 2>/dev/null || echo "Windows"
echo "WSL_DISTRO_NAME=${WSL_DISTRO_NAME:-(none)}"
echo "shell=$SHELL"

# 2. SSH — funkční test (NE existenční — každý má klíče pojmenované jinak)
# Rozlisuj PRICINU selhani — "nefunguje" muze byt klic, sit, nebo known_hosts
ssh_check() {   # $1 = host, $2 = popis
  OUT=$(ssh -T -o ConnectTimeout=5 -o StrictHostKeyChecking=accept-new "git@$1" 2>&1)
  case "$OUT" in
    *"successfully authenticated"*|*"Welcome to GitLab"*)
      echo "✓ $2 SSH OK" ;;
    *"Permission denied"*)
      echo "✗ $2 SSH — klíč chybí nebo není nahraný na účtu" ;;
    *"Could not resolve"*|*"timed out"*|*"Connection refused"*)
      echo "✗ $2 SSH — síť nebo proxy, ne klíč" ;;
    *"Host key verification failed"*)
      echo "✗ $2 SSH — known_hosts nesouhlasí (rotace klíče serveru, nebo MitM)" ;;
    *)
      echo "✗ $2 SSH — neznámý stav: $OUT" ;;
  esac
}
ssh_check github.com GitHub
# GitLab jen pri odpovedi z Kroku 0 — jinak tento radek vubec nespoustej
ssh_check gitlab.com GitLab

# 2b. SSH KVALITA — per-service klice, nebo jeden univerzalni?
if [ -f ~/.ssh/config ] && grep -qE '^[[:space:]]*Host[[:space:]]+github\.com' ~/.ssh/config \
   && grep -qE '^[[:space:]]*IdentityFile' ~/.ssh/config; then
  echo "✓ SSH per-service klíče (~/.ssh/config s IdentityFile)"
else
  echo "~ SSH funguje, ale bez per-service klíčů — jeden klíč pro všechny služby"
fi

# 3. Git config
git config --get user.email 2>/dev/null && echo "✓ git user.email set" || echo "✗ git user.email"
git config --get pull.rebase 2>/dev/null | grep -q true && echo "✓ pull.rebase=true" || echo "✗ pull.rebase"

# 3b. Git KVALITA — pojistky a rozdelena identita
[ "$(git config --global --get pull.ff)" = "only" ] \
  && echo "✓ pull.ff=only" || echo "~ pull.ff nenastaveno — chybí pojistka proti merge commitu"
[ "$(git config --global --get init.defaultBranch)" = "main" ] \
  && echo "✓ init.defaultBranch=main" || echo "~ init.defaultBranch není main"
git config --global --get-regexp '^includeif' >/dev/null 2>&1 \
  && echo "✓ identita rozdělená přes includeIf" \
  || echo "~ jedna identita pro všechna repa (řeš, jen když máš i osobní projekty)"

# 4. GitHub + GitLab CLI
gh auth status 2>&1 | grep -q "Logged in" && echo "✓ gh OK" || echo "✗ gh"

# 4b. gh KVALITA — umi git pouzit token od gh pro https URL?
if gh auth status 2>&1 | grep -q "Logged in"; then
  git config --global --get-regexp 'credential\..*github.*helper' >/dev/null 2>&1 \
    && echo "✓ gh credential helper pro https" \
    || echo "~ gh přihlášené, ale git neumí https — chybí gh auth setup-git"
fi
# glab taktez jen pri GitLabu
glab auth status 2>&1 | grep -q "Logged in" && echo "✓ glab OK" || echo "✗ glab"

# 5. Claude Code CLI binárka v terminálu (≠ VS Code extension!)
claude --version 2>/dev/null | head -1 && echo "✓ claude CLI OK" || echo "✗ claude CLI missing"

# 6. Spravovane dotfiles — symlinky do config repa (dulezite pred jakymkoli zapisem)
DOTFILES_MANAGED=0
for f in ~/.gitconfig ~/.gitconfig-* ~/.ssh/config ~/.bashrc ~/.zshrc; do
  [ -L "$f" ] && { echo "SYMLINK  $f -> $(readlink "$f")"; DOTFILES_MANAGED=1; }
done
[ "$DOTFILES_MANAGED" = 1 ] || echo "✓ žádné spravované dotfiles (zapisuje se přímo)"

# 7. Claude plugin marketplace (funguje jen pokud je CLI nainstalované)
claude plugin marketplace list 2>/dev/null | head -5 || echo "(žádné custom marketplaces, nebo CLI chybí)"
```

### Když detekce ohlásí `SYMLINK`

Config soubory jsou spravované z git repa a v home jsou jen odkazy. Uveď to
v dashboardu jako samostatný řádek (`Spravované dotfiles: ~/.gitconfig →
~/config-git/`) a **předej tuto informaci do všech dalších kroků** — zápis do
cesty v home by upravil trackovaný soubor v tom repu. Postup viz
[managed-dotfiles.md](references/managed-dotfiles.md).

## Krok 2: Dashboard — tři stavy, ne dva

Detekce rozlišuje **tři** stavy, ne jen „má / nemá":

| Stav | Význam | Co s tím |
|---|---|---|
| `✗` | chybí | plná cesta příslušným skillem |
| `~` | funguje, ale jinak, než doporučujeme | viz souhrnný dotaz níže |
| `✓` | odpovídá doporučení | přeskočit |

**Proč to rozlišení existuje:** zkušený kolega má SSH i git nastavené, takže
by při binárním `✓/✗` prošel celým onboardingem bez jediné zastávky — a nikdy
by se nedozvěděl o per-service klíčích, `pull.ff` pojistce ani o rozdělené
identitě. Stav `~` je přesně ten případ.

**`~` není chyba.** Kolega může mít vlastní setup z dobrého důvodu (config
repo, firemní politika, zvyk z jiné práce). Dashboard to popisuje neutrálně:
co má a co by doporučené nastavení přidalo — ne „máš to špatně".

### Souhrnný dotaz na konci

Po vypsání dashboardu, **jen pokud je aspoň jeden řádek `~`**, se zeptej
**jednou za všechny**:

> *"Tyhle věci máš nastavené jinak, než doporučujeme: <výčet>. Chceš je projít,
> nebo je necháme být a půjdeme dál?"*

Neptej se u každého bodu zvlášť a nevracej se k tomu podruhé. Když kolega
odmítne, pokračuj a víc to nezmiňuj. Když souhlasí, projdi jen dotčené skilly
— každý má sekci **„Už to máš, ale jinak"** s tím, co doporučení přidává.

## Krok 3: Routing — kam dál

Podle výstupu detekce rozhodnu, kterou cestou tě povedu.

### Scénář A: Windows **bez** WSL
Pokud `uname -s` vrátilo `Windows` (nebo příkaz selhal) a `WSL_DISTRO_NAME` je prázdné → jsi v PowerShell/CMD. **Nepokračujeme tudy** — nejdřív potřebuješ WSL.

→ Pokračuj skillem **`install-wsl`** (řekni "nainstaluj WSL" nebo "chci WSL").

> **Důležité**: Ostatní kroky (SSH, git, gh/glab) **musíš** dělat **uvnitř WSL Ubuntu**, ne ve Windows PowerShellu. Jinak nainstaluješ věci na špatné místo a v Claude Code (který běží v WSL) je neuvidíš.

### Scénář B: WSL / Linux / macOS
Pokud `uname -s` vrátilo `Linux` nebo `Darwin` (macOS), jedeme dál. Podle toho co v dashboardu chybí:

| Chybí | Další skill |
|---|---|
| ✗ SSH (GitHub, případně GitLab) | `setup-ssh` |
| `~` cokoli (a kolega souhlasil s projitím) | příslušný skill, sekce „Už to máš, ale jinak" |
| ✗ git user.email / pull.rebase | `setup-git` |
| ✗ gh (`glab` jen při GitLabu) | `install-gh-glab` |
| nikdy jsi nepoužil Claude Code | `claude-concepts` |
| ✗ claude CLI missing | `install-claude-cli` (prerekvizita pro marketplace) |
| patříš do firmy s vlastním marketplace | `install-marketplace` |
| vše ✓ | `next-steps` (klonuj projekt) |

### Aktivní otázka na firemní marketplace

Po zobrazení dashboardu se **aktivně zeptej uživatele** (nenech ho hádat, co je opt-in):

> *"Patříš do firmy, která má vlastní Claude plugin marketplace? Například Slevomat má privátní GitHub marketplace s `bi` pluginem (git-workflow konvence, naming patterns, audit skills). Pokud ano, řekni mi název firmy — nainstalujeme ti org-wide pluginy. Pokud ne nebo nevíš, přeskočíme rovnou na klonování projektu."*

- **Odpověď = "ano, <firma>"** → pokud detekce ukázala `✗ claude CLI missing`, nejdřív `install-claude-cli`, pak `install-marketplace`.
- **Odpověď = "ne / nevím / přeskoč"** → rovnou na `next-steps`.

### Scénář C: Všechno ✓
Výborně, máš hotovo. Pokračuj rovnou na **`next-steps`** — klonování konkrétního projektu.

## Doporučený sled pro úplného nováčka

Pokud je detekce "všude ✗", jedeme v pořadí:

1. **`install-wsl`** — jen pro Windows uživatele (Cowork Desktop). Přeskoč pokud jsi už v WSL / Linux / macOS.
2. **`setup-ssh`** — SSH klíče pro GitHub + GitLab. Vysvětlím co SSH klíč je, jak se liší od hesla / tokenu / OAuth.
3. **`setup-git`** — user.name, user.email, `pull.rebase=true`, `pull.ff=only`. Vysvětlím **proč** (lineární historie, fast-forward merge).
4. **`install-gh-glab`** — GitHub CLI (`gh`) + GitLab CLI (`glab`), auth login přes browser.
5. **`claude-concepts`** — **DŮLEŽITÉ**: rozdíl memory vs CLAUDE.md vs skills vs hooks. Kde co leží, co modifikovat ručně.
6. **`install-claude-cli`** — **prerekvizita pro marketplace**: instalace `claude` CLI binárky v terminálu (VS Code extension ji nevystavuje). Přeskoč pokud `claude --version` v detekci prošlo.
7. **`install-marketplace`** — **adaptivní**: registrace firemního privátního marketplace. Welcome router se aktivně ptá na firmu, nenechává to na iniciativě uživatele.
8. **`next-steps`** — klonuj konkrétní projekt (dbt, Streamlit, atd.) a přepni se na něj.

## Troubleshooting kdykoli později

Cokoliv se rozbije (`git push` nejede, SSH přestane fungovat, plugin marketplace odpadl) — řekni "něco mi nefunguje" a spustím skill **`troubleshoot`**. Pokrývá typické gotchy: SSH permission denied, token expirace, WSL mount problémy, Claude Code divný stav.

## Pokud jsi advanced uživatel

Dashboard ti řekne co ti chybí. Můžeš:
- **Přeskakovat** kroky, které už máš (detekce to vidí).
- **Začít rovnou** na `claude-concepts` (pro Claude novice) nebo `install-marketplace` (pokud jen chceš firemní plugin).
- **Skočit** na `next-steps`, pokud je všechno ✓.

## Začneme

Nejdřív se zeptám na hostingy (Krok 0), pak spustím detekci. Schval permission dialogy — každý bash příkaz uvidíš před spuštěním. Nebo prostě napiš *"jsem úplný nováček, nemám nic"* a začnu od `install-wsl` (Windows) nebo `setup-ssh` (jinde).
