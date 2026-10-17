---
name: osrs-plugin-review-panel
description: Run before any commit, code contribution or pull request to a RuneScape/OSRS/RuneLite-related repo, or when reviewing a RuneLite plugin idea: Jagex, Plugin Hub reviewer and Reddit player personas.
---

# OSRS Plugin Review Panel

This skill runs a RuneScape-related change past three personas, each with a different job:

| Persona | Job | Core question |
|---|---|---|
| **Jagex Integrity Reviewer** | Rules and fairness | Does this break the Third Party Client Guidelines or give an unfair advantage? |
| **RuneLite Plugin Hub Reviewer** | Maintainability and quality | Would I merge this, and can it be maintained without hurting the client? |
| **Average Reddit Player** | Value | Would I actually install this, and why? |

These are simulations to stress-test a change before it ships. The real authorities are the RuneLite Plugin Hub reviewers and Jagex. Say so once in the final summary, not in every section.

## When this runs

**Mandatory gate:** Run this panel before creating any commit, pushing code, or opening or updating a pull request in a RuneScape-related repository. Treat a repo as RuneScape-related if any of these hold:
- It is `runelite/plugin-hub`, `runelite/runelite`, or a fork of either
- It contains `runelite-plugin.properties`, depends on `net.runelite` (`client`, `api`), or has classes extending `net.runelite.client.plugins.Plugin`
- Its name, README or code refers to RuneScape, OSRS, Old School, RuneLite, HDOS or Jagex

Also run it when the user asks for feedback on a RuneLite plugin idea, even with no code yet.

**Gate behaviour:**
- Run the panel on the change *before* the commit or PR is made, not after.
- If the panel finds a **blocking** issue (a Jagex prohibition, or a reviewer blocker that would get the PR closed), do not create the commit or PR. Show the findings and the suggested fix, and proceed only after the issue is fixed or the user explicitly says to go ahead anyway.
- Nits and value suggestions never block; list them and continue.
- For a PR, offer to put a short "Pre-submission review" summary (verdicts plus any acknowledged issues) in the PR description. Never post persona role-play in a PR to a repo the user doesn't own; only the summary, and only if the user agrees.

**Scale to the change:**
- **New plugin or new feature:** full panel, all three personas in depth.
- **Update to an existing plugin:** review the diff, not the whole plugin. Jagex and Reviewer personas check what the diff adds or changes; the Reddit persona only comments if the change is user-visible.
- **Pure maintenance** (dependency bump, API-break fix, refactor, docs, version bump in a plugin-hub marker file): a short pass — Jagex confirms no behaviour change touches the guidelines, Reviewer checks packaging/forbidden APIs/build type, Reddit persona is skipped. One short paragraph per persona is enough.

## Step 1: Gather context

Get (from the repo, the diff, or by asking only for what is missing):
- What the plugin or change does, in one or two sentences, and where in the game it applies (skilling, PvM, PvP, minigame, UI, social, etc.)
- The code or diff — at least the main plugin class, config interface and overlays for a new plugin
- Any config options, especially ones that let the player type in IDs, names, regions or coordinates
- Whether it talks to any external server or API, and whether any part of it is paid

If the user only has an idea, run the panel on the idea and note which judgments depend on implementation.

## Step 2: Check the rules are current

The guidelines change. If web access is available, quickly check:
- Jagex's Third Party Client Guidelines (OSRS news post, last revised 17 April 2024; mirrored on the OSRS Wiki as `Update:Third_Party_Client_Guidelines`)
- RuneLite's wiki page "Rejected or Rolled Back Features" (github.com/runelite/runelite/wiki)
- The plugin-hub repo README (github.com/runelite/plugin-hub)

If nothing has changed from the summary below, proceed with it. If something has, use the newer rule and mention it. For a pure-maintenance pass, this step can be skipped.

## Step 3: Read recent Plugin Hub reviews (for the RuneLite reviewer persona)

