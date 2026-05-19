---
name: plugin-building-blocks
description: "Vysvětluje stavební kameny Claude Code pluginu pro nového uživatele/vývojáře — z čeho se plugin uvnitř skládá (skill, hook, slash command, subagent, MCP server) a jak je sbalený do pluginu + marketplace. Polopatický průvodce s analogie z kanceláře a rozhodovací tabulka kdy co použít. Auto-invoke při 'co je to skill / hook / agent / plugin / marketplace', 'rozdíl mezi skill a agent', 'kdy použít hook', 'jak funguje plugin', 'z čeho se skládá Claude Code', 'kdy hook a kdy skill'. Komplement ke skillu claude-concepts (kde co leží na disku) — tenhle skill řeší architektonický pohled (z čeho je plugin uvnitř)."
---

# Stavební kameny Claude Code pluginu

Když si nainstaluješ plugin, není to jedna věc — je to **balík stavebních kamenů**, které spolu hrají. Tohle je mapa, abys věděl, **z čeho je plugin složený a kdy co kámen pomáhá**.

Komplement ke skillu `claude-concepts` — ten ti řekne **kde to leží na disku a co je persistent**, tenhle skill ti řekne **z čeho se plugin uvnitř skládá a kdy co použít**.

## Šest stavebních kamenů (s analogie z kanceláře)

### 🟢 Skill = příručka na poličce

- **Aktivuje se:** Když si do prompta řekneš "pomoz mi s commit messagem" — Claude si vzpomene "aha, mám příručku `git-workflow`" a otevře ji.
- **Cena:** Otevřená příručka leží na stole = zabírá místo (kontext window). Tlustá příručka zabere víc místa.
- **Použití:** Doménová znalost, konvence, návody. Cokoli, co má Claude **vědět** a **snažit se dodržet**.
- **Slabina:** Když pracuješ na něčem jiném (např. SQL refactor) a najednou si Claude potřebuje pushnout — **na příručku zapomene**, protože v hlavě má SQL. Skill je **soft** = advisory, ne mandatory.

**Kde žije:** `<plugin>/skills/<name>/SKILL.md` (+ volitelně `references/`, `scripts/`, `examples/` pro on-demand obsah).

### 🔴 Hook = portyrka, co stojí u dveří

- **Aktivuje se:** Vždy, když kdokoliv (i Claude sám) chce projít konkrétními "dveřmi" — tj. spustit konkrétní typ akce. Eventy: `PreToolUse` (před spuštěním nástroje), `PostToolUse`, `SessionStart`, `Stop`, atd.
- **Cena:** Nula kontextu — portyrka stojí mimo kancelář, ve své kabince (subprocess).
- **Použití:** Hard enforcement. Validace před akcí. Auto-akce na lifecycle event. Notifikace.
- **Síla:** Můžeš ji ignorovat? **Ne.** Portyrka může říct "stop, ověř X" — nebo dokonce "deny, projít nesmíš". Nemůže si Claude vybrat, jestli ji obejde.

**Kde žije:** `<plugin>/hooks/hooks.json` ukazuje na bash/python script v `<plugin>/scripts/<něco>.sh`. Hook script čte event JSON ze stdin a vrací JSON s rozhodnutím.

### 🟠 Slash command = tlačítko na stěně

- **Aktivuje se:** Když ho explicitně stiskneš příkazem `/<name>` (např. `/audit`, `/release`). **Nikdy sám**.
- **Cena:** Žádná, dokud ho nezmáčkneš.
- **Použití:** Workflows, které **musíš začít sám** — typicky operace s vedlejšími efekty (deploy, release, "začni práci na tomto tasku").

**Kde žije:** `<plugin>/commands/<name>.md` (markdown s instrukcemi pro Claude + volitelně `$ARGUMENTS` pro vstup uživatele).

### 🟡 Subagent = nezávislý pracovník v sousední místnosti

- **Aktivuje se:** Když hlavní Claude řekne "tohle dej kolegovi v sousední místnosti, ať to udělá" — buď automaticky podle popisu agenta, nebo přes `@mention`.
- **Cena:** Nula pro hlavní kontext — kolega má vlastní kancelář (vlastní kontext window). Vrátí jen "hotovo, výsledek je X".
- **Použití:** Tlustý research, dlouhé batche, čistění souborů, code review — věci, které by jinak zaplnily hlavní stůl. Subagent má vlastní model (sonnet/opus/haiku), vlastní omezení nástrojů, vlastní povolení.

**Kde žije:** `<plugin>/agents/<name>.md` s frontmatterem (`tools`, `model`, případně `mcpServers`, `hooks`).

### ⚪ MCP server = telefon do banky / Slacku / databáze

- **Aktivuje se:** Když Claude potřebuje data z venku — z externí služby (Slack, GitHub API, vlastní databáze, Notion, …).
- **Cena:** Tool jména (= "telefonní čísla") jsou viditelná v seznamu, vlastní schema (jak telefon použít) se loadne až při zavolání.
- **Použití:** Cokoli, co Claude **nemůže udělat sám lokálně** — read/write do external service, real-time dotaz, agregované API.

