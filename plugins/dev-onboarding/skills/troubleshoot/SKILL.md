---
name: troubleshoot
description: "Typické gotchy v Slevomat AI dev prostředí — SSH permission denied, git authentication issues, plugin install fails, Claude Code extension divný stav, WSL mount problémy, token expirace. Poskytuje diagnostiku a fix pro každý scénář. Auto-invoke při 'něco nefunguje', 'SSH nejde', 'plugin install selže', 'Claude se zasekl'."
---

# Troubleshoot — typické gotchy

Pokud tvůj onboarding nebo dev práce naráží na problém, **tady je seznam nejčastějších issues**. Projdi relevantní sekci, zkus fix, nebo řekni Claudovi detail problému a navede tě.

## SSH / Git access

### `Permission denied (publickey)` při `git clone` / `ssh -T`

**Příčiny**:
1. Public key nenahraný v GitLab / GitHub.
2. SSH klíč je jinde než kde SSH klient hledá.
3. Špatné permissions na `~/.ssh/` souborech.
4. ssh-agent nepouští klíč (pokud máš passphrase).

**Fix**:

```bash
# 1. Otestuj přímo s verbose flag
ssh -vT git@gitlab.com

# Hledej řádky:
# "Offering public key" — jaké klíče SSH nabízí
# "Server rejected" / "Permission denied" — server odmítl

# 2. Zkontroluj permissions (strict)
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519  # privátní
chmod 644 ~/.ssh/id_ed25519.pub  # veřejný
chmod 644 ~/.ssh/config

# 3. Ověř že public key je v GitLab/GitHub
cat ~/.ssh/id_ed25519.pub   # zkopíruj výstup
# GitLab: https://gitlab.com/-/user_settings/ssh_keys
# GitHub: https://github.com/settings/keys

# 4. ssh-agent pokud máš passphrase
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
ssh-add -l   # list klíčů v agent
```

### `Host key verification failed`

Server změnil SSH host key (legitimní rotace, nebo možný MitM).

**Fix**:

```bash
ssh-keygen -R gitlab.com
ssh-keygen -R github.com
# Pak znovu ssh -T git@... → odpoví yes na fingerprint
```

