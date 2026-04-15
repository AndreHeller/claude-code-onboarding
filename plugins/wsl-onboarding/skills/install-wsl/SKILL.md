---
name: install-wsl
description: "Hloubkový tutoriál pro Windows nováčky — instalace WSL2 + Ubuntu (latest LTS), Unix user setup, VS Code Remote-WSL extension, Claude Code extension v WSL kontextu, přihlášení Slevomat Claude účtem. Spouštěno v Claude Desktop chatu skrz fetch markdown URL z https://github.com/AndreHeller/claude-welcome. Kolega copy-paste prompt z PROMPT.md, Claude tento skill stáhne a postupuje podle něj."
---

# Slevomat AI — Windows dev prostředí setup

Tenhle skill provede Windows nováčka kompletním setupem dev prostředí pro práci na Slevomat AI / BI projektech. Cíl: po dokončení má funkční WSL2 + Ubuntu + VS Code v WSL mode + Claude Code extension přihlášenou.

**Nezahrnuje** SSH, git config, Slevomat marketplace, žádný projekt-specific setup (dbt, Snowflake, …) — to řeší navazující plugin `dev-onboarding` (instaluje se v Claude Code extension po WSL setupu).

## Cíl po dokončení

1. WSL2 + Ubuntu (latest LTS, aktuálně 24.04 "Noble Numbat") na Windows.
2. Unix user account, pracovní adresář v Linux filesystemu (`/home/<user>/`), ne na Windows mount (`/mnt/c/`).
3. VS Code na Windows + Remote-WSL extension + Claude Code extension nainstalovaná **v WSL kontextu**.
4. Přihlášený do Claude Code extension Slevomat firemním emailem.
5. Připraven instalovat `dev-onboarding` plugin pro pokračování (SSH, git, Slevomat marketplace).

## Jak pracovat s tímto tutoriálem

- **Nepouštěj všechno najednou.** Krok za krokem, počkej na user reakci.
- **U každého kroku:** CO uděláme (odstavec), PROČ to děláme, přesný příkaz, v jakém prostředí ho má spouštět, co očekávat jako výsledek, preventivní warning o gotchách.
- **Česky.** Technické termíny (PowerShell, WSL, Ubuntu, VS Code) v originále.
- **Užívej "Ty" formu**, jsme v dev tutoriálu mezi kolegy.
- **Jsi v Claude Desktop** — nemůžeš sám spouštět příkazy v user PowerShellu / Ubuntu. Říkej user co spustit, on ti pošle výstup.

## Krok 1: Otevři PowerShell jako Administrator

**Win → napiš `powershell` → pravý klik na "Windows PowerShell" → Run as administrator.**
Nebo: Win+R → `powershell` → Ctrl+Shift+Enter.

Prompt musí ukazovat `PS C:\WINDOWS\system32>` (admin cesta). Pokud jen `PS C:\Users\<jméno>>`, nejsi admin — zavři a otevři správně.

**Všechny PowerShell příkazy v tomto tutoriálu spouštíš v tomto JEDNOM admin okně** (dokud neřeknu jinak — ke konci přejdeme do Ubuntu terminálu).

## Krok 2: Detekce aktuálního stavu

```powershell
winver                  # verze Windows — otevře GUI okno, řekni mi build (např. 22631)
wsl --status            # stav WSL infrastruktury
wsl --list --verbose    # nainstalované Linux distribuce (může být prázdné)
code --version          # VS Code verze (pokud existuje)
```

Pošli mi výstupy. Podle stavu navrhnu další krok:

- **Žádné WSL** → instalace WSL infrastruktury + Ubuntu.
- **WSL je, žádné Ubuntu** → jen install Ubuntu distribuce.
- **WSL + Ubuntu už jsou** → přeskočíme install, jdeme rovnou na verifikaci.
- **VS Code chybí** → install z [code.visualstudio.com](https://code.visualstudio.com/) (Windows User Installer).

Poznámka: **Git for Windows nepotřebujeme** — git je default v Ubuntu, pracuje uvnitř WSL.

## Krok 3: WSL + Ubuntu install

Pokud WSL nebo Ubuntu chybí:

**Vysvětli uživateli CO WSL je**: Linux ve Windows, bez VM, bez dual boot, bez rebootu při switchování. Pod kapotou hyperv-based lehká VM, ale uživatelsky transparentní.

**A PROČ ho potřebujeme**: Slevomat dbt / Python / dev stack je Linux-first. Bez WSL by ve Windows běžně narážel na path separator bugs (`\` vs `/`), file permission issues, line ending mismatch (CRLF vs LF), absent shebang interpreters. WSL všechny tyhle problémy odstraňuje.

**Prerequisites k zmínění**:
- BIOS virtualizace (Hyper-V, VT-x / AMD-V) — obvykle už enabled, ale občas potřeba zapnout.
- Windows 10 build 19041+ (release 2004) nebo Windows 11.

**Spusť install s `--no-launch` flagem** (klíčové — bez tohoto flagu Ubuntu hned po install otevře terminál v aktuálním PowerShell directory, což je v admin PS `C:\WINDOWS\system32` = `/mnt/c/WINDOWS/system32` ve WSL → user skončí v Windows mountu místo Linux home):

```powershell
wsl --install -d Ubuntu --no-launch
```

Vysvětli volbu **`Ubuntu`** (bez verze): meta-distribuce trackující latest LTS = aktuálně Ubuntu 24.04 "Noble Numbat". Má **Python 3.12 default** (netřeba deadsnakes PPA jako u 22.04).

Vysvětli **`--no-launch`**: zabrání auto-launch po install. Ubuntu si ručně otevřeme přes Start menu (krok 4) — tím skončíme v Linux home, ne v Windows mount.

**Restart**: Windows **může nebo nemusí** vyžadovat restart po `wsl --install`. Záleží jestli WSL infrastruktura už byla aktivní (dříve nainstalovaná, ne plně odebraná). Řiď se tím co Windows řekne — pokud "Restart required", restartuj.

## Krok 4: KRITICKÉ POŘADÍ — nezaměňovat!

**Tohle je nejčastější chyba noob kolegů**. Po dokončení `wsl --install`:

1. **Zavři PowerShell úplně** — ne minimize, ne nechat běžet, prostě zavřít okno. Kdyby zůstalo otevřené, mohl bys omylem spustit `wsl` z toho directory a skončit v `/mnt/c/...`.

2. **Zavři VS Code úplně** — včetně ikonky v system tray. Ctrl+Shift+P → "Quit" v VS Code. Pokud VS Code běží a má Remote-WSL extension, může auto-detekovat nové Ubuntu a pokusit se připojit **před** interactive Unix setup → rozbije setup (vznikne user záznam v `/etc/passwd`, ale home dir fyzicky neexistuje, VS Code se přihlásí jako root).

3. **Win+S → napiš `Ubuntu` → Enter**. Tím spustíš Ubuntu **přímo z Linux strany** = první launch začne v `/home/<jmeno>/` (= home), ne v Windows mountu. Současně se spustí **interactive Unix setup**.

**Pokud user pořadí zamění** (otevře Ubuntu přes `wsl` v PowerShellu z jiného adresáře, nebo nezavře VS Code):

- skončí v `/mnt/c/...` (Windows mount jako default directory)
- nebo bude přihlášený jako root (VS Code rozbil setup)
- nebo `whoami` ukáže user jméno, ale `/home/<user>/` fyzicky neexistuje (broken state)

**Fix při rozbitém stavu**: reinstall od nuly:

```powershell
wsl --shutdown
wsl --unregister Ubuntu
wsl --install -d Ubuntu --no-launch
```

A znovu krok 4 správně. Tohle není drahé — celý reinstall je 2-3 minuty. Lepší restartovat než hledat zaseklý state.

## Krok 5: Interactive Unix user setup

Při prvním launch z Start menu se Ubuntu zeptá:

```
Installing, this may take a few minutes...
Please create a default UNIX user account. The username does not need to match your Windows username.
For more information visit: https://aka.ms/wslusers
Enter new UNIX username:
```

**Username**: doporučuj lowercase, bez mezer, bez diakritiky. Např. `vaclav`, `michal`, `andre`. Nemusí matchovat Windows login.

**Heslo**: 
- Bude potřeba pro `sudo` (admin v Linux světě) — zapamatovat!
- Doporučuj **jiné** než Windows / firemní heslo (bezpečnost).
- Při psaní **nevidíš znaky ani hvězdičky** (Linux konvence) — prostě píšeš naslepo a Enter. Není to bug.

Po potvrzení vidíš prompt typu `vaclav@LAPTOP-AB12:~$`. Skvělé — jsi v Ubuntu jako Unix user.

## Krok 6: Ověř konzistentní stav

V Ubuntu terminálu:

```bash
whoami             # tvoje_jmeno (NE root!)
pwd                # /home/tvoje_jmeno
ls ~               # prázdné, ale BEZ chyby (adresář existuje)
lsb_release -a     # Release: 24.04, Codename: noble
python3 --version  # Python 3.12.x (rovnou, bez instalace)
git --version      # git 2.x.x (rovnou, bez instalace)
```

**Všechny checky musí pasovat.** Pokud:
- `whoami` ukazuje `root` → VS Code nebo PowerShell rozbil first-launch setup → reinstall (krok 4 s `wsl --unregister`).
- `pwd` ukazuje `/mnt/c/...` → user spustil Ubuntu přes `wsl` z PowerShellu, ne přes Start menu → zavři Ubuntu, spusť přes Win+S.
- `ls ~` vyhodí "No such file or directory" → broken state (user v passwd, home dir chybí) → reinstall.
- Cokoliv jiného neočekávaného → reinstall, je to nejjistější.

## Krok 7: Aktualizace balíků

Vysvětli **co `sudo` je** (admin v Linuxu, vyžaduje heslo z kroku 5) a **co dělá `apt`** (Ubuntu package manager — `update` refresh metadat, `upgrade` stáhne novější verze).

```bash
sudo apt update && sudo apt upgrade -y
```

Trvá 2-3 minuty. Pokud apt vypíše že nějaké balíky byly zachované (kept back), ignoruj — řešíme později.

## Krok 8: Vytvoření projektové složky

Zeptej se uživatele jaké jméno preferuje:

- **`~/dev/`** — konvence "developer", běžné v Linux komunitě.
- **`~/projects/`** — explicit, self-describing.

Default doporučení: **`~/dev/`** (kratší, ekosystém standard).

Po user volbě:

```bash
mkdir -p ~/dev
cd ~/dev
pwd                # /home/tvoje_jmeno/dev
```

**KRITICKÉ pravidlo**: všechny projekty klonuj a pracuj **zde**, NE v `/mnt/c/Users/...` (Windows mount). Důvody:
- **Výkon**: Linux filesystem je 10× rychlejší než `/mnt/c/` mount (git operace, dbt, Python imports).
- **EOL / permissions**: mixování CRLF (Windows) a LF (Linux) line endings rozbije Python, dbt parsing. File permissions jsou Windows-Linux nekompatibilní.
- **Bezpečnost**: malicious skript spuštěný v Ubuntu může poškodit Windows soubory, pokud je dostane přes mount. Pracovní disciplína v `~/dev/` minimalizuje cross-context exposure.

## Krok 9: První test projekt — "playground"

Vytvoř sandbox projekt pro test že vše funguje:

```bash
mkdir -p ~/dev/playground
cd ~/dev/playground
touch README.md
pwd                # /home/tvoje_jmeno/dev/playground
```

Vysvětli proč `touch README.md`: VS Code se trochu lépe chová s ne-prázdným adresářem, plus naznačuje budoucí zvyk (každý Slevomat repo by měl mít README).

## Krok 10: Otevři VS Code v projektu

V Ubuntu terminálu:

```bash
code .
```

**Pokud je to první spuštění `code` z WSL**: VS Code Server se stáhne do WSL automaticky (~30 sekund, vidíš "Installing VS Code Server..."). Čekej.

Po otevření VS Code **ověř WSL mode**:

- **Dole vlevo** je quick-actions oblast s ikonou `><` (a textem). 
- **Aktuální VS Code (2025-2026) má tu ikonu šedivou nebo modrou**, ne zelenou jako starší verze. **Barva není indikátor** — řiď se **textem** vedle.
- Hledej text **`WSL: Ubuntu-24.04`** (nebo podobně, dle distra). Pokud tam je, jsi v WSL mode.
- Pokud vidíš jen `><` bez WSL textu, klikni na ten symbol → menu → "Connect to WSL".

Sekundární kontrola: **titlebar VS Code okna** ukazuje `playground [WSL: Ubuntu-24.04]`. Bez `[WSL: ...]` suffixu nejsi v WSL mode.

Pokud VS Code se otevře bez WSL spojení, řekni a vyřešíme.

## Krok 11: Claude Code extension v WSL kontextu

Ve VS Code (WSL mode):

1. Otevři **Extensions** panel: **Ctrl+Shift+X**.
2. Nahoře vidíš **dvě sekce**: **LOCAL - INSTALLED** a **WSL: UBUNTU-24.04 - INSTALLED**.
3. Hledej **"Claude Code"** (publisher: Anthropic).
4. **KRITICKÉ — instaluj v WSL sekci**:
   - Pokud už extension máš na **Local** straně (Windows): klikni zelené tlačítko **"Install in WSL: Ubuntu-24.04"**.
   - Pokud neměl nikde: klikni **Install**, pak (pokud VS Code automaticky nepřidá v WSL) ručně **"Install in WSL"**.

**Vysvětli PROČ WSL kontext**: Claude Code extension potřebuje přístup k Linux shellu (bash), Linux filesystemu (`~/dev/...`), `claude` CLI v Linux paths. To vše je v WSL serveru, ne na Windows klientu. Local-only instance by nemohla pracovat s Ubuntu soubory.

Po úspěšné instalaci uvidíš Claude Code v sekci **WSL: UBUNTU-24.04 - INSTALLED**.

## Krok 12: Otevři Claude Code panel a přihlas se

**Otevři Claude Code panel přes command palette** (konzistentnější než hledat ikonu v side baru):

```
Ctrl+Shift+P → "Claude Code: Open in Side Bar"
```

V Claude Code panelu vidíš seznam přihlašovacích metod:

- ✅ **Claude.ai subscription** ← TOHLE pro Slevomat (Team/Enterprise plan).
- ❌ Anthropic Console (pay-per-API-use).
- ❌ Amazon Bedrock (AWS Claude gateway).
- ❌ Google Vertex AI / Microsoft Foundry (jiné cloud providers).

**Klikni "Claude.ai subscription"**.

**WSL remote login flow** (důležité — liší se od native Windows/macOS):

1. Otevře se browser → přihlas se Slevomat firemním emailem.
2. Místo automatického callback URL zpět do VS Code: **browser ti zobrazí token string**. Zkopíruj ho.
3. VS Code mezitím otevřel **input pole čekající na token** (může být v command palette nebo jako popup notifikace).
4. **Paste token + Enter**.

Vysvětli proč copy-paste token (ne callback): browser běží na Windows hostu, Claude Code extension běží v Linux WSL serveru. Localhost callback URL mezi těmito kontexty nefunguje. Copy-paste je jediná cesta ve WSL remote.

**Async tokens gotcha**: Po prvním přihlášení může Claude Code panel vypadat divně (prázdný, loop, znovu Sign-in tlačítko). Tokeny se načítají asynchronně, race condition. **Fix: zavři VS Code a otevři znovu** přes `code ~/dev/playground` v Ubuntu terminálu. Při druhém spuštění jsou tokeny cachované, vše funguje.

## Krok 13: Smoke test

Po druhém spuštění VS Code (s funkčními tokeny):

1. Otevři Claude Code panel: **Ctrl+Shift+P → "Claude Code: Open in Side Bar"**.
2. V chat panelu napiš:

   > *"Vytvoř mi v tomto workspace soubor `hello.py` který vypíše `Hello from Slevomat!`. Pak mi ukaž jak ho spustit."*

3. **Permission dialog**: Claude se zeptá jestli smí vytvořit soubor. Klikni **Allow**. Tohle je první z mnoha permission dialogů — Claude nemá default přístup, vše se ptá. Bezpečnostní feature.

4. Claude vytvoří `/home/tvoje_jmeno/dev/playground/hello.py`.

5. Otevři terminál ve VS Code:
   - **Ctrl + backtick** (klávesa vedle "1", pod Escape) — **NE** `Ctrl+Shift+J` (Claude může toto nabídnout, ale není to default VS Code shortcut).
   - Alternativně: menu **View → Terminal**.
   - Otevře se WSL bash terminál (prompt `tvoje_jmeno@hostname:~/dev/playground$`).

6. Spusť:
   ```bash
   python3 hello.py
   ```
   
   Očekávaný výstup: `Hello from Slevomat!`

Pokud vidíš výstup → **Claude Code funguje v WSL mode**. 🎉

## Krok 14: Handoff na dev-onboarding plugin

Řekni uživateli:

> *"Základní WSL setup hotov. Máš:*
> - *WSL2 + Ubuntu 24.04 + Unix user ✓*
> - *VS Code + Remote-WSL + Claude Code extension v WSL ✓*
> - *Claude login + permission dialog vyzkoušený + smoke test prošel ✓*
>
> ***Co tě čeká dál:**
> *Přejdi do Claude Code extension v VS Code (NE zpátky sem do Claude Desktop). V Claude Code chat panelu napiš:*
>
> ```*
> *Jsem nový Slevomat AI kolega, právě jsem dokončil WSL setup. Nainstaluj mi prosím plugin `dev-onboarding@claude-welcome` z https://github.com/AndreHeller/claude-welcome a spusť skill `welcome` — ať mě provede SSH klíčem, git konfigurací, instalací GitHub/GitLab CLI, vysvětlením Claude konceptů (memory, CLAUDE.md, skills) a registrací privátního Slevomat marketplace.*
> ```*
>
> *Plugin tě dovede k bodu kdy budeš mít přístup k Slevomat org-wide pluginům (`bi`) a budeš si moct klonovat projekt na kterém chceš pracovat (dbt, atd.). Každý projekt má vlastní `CLAUDE.md` + lokální skills, které tě povedou v jeho specifikách.*
>
> ***Důležité — permission dialogs**: V Claude Code extension budeš vidět **Allow / Deny** dialogy při každé akci (čtení souboru, spuštění příkazu, edit). To je bezpečnostní feature, ne bug. V začátcích prostě **Allow** vše — jsme v izolovaném Linux WSL prostředí, low risk. Postupně se naučíš permission modes (auto-allow, trusted patterns) — naučí tě to dev-onboarding plugin."*

## Začni krokem 1.
