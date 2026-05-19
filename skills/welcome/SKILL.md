---
name: welcome
description: "Entry point dev onboardingu — router pro nového kolegu. Detekuje OS (Windows/WSL/Linux/macOS) a stav dev prostředí (SSH, git, gh, glab, firemní marketplace), ukáže dashboard ✓/✗ a nasměruje na relevantní další skill. Auto-invoke když uživatel napíše 'jsem nový', 'proveď mě onboardingem', 'nastav mi dev prostředí', 'pokračuj onboardingem', 'co dál po WSL', nebo zadá '/welcome'. První krok co Claude v dev-onboarding pluginu udělá."
---

# Vítej v dev onboardingu 🚀

Jsem průvodce nastavením tvého dev prostředí. Povedu tě krok po kroku — vysvětlím **co** a **proč**, ne jen "spusť tohle".

## Krok 1: Detekce prostředí

Nejdřív zjistím **kde jsi** a **co už máš**. Spustím tyhle příkazy (každý uvidíš v permission dialogu — schval je):

```bash
# 1. OS detection — deterministic
uname -s 2>/dev/null || echo "Windows"
echo "WSL_DISTRO_NAME=${WSL_DISTRO_NAME:-(none)}"
echo "shell=$SHELL"

# 2. SSH — funkční test (NE existenční — každý má klíče pojmenované jinak)
ssh -T -o ConnectTimeout=5 -o StrictHostKeyChecking=no git@github.com 2>&1 | grep -qE "successfully authenticated|Hi .+!" && echo "✓ GitHub SSH OK" || echo "✗ GitHub SSH"
ssh -T -o ConnectTimeout=5 -o StrictHostKeyChecking=no git@gitlab.com 2>&1 | grep -qE "Welcome to GitLab" && echo "✓ GitLab SSH OK" || echo "✗ GitLab SSH"

# 3. Git config
git config --get user.email 2>/dev/null && echo "✓ git user.email set" || echo "✗ git user.email"
git config --get pull.rebase 2>/dev/null | grep -q true && echo "✓ pull.rebase=true" || echo "✗ pull.rebase"

# 4. GitHub + GitLab CLI
gh auth status 2>&1 | grep -q "Logged in" && echo "✓ gh OK" || echo "✗ gh"
glab auth status 2>&1 | grep -q "Logged in" && echo "✓ glab OK" || echo "✗ glab"

# 5. Claude Code CLI binárka v terminálu (≠ VS Code extension!)
claude --version 2>/dev/null | head -1 && echo "✓ claude CLI OK" || echo "✗ claude CLI missing"

# 6. Claude plugin marketplace (funguje jen pokud je CLI nainstalované)
claude plugin marketplace list 2>/dev/null | head -5 || echo "(žádné custom marketplaces, nebo CLI chybí)"
```

## Krok 2: Routing — kam dál

Podle výstupu detekce rozhodnu, kterou cestou tě povedu.

### Scénář A: Windows **bez** WSL
Pokud `uname -s` vrátilo `Windows` (nebo příkaz selhal) a `WSL_DISTRO_NAME` je prázdné → jsi v PowerShell/CMD. **Nepokračujeme tudy** — nejdřív potřebuješ WSL.

→ Pokračuj skillem **`install-wsl`** (řekni "nainstaluj WSL" nebo "chci WSL").

> **Důležité**: Ostatní kroky (SSH, git, gh/glab) **musíš** dělat **uvnitř WSL Ubuntu**, ne ve Windows PowerShellu. Jinak nainstaluješ věci na špatné místo a v Claude Code (který běží v WSL) je neuvidíš.

### Scénář B: WSL / Linux / macOS
Pokud `uname -s` vrátilo `Linux` nebo `Darwin` (macOS), jedeme dál. Podle toho co v dashboardu chybí:

| Chybí | Další skill |
|---|---|
| ✗ GitHub/GitLab SSH | `setup-ssh` |
| ✗ git user.email / pull.rebase | `setup-git` |
| ✗ gh / glab | `install-gh-glab` |
| nikdy jsi nepoužil Claude Code | `claude-concepts` |
| ✗ claude CLI missing | `install-claude-cli` (prerekvizita pro marketplace) |
| patříš do firmy s vlastním marketplace | `install-marketplace` |
| vše ✓ | `next-steps` (klonuj projekt) |

### Aktivní otázka na firemní marketplace

Po zobrazení dashboardu se **aktivně zeptej uživatele** (nenech ho hádat, co je opt-in):

> *"Patříš do firmy, která má vlastní Claude plugin marketplace? Například Slevomat má privátní GitLab marketplace s `bi` pluginem (git-workflow konvence, naming patterns, audit skills). Pokud ano, řekni mi název firmy — nainstalujeme ti org-wide pluginy. Pokud ne nebo nevíš, přeskočíme rovnou na klonování projektu."*

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

Nejdřív spustím detekci. Schval permission dialogy — každý bash příkaz uvidíš před spuštěním. Nebo prostě napiš *"jsem úplný nováček, nemám nic"* a začnu od `install-wsl` (Windows) nebo `setup-ssh` (jinde).
