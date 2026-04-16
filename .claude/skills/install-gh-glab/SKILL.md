---
name: install-gh-glab
description: "Instalace GitHub CLI (gh) a GitLab CLI (glab) v Ubuntu WSL. Oba jsou nutné protože Slevomat má repa na GitLabu (hlavní) i GitHubu (veřejné tooling). Vysvětluje k čemu slouží, jak se přihlásit, základní commands. Auto-invoke při 'nainstaluj gh glab', 'GitHub CLI', 'GitLab CLI'."
---

# GitHub CLI (gh) a GitLab CLI (glab)

Oba jsou command-line nástroje pro práci s Git hosting službami **z terminálu** bez nutnosti otevírat browser. Slevomat má:
- **Hlavní repa na GitLabu** (`gitlab.com/slevomat/*`) — potřebuješ `glab`.
- **Nějaká veřejná tooling repa na GitHubu** (např. `github.com/AndreHeller/claude-welcome` co sis právě klonoval) — potřebuješ `gh`.

Proto nainstalujeme oboje.

## Co tyto CLI umí

- **Issues**: `gh issue list`, `glab issue create` bez otevření browseru.
- **PRs / MRs**: `gh pr create --title "..." --body "..."`, `glab mr view`.
- **CI/CD status**: `gh run watch` sleduje CI v realtime.
- **API access**: `gh api repos/owner/repo/commits`, `glab api projects/123/pipelines`.
- **Auth**: `gh auth login`, `glab auth login` nastaví credentials (OAuth nebo token).
- **Repo operations**: `gh repo clone`, `gh repo create`, `glab repo view`.

Pro dev flow zbytečně ušetří spoustu alt-tab mezi IDE a browserem.

## Instalace v Ubuntu

### gh (GitHub CLI)

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

### glab (GitLab CLI)

Dvě cesty:

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

### glab auth login

```bash
glab auth login
```

Podobný flow:
1. **GitLab.com** (ne self-hosted).
2. **HTTPS** nebo **SSH** — **SSH**.
3. Přes browser → OAuth → uloží token.

Ověř:
```bash
glab auth status
```

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

Máš `gh` i `glab` funkční. Pokračuj na **`/claude-concepts`** — teď je důležité pochopit **Claude architekturu** (memory, CLAUDE.md, skills), než začneš pracovat na projektech. Bez toho budeš v Claude flow tápat.