**Kde žije:** `<plugin>/.mcp.json` definuje které MCP servery se spustí spolu s pluginem. Server samotný je separátní proces (Node.js / Python).

### 📦 Plugin = krabice se sadou nástrojů

- **Co to je:** Plugin je **balík**, který v sobě obsahuje libovolnou kombinaci výše uvedených 5 stavebních kamenů — skills, hooks, agents, slash commands, MCP servery. Plus svůj manifest (`.claude-plugin/plugin.json` — jméno, verze, popis, autor).
- **Aktivace:** `claude plugin install <plugin>@<marketplace>` — stáhne celou krabici do `~/.claude/plugins/cache/` a zapne vše uvnitř.
- **Cena:** Záleží **co je v krabici**:
  - Skills uvnitř → jejich descriptions zaberou trochu kontextu (vždy)
  - Hooks → zero (žijí mimo kontext)
  - Agents/commands → zero, dokud je někdo nezavolá
  - MCP servery → tool jména viditelná, full schemas až při použití
- **Verzování:** SemVer v `plugin.json` (MAJOR/MINOR/PATCH). Vydání = bump version + tag.

**Analogie:** Stáhneš si aplikaci z App Store. Aplikace uvnitř obsahuje různé komponenty (tlačítka, services, notifikace). Nemusíš vědět, jak jednotlivé komponenty fungují — víš, že "tahle appka mi dělá X".

## A nad tím vším — Marketplace

**Marketplace = App Store, kde pluginy bydlí.**

- **Co to je:** Git repo s `.claude-plugin/marketplace.json`, kde je seznam pluginů a kam ukazují (branch / tag / commit). Někdy obsahuje pluginy přímo (`./plugins/<name>/`), někdy linkuje na cizí repa.
- **Příklady:**
  - **`claude-plugins-official`** — oficiální Anthropic marketplace ([github.com/anthropics/claude-plugins-official](https://github.com/anthropics/claude-plugins-official)). Obsahuje pluginy jako `skill-creator`, `plugin-dev`, `agent-sdk-dev`.
  - **Firemní marketplace** — typicky GitLab/GitHub repo s pluginama, které tým interně udržuje (např. konvence specifické pro vaše projekty).
  - **Osobní** — můžeš si vytvořit vlastní pro svoje experimentální pluginy.
- **Připojení:** `claude plugin marketplace add <git-url>` — Claude Code zaregistruje marketplace a může z něj instalovat pluginy.

**Vztah ke pluginu:** Marketplace je **distribuce**, plugin je **artefakt**. Když řekneš `claude plugin install bi@slevomat-ai`, znamená to "stáhni plugin `bi` z marketplaceu `slevomat-ai`".

## Kdy co použít — rozhodovací tabulka

| Když chceš… | Použij | Proč |
|---|---|---|
| …aby Claude *znal pravidlo* a snažil se ho dodržet | **Skill** (soft) | Auto-invoke na user prompt match, advisory |
| …aby nikdo nemohl pravidlo obejít, ani Claude sám | **Hook** (hard) | Deterministická validace, blokuje akci |
| …delegovat tlustou práci a šetřit hlavní kontext | **Subagent** | Vlastní kontext window, vrátí jen result |
| …pravidlo zapnout jen na vyžádání | **Slash command** | Explicit trigger, žádný auto |
| …volat externí systém | **MCP server** | API/DB/službu integrovat jako tool |
| …distribuovat všechno výše dohromady | **Plugin** | Balík + manifest + verzování |
| …pluginy publikovat ostatním | **Marketplace** | Git repo, který Claude umí listit/instalovat |

## Příklad ze života — bolest "skill se nepoužije"

Klasická bolest: máš skill `git-workflow` s pravidly pro commit/push, ale Claude si na něj v kontextově nabitém workflow **nevzpomene** a pushne bez ohlížení.

**Diagnóza:** Skill se aktivuje na **tvůj prompt**. Když Claude **sám** spouští `git push` mid-session (po jiném workflowu, např. SQL refactor), žádný prompt od tebe nepřišel → skill se ani nepokusí načíst.

**Lék:** Přidat k němu **hook** na `PreToolUse → Bash` matcher se scriptem, který deterministicky ověří git pravidla **při každém volání `git push`**. Hook se nemůže zapomenout, protože sedí mimo Claudeův kontext.

**Princip:** Skill + Hook = stratifikovaný hybrid. Skill = soft (Claude se snaží), Hook = hard (Claude nemůže obejít).

## Co dál

- Pokud chceš **postavit vlastní plugin** — Anthropic má oficiální `plugin-dev` plugin (7 expertních skills: hook-development, agent-development, skill-development, …). Instaluj `claude plugin install plugin-dev@claude-plugins-official`.
- Pokud chceš **vylepšit existující skill** — `claude plugin install skill-creator@claude-plugins-official` (eval framework, description optimization, benchmarking).
- Pokud chceš pochopit **kde co leží na disku a co je persistent** — viz [claude-concepts](../claude-concepts/SKILL.md) skill.
