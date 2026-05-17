# SS-SEvFI — Mod Conventions

## What the mod does

Starting Scenario: Socialist Europe vs Fascist Italy (SS:SEvFI) is an alpha-stage alternate-history starting scenario for Victoria 3. The default 1836 setup is replaced with a Cold-War-style bipolar Europe: a socialist bloc — either the **Union of Socialist States** (USSR-style, Russia-centred) or the **European Socialist Union (ESU)** (NGF-led) — facing a unified fascist Italy as its primary ideological counterweight.

The intended player experience is shaped by the `sevfi_content` game rule selected at game start:

- **create_ussr** — Russia anchors a classic Soviet federation with Constituent Republics across Poland, Ukraine, the Baltics, Central Asia.
- **create_esu** — North German Federation leads a pan-European socialist union (the default).
- **no_sevfi_content** — disables all mod content for a standard 1836 start.

Supporting systems include the **Constituent Republic** subject type (high-integration diplomatic pact), the **Ducist Regime** government type for fascist Italy (`gov_ducist_regime`, titles Duce / Ducessa), dynamic country naming (`X SSR` for Soviet-style subjects, `X Volkssektor` for German socialist subjects), strong decaying ideological-bias state modifiers that lock the starting alignment in, and two journal-entry tracks — **The Bolshevik Menace** (for fascist Italy and other right-wing European nations) and **Bringing the World Revolution** (for the socialist bloc). Two on-action chains drive ongoing pressure: `puppet_management` (overlord pressure on socialist puppets to align with council-republic laws — submit / resist / break free) and `modifier_management` (yearly maintenance of the bias modifiers).

Tone is serious-historical with Cold-War alt-history pulp underneath. The README's closing — "for feedback … please contact either me or your next Volkskommissar" — is the authoritative voice marker: dry, in-period bureaucratic, not edgy.

## Mod abbreviation

