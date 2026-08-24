---
name: install-gh-glab
description: "ONBOARDING krok — první instalace GitHub CLI (gh) a GitLab CLI (glab) v Ubuntu/WSL/macOS. Vysvětluje k čemu slouží, jak se přihlásit přes browser (Token volba, ne SSH), jak propojit git s gh přes gh auth setup-git, základní commands. Auto-invoke JEN v onboarding kontextu: 'jsem nový kolega, nainstaluj gh glab', 'onboarding gh glab', 'welcome mě poslal na install-gh-glab'. NEVOLAT při pozdějších dotazech typu 'jak funguje gh pr create' — ty zvládne Claude z obecných znalostí."
---

# GitHub CLI (gh) a GitLab CLI (glab)

Command-line nástroje pro práci s Git hosting službami **z terminálu** bez nutnosti otevírat browser. `gh` budeš potřebovat téměř jistě (Slevomat i většina open-source žije na GitHubu); `glab` přidej, jen pokud tvůj tým používá i GitLab.

## Co tyto CLI umí

- **Issues**: `gh issue list`, `glab issue create` bez otevření browseru.
- **PRs / MRs**: `gh pr create --title "..." --body "..."`, `glab mr view`.
- **CI/CD status**: `gh run watch` sleduje CI v realtime.
- **API access**: `gh api repos/owner/repo/commits`, `glab api projects/123/pipelines`.
- **Auth**: `gh auth login`, `glab auth login` nastaví credentials (OAuth nebo token).
- **Repo operations**: `gh repo clone`, `gh repo create`, `glab repo view`.

Pro dev flow zbytečně ušetří spoustu alt-tab mezi IDE a browserem.

## Instalace

Nejdřív zjisti platformu — recepty se liší:

```bash
uname -s    # Darwin = macOS (Homebrew) · Linux = Ubuntu/WSL (apt)
```

### gh (GitHub CLI) — Ubuntu / WSL

GitHub doporučuje **oficiální apt repository**:

```bash
(type -p wget >/dev/null || (sudo apt update && sudo apt install wget -y)) \
  && sudo mkdir -p -m 755 /etc/apt/keyrings \
  && wget -qO- https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo tee /etc/apt/keyrings/githubcli-archive-keyring.gpg > /dev/null \
  && sudo chmod go+r /etc/apt/keyrings/githubcli-archive-keyring.gpg \
  && echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null \
  && sudo apt update \
  && sudo apt install gh -y
```

Ověření:
```bash
gh --version
```

### gh (GitHub CLI) — macOS

```bash
brew install gh
gh --version
```