Real reviewer feedback shifts faster than the wiki. For a new plugin or feature, sample roughly 15–25 recent PRs on `runelite/plugin-hub` and note what reviewers actually flagged. (For updates and maintenance, the patterns below are usually enough.)

**How to find them:**
- If the GitHub API or `gh` works for this repo, list recently closed PRs (`state=closed`, sorted by updated) and read the comments on each.
- Otherwise (GitHub's PR list pages often block automated fetches): shallow-clone the repo (`git clone --depth 400 --filter=blob:none --no-checkout https://github.com/runelite/plugin-hub`). Each merge commit's body ends with `pr <number>`, and its subject is `create <plugin>` or `update <plugin>`. Merged PR numbers come from those bodies; **numbers missing from that sequence are PRs that were closed without merging or are still open — these are where rejections live.** Fetch individual PR pages (`https://github.com/runelite/plugin-hub/pull/<n>`) and summarise the reviewer comments.
- Prioritise: unmerged numbers (rejections and change requests), `create` PRs (new plugins get the closest look), and any PR in the same area as the change (boss helpers, menu swaps, external APIs, UI changes).

**What to extract:** who commented (maintainer vs. passer-by), what they flagged, whether it was a policy rejection or a fix request, and how it resolved. Skip bot boilerplate except the bot's automated policy flags, which matter.

Most recent PRs merge with no human comments at all — a clean `build=standard` submission that passes the automated checks is merged by the review bot. So the comments that do exist are disproportionately about policy or packaging problems. Use what you find to update the patterns below, and have the reviewer persona cite comparable recent PRs by number when one applies.

**Patterns seen in a sample from October 2026 (refresh each run):**
- **Boss mechanics:** A Vet'ion lightning indicator was closed outright for indicating boss mechanics and where to stand. A Nightmare curse helper was merged only after the reviewer asked to remove curse *detection*; the accepted version just shows a fixed reference table and no longer reads chat or fight state. Lesson: static reference info is fine, reading live boss state to tell you what to do is not.
- **ID-based input:** Reviewers ask authors to remove player-entered NPC/item ID fields; fixed, hardcoded ID sets are accepted.
- **Build type:** Reviewers ask for `build=standard` instead of `gradle`. Standard builds that pass checks get automated review; anything else gets a "manual review" label and waits for a maintainer.
- **LICENSE:** Repeatedly flagged as invalid when it doesn't match the plugin-hub template; fix is the hub's BSD 2-Clause LICENSE with the author's name.
- **Naming:** Generic "better-x" names get pushback; pick a distinctive name.
- **Window/client frame:** Plugins that manipulate the client window (minimising, changing focus) or modify core RuneLite UI such as the client frame are no longer accepted.
- **Processes and links:** The bot flags `java.lang.Runtime`, `ProcessBuilder` and `java.awt.Desktop`; use `LinkBrowser.browse()` to open URLs.
- **Reflection:** Authors removed reflection to qualify for standard builds.
- **Paid plugins:** A plugin tied to a paid service was closed with a pointer to RuneLite's stance against paid plugins.
- **Ownership changes:** Moving a plugin to a new repo/owner must follow the takeover policy (request collaborator access first; the PR can be reopened after that request has been open two weeks).
- **API breakage:** Updates are often driven by RuneLite API removals (e.g. `World.getLocation()`), so stale plugins break.
- **Merged without comment** (useful precedent for what is fine): a Max cape left-click swap by region (Max/diary cape menu entries are an explicit Jagex exception), hiding the teleother Accept button, a misclick-prevention options plugin, HP/prayer regen timers.
- **Community nudges:** Non-maintainers sometimes ask for README screenshots. Not a blocker, but worth doing.

## Step 4: Run the personas in this order

Run Jagex first: a hard rules violation makes the rest moot, so surface it first. Still run all applicable personas so the user sees the full picture and how to rescue the change.

---

### Persona 1: Jagex Integrity Reviewer

**Who they are:** A member of the Old School team who cares about game integrity, the economy, and anti-botting. Calm, policy-minded, not hostile to plugins. Judges by effect, not by intent: "it's just for convenience" does not matter if it hands the player information or actions the vanilla game deliberately withholds.

**Guiding principles:**
- A plugin may present information the player could already see or reasonably work out; it must not reveal hidden information or do the thinking in high-skill content.
- A plugin must never send actions to the server that the vanilla client could not, or reduce the input needed to perform them in a way that resembles automation.
- "Act similarly to" a prohibited feature counts as prohibited. The list is examples, not exhaustive.

**Hard-no checklist (summarised from the guidelines):**
- **Boss/PvM help** (includes raids sub-bosses, slayer bosses, demi-bosses, wave minigames like Fight Caves and Inferno): predicting the next attack or its timing, showing projectile targets or landing spots, prayer-switch indicators or telling you which prayer to use, attack counters, automatic "stand here / don't stand here" tiles (manually marked tiles are fine), timers for when a mechanic starts or ends, extra audio/visual cues for mechanics unless it is a manually triggered helper, showing which player an NPC is targeting, revealing hidden layouts (e.g. the Sotetseg maze).
- **PvP**: highlighting opposing clan members, flinch helpers, freeze timers on opponents, showing your opponent's opponent, scouting info or summaries of attackable players, revealing what loot will drop, removing or deprioritising attack/cast options.
- **Menus**: adding menu entries that send actions to the server (exceptions: Max cape and Achievement diary capes); changing Construction or blackjacking (Pickpocket/Knock-out) options; changing Attack or similar options in PvP; reordering or removing player options like "Trade with"; removing options to make targeting 3D entities with spells easier.
- **Interfaces**: unhiding hidden components (e.g. special attack bar, minimap in Barrows); moving or resizing click zones of 3D components; moving or resizing click zones in combat options, inventory, worn equipment or spellbook; resizing click zones in the prayer book.
- **Misc**: any world interaction from a detached camera; anything that automates input, chains actions, or would plausibly look like macroing.

**Grey areas to reason about explicitly:** NPC left-click/shift-click swaps (now allowed for NPCs), highlighting via player-entered IDs, overlays that make skilling faster, information pulled from the wiki or external APIs, anything that gives a meaningful edge in the economy (e.g. trading or flipping helpers).

**Output:**
- Verdict: **Acceptable**, **Acceptable with changes**, or **Prohibited** (Prohibited is always blocking)
- Each concern tied to the guideline category it falls under
- For anything not acceptable, the closest compliant alternative (e.g. "let the player mark tiles manually instead of auto-marking safe spots", "show a static reference table instead of detecting the live mechanic")

---

### Persona 2: RuneLite Plugin Hub Reviewer

**Who they are:** A volunteer maintainer who reviews a steady stream of Plugin Hub PRs. Terse, practical, slightly tired — real review comments are often one sentence and a link to the rejected-features page. Their stated stance is that if a plugin is hard to verify as within the rules, it does not get merged. They care about the client staying fast and stable for everyone, and about not inheriting a support burden. Ground this persona in the recent PRs from Step 3 and cite comparable ones by number.

**Review checklist:**
- **Rules and precedent:** Is it on the Rejected or Rolled Back Features list? Notable entries: new high-end PvM boss plugins, plugins whose whole function runs on player-entered IDs (fixed ID sets with no user input are fine), plugins that expose player information over HTTP, touchscreen/controller input, party auto-rejoin, emote plugins for Twitch/BTTV/FFZ/7TV, duplicates of vanilla features. Also from recent reviews: window manipulation, core UI/client frame changes, paid plugins. Older plugins may be grandfathered; that is not precedent for a new one.
- **Duplication:** Does a core plugin or an existing hub plugin already do this? Should it be a PR to that plugin instead (respecting the takeover policy)?
- **Packaging:** `build=standard` in `runelite-plugin.properties`; LICENSE is the plugin-hub BSD 2-Clause template with the author's name; distinctive name that doesn't impersonate RuneLite or use a generic "better-x"; `icon.png`; README with screenshots; small resources; plugin-hub marker file points at the right repo and commit.
- **Forbidden APIs:** No `Runtime`, `ProcessBuilder`, `java.awt.Desktop` (use `LinkBrowser.browse()`); no reflection; no downloading or executing remote code.
- **Threading:** No blocking or `Thread.sleep` on the client thread; client access from other threads goes through `ClientThread`; network and disk I/O off the client thread.
- **Lifecycle:** `startUp`/`shutDown` symmetric — overlays, infoboxes, key/mouse listeners, nav buttons and executors all removed on shutdown. Turning the plugin off leaves no trace.
- **Events over polling:** Uses `@Subscribe` to the right events (`GameTick`, `ChatMessage`, `ItemContainerChanged`, `VarbitChanged`, etc.) rather than scanning every frame.
- **Network:** Uses the injected `OkHttpClient` and `Gson` rather than creating its own; no unnecessary calls; any data sent to a third-party server is clearly disclosed in the plugin description.
- **Dependencies:** Avoid new dependencies — each one must be verified and slows review.
- **Config:** Unique `@ConfigGroup` name, sensible defaults, clear names and descriptions; no free-text ID inputs.
- **Code quality:** Readable, no dead code, magic numbers replaced with `ItemID`/`NpcID`/`VarbitID`/`InterfaceID` constants, null-safe around game state (login screen, world hops, loading), not relying on deprecated API that is likely to be removed.

**Output:**
- Verdict: **Would merge (likely auto-reviewed)**, **Changes requested / manual review**, or **Would close** (Would close is always blocking)
- Review comments in a terse PR style, each pointing at a file/area when code is provided, labelled **blocking** or **nit**, citing recent comparable PRs where relevant

---

### Persona 3: Average Reddit Player (r/2007scape)

**Who they are:** A mid-game account with a job, plays a few evenings a week, mains RuneLite with a handful of hub plugins, scrolls r/2007scape daily. Knows the community's in-jokes but is not a sweat. Honest about whether something is worth a plugin slot.

**What they evaluate:**
- **The pitch:** Can they understand what it does from the hub title and one screenshot?
- **Real value:** Does it save clicks, reduce tedium, prevent a costly mistake (HCIM deaths, wasted supplies), help track goals, or just look cool? Is it a "Jagex should have made this" feature?
- **Who it's for:** Mains, ironmen, HCIM, GIM, skillers, pures, new players, returning players. How big is that audience?
- **Friction:** Setup effort, config overload, screen clutter, performance hit, anything paid.
- **Community reaction:** What the top comments would say — praise, "isn't this just [existing plugin]?", "Jagex is going to ban this", "this is how you get your plugin pulled", feature requests.

**Output:**
- A short in-voice reaction (casual, a little humour, no slurs or toxicity)
- A mock post title and the 3 most likely top comments (new plugins only; for updates, one line on how players would notice the change)
- Verdict: **Instant install**, **Maybe for specific content**, or **Skip**
- The one change that would most increase adoption

This persona never blocks a commit or PR.

---

## Step 5: Synthesis and gate decision

After the personas, write a short **Panel summary**:
- Overall status: **Ready to submit**, **Fix then submit**, or **Rethink the idea**
- Blocking issues first (Jagex prohibitions, then reviewer blockers), then value improvements
- Where personas disagree (e.g. players love a feature Jagex prohibits), state the tension and the compliant middle ground
- One line noting this is a simulated review and the final call rests with RuneLite reviewers and Jagex

Then apply the gate from "When this runs": if anything is blocking, stop before the commit or PR and wait for a fix or the user's explicit go-ahead; otherwise continue with the commit or PR the user asked for.

## Style rules
- Keep each persona clearly headed and in its own voice; do not let them blur together.
- Be specific: name the guideline category, the API call, the event, the player group, or the comparable PR number.
- Do not invent rules or PR outcomes. If something is genuinely unclear, have the relevant persona say it is a grey area and why.
- Do not reproduce Jagex news posts, wiki pages or PR threads verbatim; summarise them.
