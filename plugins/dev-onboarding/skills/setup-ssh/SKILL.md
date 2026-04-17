---
name: setup-ssh
description: "ONBOARDING krok — první SSH setup pro nového kolegu. Hloubkový tutoriál: co je SSH klíč, jak se liší od hesla / access tokenu / OAuth, kde klíče leží na disku, nejlepší praxe (per-service klíče s SSH config). Provede generací ed25519 klíčů pro GitLab + GitHub, upload public klíčů, test spojení, ssh-agent. Auto-invoke JEN při dev onboardingu: 'jsem nový kolega a nastavuji SSH', 'onboarding SSH', 'první SSH klíče pro GitHub/GitLab', 'welcome mě poslal na setup-ssh'. NEVOLAT při běžných dotazech o SSH (např. 'potřebuju další klíč pro server X', 'debug SSH connection') — ty řeší obecné znalosti nebo troubleshoot skill."
---

# SSH klíče — co to je a jak je správně nastavit

Tohle je jeden z nejdůležitějších kroků onboardingu. Bez SSH nemůžeš klonovat privátní repa, nemůžeš pushnout commity, nebudeš mít přístup k privátním projektům. **Důležitější je ale pochopit co děláš** — abys za 3 měsíce, když něco selže, věděl co se děje. Tak si to projdeme klidně a laicky.

## Část 1: Jak se vůbec přihlašujeme ke službám (4 způsoby)

Služby jako GitHub, GitLab, Claude.ai a další služby — všechny potřebují ověřit **kdo jsi**, než tě pustí dovnitř. Máme 4 hlavní způsoby:

### 1. Heslo (password)

Nejstarší. Zadáš username + heslo. Server si pamatuje hash tvého hesla a porovnává.

**Problém**: pokud heslo unikne (data breach, keylogger, lidská chyba), útočník má **plný přístup**. Proto moderní služby heslo používají **jen pro první přihlášení**, pak dávají něco bezpečnějšího.

### 2. SSH klíč

**Analogie**: SSH klíč je jako **fyzický klíč od domu**, ale digitální. Skládá se z dvou částí:

- **Privátní klíč** (`~/.ssh/ed25519_gitlab` nebo podobně) — **zůstává jen na tvém stroji**. Nikdy ho nikomu neukazuj. Jako klíč od sejfu ve tvé kapse.
- **Veřejný klíč** (`~/.ssh/ed25519_gitlab.pub`) — nahraješ na server (GitLab, GitHub). Je to jako **zámek** vyrobený specificky pro tvůj privátní klíč. Může být klidně veřejný — nejde zpětně spočítat.

Když se chceš přihlásit: server ti pošle náhodný "challenge" text, tvůj stroj ho podepíše privátním klíčem, server ověří podpis veřejným klíčem. Pokud sedí → pustí tě dovnitř. **Nikdo nemusel slyšet heslo.**

**Proč lepší**: pokud ti někdo ukradne `.bashrc` nebo nakoukne do mailu, **SSH klíč neobsahuje**. Ten je jen v `~/.ssh/`, chráněný `chmod 600` permissions. Kompromitace vyžaduje fyzický / root přístup na tvůj stroj.

**Kdy se používá**: `git clone git@gitlab.com:...`, `git push`, SSH do serveru, podpis commitů.

### 3. Access Token / Personal Access Token (PAT)

**Analogie**: access token je jako **razítko s omezenou platností a scope**. Vygeneruješ si ho na webu služby (GitHub Settings → Developer → Tokens), řekneš "platí 90 dní, smí jen číst repa, nesmí mazat".

Pak ho používáš **místo hesla** při API voláních nebo `git clone` přes HTTPS. Typicky v env variable (`GITHUB_TOKEN`, `GITLAB_TOKEN`) nebo v `.bashrc`.

**Proč lepší než heslo**: Když token unikne, útočník má jen **omezený scope** (např. read-only). Navíc token můžeš okamžitě revoke, aniž bys měnil heslo. Heslo revoke = změna všude kde je použito.

**Kdy se používá**: API access, CI/CD, automatizované skripty, `gh` / `glab` CLI login (pokud nepoužívá OAuth).

### 4. OAuth

**Analogie**: OAuth je flow kdy **aplikace tě nikdy nezná tvoje heslo**. Místo toho tě přesměruje do oficiálního login (GitHub, Google), tam se přihlásíš, služba ti dá token, aplikace pak používá ten token.