Pokud `brew` chybí, nainstaluj nejdřív Homebrew podle [brew.sh](https://brew.sh).
Po instalaci ověř `brew --version` — když příkaz není k nalezení, chybí PATH:
Homebrew leží na **Apple Siliconu** v `/opt/homebrew`, na **Intelu**
v `/usr/local`, a instalátor na konci vypíše přesný `eval` řádek pro tvůj
shell config. To je nejčastější důvod, proč `brew` po instalaci „není".

### glab (GitLab CLI) — jen když tým GitLab používá

> **Přeskoč celou tuhle sekci**, pokud vaše repa žijí jen na GitHubu. `glab`
> bez GitLabu je jen nepoužívaná binárka a `glab auth login` nemá kam se
> přihlásit. Když si nejsi jistý, zeptej se team-leada, jestli tým má něco
> na GitLabu.

**macOS:**
```bash
brew install glab
glab --version
```

**Ubuntu / WSL** — dvě cesty:

**Přes apt** (pokud GitLab má pro tvé Ubuntu):
```bash
curl -s https://gitlab.com/gitlab-org/cli/-/raw/main/scripts/install.sh | sudo bash
```

**Přes oficiální apt repository** (doporučeno):
```bash
curl -fsSL https://packages.gitlab.com/gitlab-org/cli/script.deb.sh | sudo bash
sudo apt install glab -y
```

Ověření:
```bash
glab --version
```

## Přihlášení

### gh auth login

```bash
gh auth login
```

Interactive prompt:
1. **GitHub.com** (ne Enterprise).
2. **HTTPS** nebo **SSH** protokol — **SSH** pokud jsi dokončil `/setup-ssh` (autentizuješ přes SSH key místo tokenu).
3. **Login with a web browser** — doporučeno (OAuth flow, nejbezpečnější). Otevře URL, zkopíruje one-time code, přihlásíš se v prohlížeči.
4. GitHub ti dá access token, `gh` ho uloží do OS keyring nebo `~/.config/gh/hosts.yml`.

Ověř:
```bash
gh auth status
# github.com
#   ✓ Logged in to github.com account <ty> ...
```

### gh auth setup-git — nauč git používat token od gh

`gh` a `git` spolu po loginu **nemluví**: `gh` má OAuth token pro API, `git` umí
tvůj SSH klíč — ale pro **`https://` URL** git potřebuje credential helper,
který po čisté instalaci nemáš. Bez něj `git clone
https://github.com/<org>/<privátní-repo>` spadne na
`could not read Username for 'https://github.com'`, i když máš `gh` přihlášené
a SSH funkční.

```bash
gh auth setup-git
```

Zapíše credential helper do `~/.gitconfig` — git si od té chvíle pro
`https://github.com/...` bere token od `gh`. SSH cesty
(`git@github.com:...`) se to nijak nedotkne, jen přibude druhá funkční cesta.
Vratné smazáním `[credential "https://github.com"]` sekce z `~/.gitconfig`.

### glab auth login (jen při GitLabu)

```bash
glab auth login
```

Podobný flow:
1. **GitLab.com** (ne self-hosted).
2. **HTTPS** nebo **Token** — vyber **Token** (v glab CLI se SSH varianta jmenuje "Token", ne "SSH" — matoucí, ale je to to samé — autentizuje přes tvůj SSH klíč).
3. Přes browser → OAuth → uloží token.

Ověř:
```bash
glab auth status
```

## Už to máš, ale jinak (stav `~` z detekce)

Nejčastější případ: `gh` je přihlášené, ale chybí credential helper. Detekce to
hlásí jako `~ gh přihlášené, ale git neumí https`.

Projeví se to až ve chvíli, kdy něco klonuješ přes `https://` URL místo SSH —
typicky když zkopíruješ odkaz z prohlížeče nebo tě na něj pošle CI skript. `git`
se zeptá na uživatelské jméno a spadne na `could not read Username`, i když máš
`gh` přihlášené a SSH funkční. Řešení je jednorázové:

```bash
gh auth setup-git
```

Detail v sekci `gh auth setup-git` níže.

## Časté commands (krátký cheat sheet)

```bash
# Issues
gh issue list
gh issue create --title "Bug: ..." --body "Repro: ..."
glab issue list
glab issue create -t "Feature: ..." -d "Chceme aby..."

# Pull / Merge Requests
gh pr create --title "Fix login" --body "Co a proč" --reviewer <login>
gh pr view
gh pr checkout 123
glab mr create -t "Fix login" -d "Co a proč"
glab mr view

# Repo
gh repo clone owner/repo
gh repo view
glab repo view

# API
gh api repos/owner/repo/contents/README.md
glab api projects/<id>/pipelines

# CI
gh run list
gh run watch <id>
```

## Kde tokens/config žijí

- **gh**: `~/.config/gh/hosts.yml` + OS keyring pro token.
- **glab**: `~/.config/glab-cli/config.yml` + OS keyring.

**Nesahej ručně** — používej `gh auth login` / `glab auth login` pro správnou registraci.

## Hotovo, co dál

Máš `gh` (a případně i `glab`) funkční. Pokračuj na **`/claude-concepts`** — teď je důležité pochopit **Claude architekturu** (memory, CLAUDE.md, skills), než začneš pracovat na projektech. Bez toho budeš v Claude flow tápat.
