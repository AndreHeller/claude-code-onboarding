# Spravované dotfiles (symlinky do config repa)

Běžný pattern: config soubory nejsou v home, ale v git repu, a v home jsou jen
symlinky (`~/.gitconfig -> ~/config-git/gitconfig`). Typicky to zařizuje
`install.sh` v tom repu.

## Proč na tom záleží

Zápis skrz symlink projde — a upraví **trackovaný soubor v repu**. Nastavení
se aplikuje, ale zůstane jako necommitnutá změna, o které uživatel neví.
Platí pro `git config --global` i pro `>>` append do shell configu. Při příští
re-aplikaci configu z repa (nebo `git checkout`) se změna tiše vrátí zpátky.

## Postup, když je cíl symlink

1. Najdi repo, kterému cíl patří:

   ```bash
   git -C "$(dirname "$(readlink -f ~/.gitconfig)")" rev-parse --show-toplevel
   ```

2. Zapiš do trackovaného souboru v repu, ne do cesty v home.
3. Ukaž uživateli výsledek: `git -C <repo> diff`.
4. Zeptej se, jestli změnu commitnout. **Necommituj bez vyžádání**, a nikdy
   nepushuj.
5. Podívej se, jestli repo nemá `install.sh` nebo obdobný skript, který
   symlinky zakládá — nové soubory (např. další `gitconfig-*`) mohou patřit
   i do něj, jinak je na dalším stroji nikdo nenalinkuje.

## Na co si dát pozor u jednotlivých souborů

- **`~/.ssh/config`** — SSH kontroluje práva cílového souboru. Repo běžně drží
  `644`, což pro `config` stačí; privátní klíče do repa nepatří vůbec.
- **`~/.gitconfig`** — hodnoty piš do repa, ne přes `git config --global`.
  Ověřit efekt pak jde normálně: `git config --global --get <klíč>`.
- **Shell config** (`~/.bashrc`, `~/.zshrc`) — append `>>` míří do repa;
  po zápisu ověř, že export platí v novém interaktivním shellu.

## Poznámka k přenositelnosti

`readlink -f` je na Linuxu vždy, na macOS od 12.3. Na starším macOS použij
`python3 -c "import os;print(os.path.realpath(os.path.expanduser('~/.gitconfig')))"`.