**Příklad**: přihlásíš se do Claude.ai v Claude Code extension. Claude Code tě pošle do browser na `claude.ai/oauth`. Tam se přihlásíš (heslo, nebo SSO přes firemní email, nebo hardware key). Claude.ai pak vrátí Claude Code access token. Claude Code nikdy neviděl tvoje heslo, a když firma změní tvoje heslo, funguje všechno dál (token zůstává platný).

**Proč lepší**: zero-knowledge for third-party apps. Plus revoke je centrální — v SSO vidíš seznam všech aplikací které jsi povolil, klik revoke odřízne jednu konkrétní.

**Kdy se používá**: Claude Code login, GitHub OAuth apps, Google login, firemní SSO.

### Shrnutí — kdy co

| Auth | Typický scénář | Úroveň security |
|---|---|---|
| Heslo | První přihlášení do služby | Nejslabší |
| SSH klíč | `git clone`, `git push`, SSH server | Vysoká |
| Access token (PAT) | API, CI/CD, CLI tools | Střední-vysoká |
| OAuth | App login přes SSO, Claude.ai, atd. | Nejvyšší (centralní revoke) |

**Pro tebe dnes**: řešíme SSH klíče (git clone privátní repa). Token a OAuth přijdou později v jiných skillech.

## Část 2: Kde SSH klíče leží (filesystem)

Všechny SSH věci jsou v **`~/.ssh/`** (`/home/<jmeno>/.ssh/`):

```
~/.ssh/
├── id_ed25519          ← PRIVÁTNÍ klíč (pro default SSH)
├── id_ed25519.pub      ← veřejný klíč
├── github              ← privátní klíč pro GitHub (per-service variant)
├── github.pub          ← veřejný pro GitHub
├── gitlab              ← privátní pro GitLab
├── gitlab.pub          ← veřejný pro GitLab
├── config              ← konfigurace "který klíč pro který server"
├── known_hosts         ← fingerprints serverů co jsi už viděl (anti-MitM)
└── authorized_keys     ← veřejné klíče co mají povolený login NA TENHLE stroj (nerelevantní pokud serveruješ nikomu)
```

**Permissions** musí být striktní, jinak SSH odmítne klíče použít:
- `~/.ssh/` → `chmod 700` (jen ty čteš, jen ty zapisuješ).
- Privátní klíče (bez `.pub`) → `chmod 600`.
- Veřejné klíče a config → `chmod 644`.

Standardní install to nastavuje samo. Pokud ne, `chmod -R go= ~/.ssh` opraví.

## Část 3: Jeden klíč pro vše vs. per-service klíče

Dvě školy, obě legitimní:

### Škola A: **Jeden klíč pro všechno** (doporučeno pro start)

```
~/.ssh/id_ed25519         (privátní, jediný)
~/.ssh/id_ed25519.pub     (veřejný)
```

Tenhle jediný klíč funguje pro GitHub, GitLab, SSH servery, všechno. Nahraješ stejný `id_ed25519.pub` všude.

**Výhody**:
- **Jednoduché** — nic se neplete.
- Default pattern — SSH ho najde bez extra konfigurace.
- Většina tutorialů na internetu předpokládá tohle.

**Nevýhody**:
- **Jeden kompromitovaný klíč = kompromitace všude**. Pokud z nějakého důvodu unikne, musíš odvolat na všech službách.
- Ne-granularni revoke — nemůžeš "přestat důvěřovat GitHubu" bez impactu na GitLab.

### Škola B: **Per-service klíče** (advanced, Andreho setup)

```
~/.ssh/github           (pro GitHub)
~/.ssh/github.pub
~/.ssh/gitlab           (pro GitLab)
~/.ssh/gitlab.pub
~/.ssh/config           (mapování: github.com → použij ~/.ssh/github)
```

Každá služba má vlastní klíč. SSH config říká "pro github.com použij tento klíč, pro gitlab.com tento".

**Výhody**:
- **Izolace** — jeden klíč kompromitovaný, ostatní v pořádku.
- **Granulární revoke** — můžu stahnout gitlab klíč a github nechat.
- **Organizace** — jasně vidíš co kam patří.

**Nevýhody**:
- **Víc setup** (generovat víckrát, nahrát víckrát, napsat config).
- SSH config se snadno rozbije (překlep = neautentizuje).

