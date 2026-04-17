# Onboarding prompt pro Claude Desktop (fallback cesta)

> **Primární cesta** je firemní Claude Team plugin marketplace — admin tam publikuje `dev-onboarding` plugin a nový kolega ho má v Cowork automaticky (Installed by default) nebo self-service. Tenhle prompt je **fallback** pro kolegy, kteří Team plan / plugin marketplace nemají, nebo chtějí rozjet ručně.

Copy-paste do **Claude Desktop chatu** (z [claude.ai/download](https://claude.ai/download)).

Claude si stáhne WSL install tutoriál z tohoto repa a provede tě krok za krokem.

---

```
Ahoj, potřebuju nainstalovat WSL2 + Ubuntu na Windows, VS Code s Remote-WSL extension, a Claude Code extension v WSL kontextu.

Stáhni prosím obsah tohoto souboru z GitHubu:

https://github.com/AndreHeller/claude-code-onboarding/blob/main/plugins/dev-onboarding/skills/install-wsl/SKILL.md

Pokud k té URL nemáš přístup, řekni mi to a já si URL otevřu sám v browseru a obsah ti zkopíruju sem do chatu jako další zprávu.

Pak postupuj přesně podle toho obsahu — tutoriál stylem, krok po kroku, vysvětluj co a proč, počkej na moji reakci a teprve pak pokračuj. Jsem noob, nespěchej.

Cíl: po dokončení mám funkční WSL + Ubuntu + Unix user + VS Code v WSL mode + Claude Code extension přihlášenou. Pak mě navedeš jak naklonovat onboarding workspace a pokračovat tam — zbytek onboardingu (SSH, git, gh/glab, Claude koncepty, firemní marketplace) už poběží v Claude Code v WSL prostředí.
```

---

## Pokud Claude nemůže stáhnout URL

1. Otevři v browseru: https://github.com/AndreHeller/claude-code-onboarding/blob/main/plugins/dev-onboarding/skills/install-wsl/SKILL.md
2. Klikni vpravo nahoře **Raw**.
3. Ctrl+A → Ctrl+C.
4. Vrať se do Claude Desktop, vlož jako další zprávu.

---

## Po WSL instalaci

Kolega bude v Claude Code v WSL. Tam dál pokračuje instalací `dev-onboarding` pluginu:

- **Slevomat kolega**: admin má plugin jako *Installed by default* v Team marketplace → plugin je už aktivní, stačí napsat "proveď mě onboardingem".
- **Ostatní / open-source**: `claude plugin marketplace add https://github.com/AndreHeller/claude-code-onboarding.git` + `claude plugin install dev-onboarding`, pak "proveď mě onboardingem".