**Ověř fingerprint** před `yes` — [GitLab fingerprints](https://docs.gitlab.com/ee/user/gitlab_com/#ssh-host-keys-fingerprints), [GitHub fingerprints](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/githubs-ssh-key-fingerprints).

### `git push` → `fatal: unable to access 'https://...': ...`

Pokud repo používá HTTPS remote místo SSH:

```bash
# Zkontroluj
git remote -v
# origin  https://gitlab.com/slevomat/... (push)

# Přepni na SSH
git remote set-url origin git@gitlab.com:slevomat/....git

# Ověř
git remote -v
# origin  git@gitlab.com:slevomat/... (push)
```

## Claude Code extension

### Po loginu Claude Code sidebar prázdný / loop

**Příčina**: async tokens race condition po WSL remote loginu.

**Fix**: úplně zavři VS Code (`Ctrl+Shift+P → Quit`, včetně system tray), otevři znovu přes `code ~/dev/<project>` v Ubuntu terminálu. Při druhém spuštění jsou tokeny cachované.

### Claude Code se tě ptá na přihlášení ač jsi před chvílí login

**Příčina**: Claude Code extension má **dvě auth contexty** — sidebar panel a CLI/terminal. Sign-in flow přihlašuje jeden, ne oba.

**Fix**: Sign in v obou (uvidíš Sign In tlačítko → klik → stejný OAuth flow). Po druhém login obě fungují.

### "Skill not available in this environment"

**Příčina**: pokoušíš se spustit skill jako slash command (`/welcome`), ale skill nemá `disable-model-invocation: true` v frontmatter — je to **auto-invoke** skill, ne slash command.

**Fix**: 
- Napiš Claudovi **chat zprávu** co matche skill description: *"Proveď mě onboardingem"* → Claude auto-invoke `welcome` skill.
- Nebo použij **command palette**: `Ctrl+Shift+P → "Claude Code: Open in Side Bar"` pro sidebar, ne slash command.

### Plugin install selže s `Permission denied (publickey)`

**Příčina**: plugin repo je v privátním GitLabu, SSH key nenalezen.

**Fix**: dokonči `/setup-ssh` nejdřív, ověř `ssh -T git@gitlab.com`, pak opakuj install.

### `/plugin list` neznámý příkaz

**Příčina**: jsi v **Claude Desktop** (chat app), ne **Claude Code CLI / VS Code extension**. Claude Desktop nemá slash commands pro plugins — ty jsou admin-managed v Cowork UI.

**Fix**: pro per-user plugin install musíš být v Claude Code CLI / VS Code extension ve WSL.

## WSL gotchy

### Ubuntu skončí v `/mnt/c/...` místo `/home/<jmeno>/`

**Příčina**: spustil jsi `wsl` z PowerShellu v Windows adresáři, takže first shell dědí PWD.

**Fix**:
```bash
cd ~
pwd    # /home/<jmeno>
```

Nebo napříště otevírej Ubuntu přes **Start menu → Ubuntu ikona**, ne přes `wsl` v PowerShellu.

### `/home/<jmeno>/` neexistuje, ale `whoami` vrací tvé jméno

**Příčina**: VS Code se připojil k novému Ubuntu **před** interactive Unix setup, rozbil user state.

**Fix**: reinstall Ubuntu.

```powershell
# V Windows PowerShell
wsl --shutdown
wsl --unregister Ubuntu
wsl --install -d Ubuntu --no-launch
```

Pak Start menu → Ubuntu → interactive setup. **Zavři VS Code před tím** aby se znovu nenapojil dřív.

### `code .` v Ubuntu otevírá VS Code ale ne ve WSL mode

**Příčina**: Remote-WSL extension nenainstalovaná v VS Code.

**Fix**: Extensions panel (Ctrl+Shift+X) → hledej "Remote - WSL" (Microsoft) → Install. Restart VS Code, pak `code .` znovu.

### VS Code WSL indicator dole vlevo je jen `><`, žádný text

**Příčina**: nejsi v WSL mode.

**Fix**: klikni na `><` symbol → "Connect to WSL" z menu. Nebo zavři window a spusť `code .` přímo z Ubuntu terminálu.

## Git config

### `git commit` si stěžuje na missing user.name / user.email

**Fix**: dokonči `/setup-git` — nastaví `git config --global user.name` a `user.email`.

### `git pull` vytváří merge commity i když jsem nastavil pull.rebase

**Příčina**: někdo override s `git pull --no-rebase` nebo má repo-local override v `.git/config`.

**Fix**:
```bash
# Zkontroluj repo-local override
cat .git/config | grep -A 2 "\[pull\]"

# Pokud je tam rebase=false, smaž
git config --local --unset pull.rebase

# Nebo vynuť ff-only pojistku globally:
git config --global pull.ff only
```

## Obecné

### Claude mi nechce něco udělat — říká "nemám permission"

**Fix**: zkontroluj `.claude/settings.json` (nebo `~/.claude/settings.local.json`). Permissions blok může obsahovat `deny` rule co blokuje akci.

Nebo: při dialog **"Always allow X"** klikni → permission se uloží.

### Claude zapomněl něco co jsem mu řekl v minulé session

**Příčina**: conversation context je ephemeral (session-scoped), memory není perzistentní pokud Claude explicitně nezapsal.

**Fix**: řekni *"zapamatuj si X"* → Claude zapíše do `~/.claude/projects/.../memory/`. Napříč sessions pak drží.

Pro info co Claude o tobě ví: `cat ~/.claude/projects/.../memory/MEMORY.md`.

### Claude zachází s verzí souboru co už není aktuální

**Příčina**: Claude má v context starší obsah. Pokud ti změní něco a tys to pak přepsal, je confused.

**Fix**: řekni *"znovu přečti soubor X"* → Claude si stáhne aktuální obsah. Nebo restart session (zapomene cache).

## Kam eskalovat

Pokud něco tady není pokryté, nebo fix nefunguje:

1. Napiš mi (v chatu) přesný text chyby + co jsi dělal.
2. Pokud je to bug Claude Code: [github.com/anthropics/claude-code/issues](https://github.com/anthropics/claude-code/issues).
3. Slevomat-specific: napiš Andrému (`andre.heller@slevomat.cz`).

## Rychlý reset (nouzová možnost)

Pokud je Claude Code totálně zaseklý:

```bash
# Zavři VS Code úplně

# Vypni WSL
# (v Windows PowerShell)
wsl --shutdown

# Otevři znovu Ubuntu, pak VS Code přes code .
```

Pokud ani to nepomůže → reinstall Claude Code extension (Extensions panel → Uninstall → Install).