### Doporučení — per-service klíče (Škola B)

Doporučujeme **per-service klíče** jako standard — každá služba (GitHub, GitLab) má vlastní klíč + SSH config. Výhody izolace a granulárního revoke stojí za 2 minuty navíc při setupu.

Dál v tomto skillu **postupujeme podle Školy B** (per-service). Pokud chceš jednodušší variantu (jeden klíč pro vše), řekni a přepnu na Školu A.

## Část 4: Praktický setup — generace per-service klíčů

Zkontroluj nejdřív co už máš:

```bash
ls -la ~/.ssh/
```

Pokud vidíš `github` + `github.pub` + `gitlab` + `gitlab.pub` + `config`, máš klíče. Ověř a pokračuj na Část 5 (upload).

Pokud `.ssh/` je prázdné nebo obsahuje jen `known_hosts`, vygenerujeme oba klíče:

### Klíč pro GitHub

```bash
ssh-keygen -t ed25519 -C "your.name@company.com" -f ~/.ssh/github
```

### Klíč pro GitLab

```bash
ssh-keygen -t ed25519 -C "your.name@company.com" -f ~/.ssh/gitlab
```

Vysvětlení flagů:
- `-t ed25519` — algoritmus. **ed25519** je moderní, kratší, rychlejší, bezpečnější než starší RSA. Použij ho, není důvod pro RSA v 2026.
- `-C "email"` — komentář. Neovlivňuje kryptografii, ale GitHub/GitLab ti v UI ukazuje "klíč s popisem X" — dobré pro identifikaci.
- `-f ~/.ssh/github` (resp. `gitlab`) — pojmenujeme klíč podle služby. SSH default hledá `id_ed25519`, proto potřebujeme SSH config (krok níže) aby věděl který klíč pro který server.

**Passphrase** (volitelné heslo na klíč):
- **Prázdné (ENTER, ENTER)** = pohodlí. Klíč se dá použít bez dotazu. Pro většinu dev práce OK.
- **S passphrase** = větší bezpečnost. Pokud ti někdo ukradne privátní klíč, potřebuje ještě heslo. Ale musíš ho psát často, nebo použít ssh-agent.

Doporučení pro start: **prázdné** (simpler). Později můžeš přepnout na s passphrase (`ssh-keygen -p -f ~/.ssh/github`).

### SSH config — mapování klíč → služba

Bez configue by SSH nevěděl který klíč pro který server (protože klíče se nejmenují default `id_ed25519`). Vytvoř `~/.ssh/config`:

```bash
cat > ~/.ssh/config << 'EOF'
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/github
    AddKeysToAgent yes

Host gitlab.com
    HostName gitlab.com
    User git
    IdentityFile ~/.ssh/gitlab
    AddKeysToAgent yes
EOF
chmod 644 ~/.ssh/config
```

Vysvětlení:
- **`Host github.com`** — pattern match: když SSH vidí `git@github.com`, použije tuto sekci.
- **`IdentityFile ~/.ssh/github`** — který privátní klíč nabídnout serveru.
- **`AddKeysToAgent yes`** — automaticky přidá klíč do ssh-agent (nemusíš ručně `ssh-add`).
- **`User git`** — GitHub/GitLab vždy používají user `git` (ne tvoje jméno).

Po generaci by měl `~/.ssh/` vypadat takto:

```
~/.ssh/
├── github          (privátní klíč pro GitHub)
├── github.pub      (veřejný klíč pro GitHub)
├── gitlab          (privátní klíč pro GitLab)
├── gitlab.pub      (veřejný klíč pro GitLab)
├── config          (mapování klíč → server)
└── known_hosts     (fingerprints, vytvoří se při prvním ssh -T)
```