Internally the mod is **SEVFI** (Socialist Europe vs Fascist Italy — the `SS:` prefix is the repo's "Starting Scenario" series tag, not part of the mod abbreviation). Use `sevfi_` / `SEVFI` prefixes consistently in new files and SSM-style identifier slots as documented below.

## File and identifier prefixing

All script and localization files carry the `sevfi_` prefix without exception:

- `common/<category>/sevfi_*.txt`
- `events/sevfi_*.txt`
- `localization/english/sevfi_*_l_english.yml`
- `dlc_metadata/sevfi_dlc_metadata.txt`

This is a hard rule. Vanilla overrides must still live in `sevfi_`-prefixed files so the mod's footprint is greppable.

In-game definitions (journal entries, government types, subject types, diplomatic actions, dynamic country names, game rules) generally do **not** carry the `sevfi_` prefix when they belong to vanilla-style identifier families: `je_bolshevik_menace`, `je_bringing_the_world_revolution`, `gov_ducist_regime`, `dyn_c_constituent_soviet_republic`, `dyn_c_volkssektor`, `create_ussr`, `create_esu`, `no_sevfi_content`.

Exceptions where `sevfi_` *is* used in identifiers:

- Scripted triggers that scope geography by SEVFI-specific logic and could collide with vanilla wording: `sevfi_state_is_in_europe`, `sevfi_country_is_in_central_asia`, etc.
- On-action handlers and per-mod hook effects: `on_game_started_sevfi_init`, `on_monthly_pulse_country_puppet_management`, `on_yearly_pulse_state_modifier_management`. (Note: the `puppet_management` / `modifier_management` halves of those names are theme-based, not `sevfi_`-prefixed.)
- Game rule key: `sevfi_content` (and the `no_sevfi_content` option, which carries `sevfi` so a vanilla-style `no_content` doesn't get poached).
- DLC metadata key: `modsevfi`.

## Event namespaces

Default rule: use the **theme** as the namespace, no `sevfi_` prefix.

Examples currently in the codebase: `puppet_management`, `modifier_management`.

Add the `sevfi_` prefix to the namespace only when the bare name would collide with vanilla or with a different mod, or is too generic to stand alone.

## Localization key suffixes for event heads

Use these suffixes for the three head fields of every event localization entry:

| Field   | Suffix | Example                       |
| ------- | ------ | ----------------------------- |
| title   | `.tt`  | `puppet_management.2.tt`      |
| desc    | `.dd`  | `puppet_management.2.dd`      |
| flavor  | `.ff`  | `puppet_management.2.ff`      |

This is a project-wide convention across my mods. Do not standardize to vanilla `.t` / `.d` / `.f`.

Option keys keep single-letter suffixes (`.a`, `.b`, `.c`, ...).
Custom tooltip keys keep the `_tt` suffix.

## Variable naming

Mandatory suffixes for stored variables:

| Scope                                              | Suffix      |
| -------------------------------------------------- | ----------- |
| Scope variables (country, state, character, …)     | `_var`      |
| Global variables (`set_global_variable`)           | `_globvar`  |

The mod currently doesn't lean on stored variables — the ideological setup is anchored by state modifiers, on-actions and game rules rather than per-country flags. If new logic does need variables, apply the suffixes above; don't reach for flags.

## Custom tooltip suffix

Custom tooltips and `custom_tooltip = { text = … }` keys end in `_tt`. None currently in the codebase, but the convention is the same as the other mods in this project.

## Country-tag and scenario shorthands

`RUS` anchors the USSR variant; `NGF` / `SGF` anchor the ESU variant; `ITA` (after the `create_contrabolshevist_italy` afterburn) is the fascist Ducist Italy. State-spawning effects use the pattern `create_<flavour>_<country>` (`create_socialist_ngf`, `create_socialist_sgf`, `create_polish_soviet_republic`, `create_socialist_albania`, …) — keep that pattern when adding new constituent republics rather than inventing a fresh verb.

The bloc-creation entry points are `create_soviet_bloc` (USSR) and `create_pb_esu` (ESU power bloc). The `organise_southern_balkan` helper sets up Yugoslavia-adjacent territory and is called from `on_game_started_sevfi_init`.

## Dynamic country names

Two dynamic-name patterns drive the scenario flavour:

- `dyn_c_constituent_soviet_republic` — applied to subjects of `c:RUS` (or equivalent) under `law_council_republic`; adjective `root_adj_soviet`.
- `dyn_c_volkssektor` — applied to subjects of North or South German overlords under `law_council_republic`; adjective `root_adj`.

Both use `priority = 6` and `is_main_tag_only = yes`. Keep that priority unless the new dynamic name should outrank both — colliding at the same priority is asking for inconsistent labelling.

## Localization style

Localization is serious in-period prose, written from the bureaucratic and diplomatic register of Cold-War-style ideological pressure (see `puppet_management.2` — a "final reminder" letter from the overlord). Heavy use of `#italic`, `#bold` and `#!` for the in-letter / in-cable framing, and `[SCOPE.sState('overlord_capital').GetCityHubName]` token expansion so the same event text adapts to USSR-Moscow or ESU-Berlin runs.

Code comments and commit messages remain English-only (see project-level instruction).

## Content gating

All gameplay-relevant content must be guarded by `NOT = { has_game_rule = no_sevfi_content }` (the off-switch), and may additionally branch on `has_game_rule = create_ussr` vs `has_game_rule = create_esu` for content that belongs to only one of the two scenario flavours. Mirror the `on_game_started_sevfi_init` pattern when adding setup effects: guard the whole block once at the top, then call the per-country builders.

If new optional sub-features are added that should be independently toggled (e.g. a "no purges" or "extra agitator events" toggle), introduce a separate game rule rather than overloading `sevfi_content`.

## Asset pipeline

Binary assets (`*.dds`, `*.png`, `*.tga`, `*.ttf`, `*.otf`, `*.ogg`, `*.bank`) are tracked via Git LFS — see `.gitattributes`. Don't commit large binaries without LFS.

`.gitignore` excludes vanilla mirror folders kept locally for reference (`dlc/`, `soundtrack/`, `fonts/`, `map_data/`) plus scratch artifacts (`changes.txt`, `localizations_*.txt`, `scripts_*.txt`, `diff.txt`). Keep new throwaway tooling output matching those patterns.

## CHANGELOG / README

Per project instruction, CHANGELOG and README entries describe *what the player notices*, not implementation details. The README's framing — game rule, new countries, subject type, journal entries, puppet-management events — is the player-facing structure; keep new release entries within those buckets rather than reorganising by file or by mechanic. The "Alpha — not yet fully balanced" disclaimer stays until the mod settles.

The README also calls out incompatibility with other major European overhaul mods — keep that statement honest. If a change extends the European setup further, note the broadened scope in the README rather than silently shipping it.
