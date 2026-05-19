---
name: install-claude-cli
description: "ONBOARDING prerekvizita pro install-marketplace — instalace Claude Code CLI binárky v Ubuntu/Linux/macOS terminálu. Pozor: CLI binárka (`claude` command v terminálu) je NĚCO JINÉHO než VS Code Claude Code extension (chat panel). Oba čtou stejný `~/.claude/` config a sdílejí auth, ale instalují se odděleně. VS Code extension NEPOSKYTUJE `claude` CLI v terminálu. Auto-invoke: 'nemám claude CLI', 'claude: command not found', 'bash: claude: command not found', 'welcome mě poslal na install-claude-cli', 'nainstaluj claude CLI', 'install-marketplace mi říká že nemám claude', 'claude plugin command not found'."
---

# Claude Code CLI — instalace v terminálu

Tenhle skill nainstaluje `claude` CLI binárku v Ubuntu/Linux/macOS terminálu. Je to **prerekvizita pro `install-marketplace`** — plugin příkazy (`claude plugin marketplace add ...`, `claude plugin install ...`) fungují **jen v CLI binárce**, ne v chat panelu VS Code extension.

## Rozdíl: VS Code extension vs CLI binárka

Tohle plete skoro každého nováčka, tak si to nejdřív vyjasníme:

| | VS Code extension | CLI binárka |
|---|---|---|
| **Co to je** | Chat panel uvnitř VS Code | Program `claude` v bash terminálu |
| **Instalace** | VS Code → Extensions → Claude Code (publisher Anthropic) | `curl -fsSL https://claude.ai/install.sh \| bash` |
| **Config** | `~/.claude/` (sdílený) | `~/.claude/` (sdílený) |
| **Auth** | Login v panelu | Sdílí session s extension (netřeba znova) |
| **Plugin příkazy** | ❌ nemá slash commandy pro pluginy | ✅ `claude plugin marketplace add/install/list` |

**Klíčový bod**: i když máš VS Code extension přihlášenou a funkční, **CLI binárka v terminálu NENÍ automaticky nainstalovaná**. Musíš ji nainstalovat zvlášť tímto skillem.

## Krok 0: Detekce

Otevři bash terminál (Ctrl+backtick ve VS Code pro integrovaný, nebo Ubuntu ze Start menu) a spusť:

```bash
which claude
claude --version
```

- **Pokud `which claude` vypíše cestu** (např. `/home/tvuj_user/.local/bin/claude`) a `claude --version` vrátí verzi → **máš hotovo**. Vrať se do welcome routeru nebo na `install-marketplace`.
- **Pokud `command not found`** nebo `which` vrátí prázdno → pokračuj krokem 1.

## Krok 1: Instalace přes oficiální curl installer

Spusť v terminálu:

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

Co se stane:
- Stáhne `install.sh` z `claude.ai` a spustí ho v bashi.
- Installer stáhne aktuální CLI verzi (~50 MB) do `~/.local/share/claude/versions/<ver>/`.
- Vytvoří symlink `~/.local/bin/claude` → aktuální verze.
- Netřeba `sudo` — všechno v user home.

**⚠ Silent mode gotcha**: curl má `-fsS` flagy. `-s` = **silent** — žádný progress, žádný output do doby než bash installer doběhne. Pro tebe to vypadá, že terminál zamrzl. **Nemačkej Ctrl+C** — čekej 30-60 sekund (stahuje ~50 MB). Skončí textem typu `Claude Code installed successfully!` s verzí.

## Krok 2: PATH fix pro `~/.local/bin`

Installer binárku umístí do `~/.local/bin/claude`, ale **tento adresář nemusí být v PATH**. V čerstvém Ubuntu WSL instalaci typicky **není** (některé distra — Fedora, Arch — ho mají defaultně). Ověř:

```bash
claude --version
```

- **Pokud vrátí verzi** (např. `2.1.114 (Claude Code)`) → PATH OK, přeskoč na Krok 3.
- **Pokud `command not found`** → PATH neobsahuje `~/.local/bin`. Fix:

```bash
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
claude --version    # teď by mělo fungovat
```

Co to udělalo:
- `echo ... >> ~/.bashrc` — permanentně přidá PATH export do startup scriptu bashe (spustí se při každém novém terminálu).
- `source ~/.bashrc` — načte změnu do **aktuálního** shellu (jinak bys musel zavřít a otevřít terminál).

**Pokud používáš zsh** (macOS default, nebo custom setup): zaměň `~/.bashrc` za `~/.zshrc`. Jinak je postup stejný.

## Krok 3: Verifikace

```bash
claude --version
which claude
```

Očekávaný výstup:
```
2.x.y (Claude Code)
/home/tvuj_user/.local/bin/claude
```

**Bonus check — sdílená auth s VS Code extension**:

```bash
claude /status 2>&1 | head -10
```

Pokud vidíš svůj email a/nebo `Logged in` → auth sdílená s extension OK, netřeba se znova přihlašovat. Pokud `Not logged in`, sdílená session nezabrala — to je OK. Plugin příkazy (`claude plugin marketplace add ...`) fungují typicky i bez přihlášeného CLI, protože k Git marketplace repu se přistupuje přes SSH key, ne přes Claude auth.

## Hotovo, co dál

Máš `claude` CLI v terminálu. Pokračuj tam, kam tě welcome router poslal:

- **Jsi tady kvůli `install-marketplace`** (typický důvod) → napiš *"pokračuj install-marketplace"* nebo se vrať do welcome routeru.
- **Spustil jsi skill samostatně** → vrať se na `welcome` (*"pokračuj onboardingem"*) a dokonči zbytek dashboardu.