### Ověření permissions

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/github ~/.ssh/gitlab
chmod 644 ~/.ssh/github.pub ~/.ssh/gitlab.pub ~/.ssh/config
```

SSH je **striktní** na permissions — pokud privátní klíč má příliš volné permissions (readable pro ostatní), SSH ho odmítne použít.

## Část 5: Upload public klíčů na GitLab + GitHub

Musíš nahrát **každý .pub na odpovídající službu**. Privátní klíče (bez `.pub`) NIKDY neukazujeme nikomu.

### Upload do GitLab

```bash
cat ~/.ssh/gitlab.pub
```

Zkopíruj celý výstup (od `ssh-ed25519` po email). Pak:

1. Otevři: https://gitlab.com/-/user_settings/ssh_keys
2. **Key**: vlož public key z `cat ~/.ssh/gitlab.pub`.
3. **Title**: dej popisný — např. `WSL laptop (GitLab)`.
4. **Usage type**: `Authentication & Signing` (default).
5. **Expires at**: **POZOR — GitLab automaticky předvyplní expiraci** (typicky 1 rok). Pokud chceš klíč bez expirace (doporučeno pro dev stroje), **smaž datum z pole** — nech ho prázdné.
6. **Add key**.

### Upload do GitHub

```bash
cat ~/.ssh/github.pub
```

Zkopíruj výstup. Pak:

1. Otevři: https://github.com/settings/keys
2. **New SSH key**.
3. **Title**: `WSL laptop (GitHub)`.
4. **Key type**: `Authentication Key`.
5. **Key**: vlož public key z `cat ~/.ssh/github.pub`.
6. **Add SSH key**.

## Část 6: Test spojení

Ověř že klíč funguje:

```bash
ssh -T git@gitlab.com
ssh -T git@github.com
```

### První spojení — host key fingerprint

Při prvním spojení dostaneš:

```
The authenticity of host 'gitlab.com (172.65.251.78)' can't be established.
ED25519 key fingerprint is SHA256:eUXGGm1YGsMAS7vkcx6JOJdOGHPem5gQp4taiCfCLB8.
Are you sure you want to continue connecting (yes/no)?
```

Tohle je **anti-MitM** feature SSH. Server ti posílá svůj public key fingerprint. Ty ho porovnáš s oficiálně publikovaným (u GitLabu/GitHubu najdeš na webu v docs), pokud sedí → odpověz `yes`. Od teď tvůj stroj **pamatuje** tento server (v `~/.ssh/known_hosts`) a příště se ptát nebude.

Pokud sedí fingerprint (pro GitLab/GitHub je jich pár, všechny veřejně publikované), odpověz `yes`.

### Úspěšný výstup

GitLab:
```
Welcome to GitLab, @<tvoje_username>!
```

GitHub:
```
Hi <tvoje_username>! You've successfully authenticated, but GitHub does not provide shell access.
```

"Does not provide shell access" není chyba — GitHub říká "autentizoval jsi se, ale není co dělat interaktivně, jen git operace". OK.

### Pokud test selže

- `Permission denied (publickey)` → public key nenahraný, nebo nenaleznutý privátní klíč.
- `Connection refused` → firewall / proxy problém.
- `Host key verification failed` → fingerprint v `known_hosts` se liší od současného — buď server změnil klíč (legitimní, např. GitHub rotace), nebo MitM útok. Zeptej se na support.

Pokud něco selže, napiš mi přesný text chyby a vyřešíme to.

## Část 7: ssh-agent (bonus — pro pohodlí)

**Co to je**: service co drží dešifrované privátní klíče v paměti, takže nemusíš psát passphrase při každém SSH spojení.

Pokud máš klíč **bez passphrase** (jako teď), ssh-agent není nutný — SSH ho přečte přímo z disku.

Pokud jsi dal **passphrase**, bez ssh-agent bys psal heslo při každém `git clone`/`push`. S agent ho napíšeš jednou na startu session, pak ho SSH bere z paměti.

Ubuntu má ssh-agent ready — spustíš:

```bash
eval "$(ssh-agent -s)"       # spustí agent v aktuálním shellu
ssh-add ~/.ssh/id_ed25519     # přidá klíč (zeptá se na passphrase)
```

Automaticky na startu shellu: přidej do `~/.bashrc`:

```bash
# Start ssh-agent if not running
if [ -z "$SSH_AUTH_SOCK" ]; then
   eval "$(ssh-agent -s)" > /dev/null
fi
```

(Pro advanced: `keychain` utility řeší agent pro multiple shell sessions elegantněji. Pro teď skip.)

## Část 8: Hotovo, co dál

Pokud oba `ssh -T` testy prošly → **máš SSH funkční**. Můžeš klonovat privátní repa přes `git@gitlab.com:your-org/...` nebo `git@github.com:...`.

Pokračuj na **`/setup-git`** — git konfigurace dle týmových konvencí (pull.rebase, commit messages, atd.).

Nebo pokud chceš přepnout na per-service klíče (Školu B z části 3), řekni a provedu tě migrací.
