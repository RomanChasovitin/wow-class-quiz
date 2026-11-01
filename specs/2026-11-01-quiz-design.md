# WoW Forever class quiz — design

Status: agreed 2026-11-01 (handoff from life, then the discuss round in this repo).

## Goal

A Russian-language quiz that tells a player which role, class and race fits them in WoW Forever,
from indirect questions about habits, tastes and group behavior. First real users: Roman's friends
Серега and Артур, who pick classes for a party playthrough. Quality bar: good visual design and
high-quality questions.

## Users and constraints

- Players who know WoW a little or not at all; the questions must not need game knowledge.
- Russian UI and questions. Class and race names follow the ruRU client
  (`docs/forever-race-class.md`).
- One static HTML file: no backend, no build step, works offline once loaded, works on a phone.
- Shareable by link: this repo becomes public and is served by GitHub Pages from `main`
  (accepted 2026-11-01).

## Data the quiz rests on

- Race × class × faction: `docs/forever-race-class.md` (5 sources, all agree).
- Roles per class in Forever, from Warcraft Tavern's class guide
  (https://www.warcrafttavern.com/forever/guides/classes/, checked 2026-11-01):

| Class | Tank | Healer | Melee DPS | Ranged DPS |
|-------|:----:|:------:|:---------:|:----------:|
| Warrior | Protection | | Arms, Fury | |
| Paladin | Protection | Holy | Retribution | |
| Hunter | | | Survival | Beast Mastery, Marksmanship, Survival |
| Rogue | | | all three | |
| Priest | | Discipline, Holy | | Shadow |
| Shaman | Enhancement (partial) | Restoration | Enhancement | Elemental |
| Mage | | | | all three |
| Warlock | | | | all three |
| Druid | Feral | Restoration | Feral | Balance |

  Shaman tank is new and disputed: Warcraft Tavern and the Mobalytics Enhancement guide say yes
  (beta), other guides list no tank role. The quiz treats it as a partial fit (see scoring). Hunter
  melee (Survival) is also a partial fit.

## Observable behavior

### Flow

1. **Start screen.** Title, one line on what the quiz does, depth choice: «Лёгкий» (10 questions)
   and «Обычный» (30 questions). Deeper levels (60, 100) are shown as "скоро" or hidden — builder's
   choice, no logic for them in v1.
2. **Questions.** One question per screen, 3–4 answers, single choice, tap to answer and move on.
   A progress bar and a "назад" button that changes the previous answer. No game terms in the
   questions or answers.
3. **Result screen.**
   - Role: top 3 with match percent; the first one is the headline.
   - Class: top 3 with match percent and the spec that fits the top role (e.g. «Паладин — Защита»).
   - Faction switch «Альянс / Орда» (default: Альянс; no faction question in the quiz).
   - Race: top 3 for the chosen faction, **only races that can play the selected class in that
     faction**. The selected class is the top-1 class; tapping another class in the top 3 selects
     it instead. When fewer than 3 are valid, show only those (Horde Paladin → only Нежить).
     Switching faction recalculates races at once, without retaking the quiz.
   - Skyborne toggle «Учитывать Небеснорожденных» (default: on). Off removes both Skyborne variants
     from race matching. Note next to it: the race needs a paid edition.
   - "Почему": for the headline role, class and race, 2–3 short reasons taken from the answers that
     contributed most.
   - "Поделиться": copies a link that opens the same result screen (see Share link).
   - "Пройти ещё раз".

### Share link

The result is encoded in the URL fragment (`#r=...`): depth level and the chosen answer indexes,
plus faction and Skyborne toggle. Opening the link recomputes the result from the answers, so a link
stays valid after weights change, as long as the question bank keeps the same IDs and order. A
broken or old link falls back to the start screen without an error dialog.

## Mechanics

### Axes

Each answer adds weights to a vector of axes. Classes and races have profiles on the same axes.

- **Role axes** (class matching): `tank`, `heal`, `melee`, `ranged`.
- **Playstyle axes** (class matching): `burst` (−1 steady … +1 burst), `complexity`, `pet`,
  `stealth`, `utility` (buffs, control, support), `solo` (−1 group … +1 solo), `magic`
  (−1 physical … +1 magic), `shadow` (−1 light … +1 dark themes), `nature`.
- **Race axes** (race matching): `wild` (−1 civilization … +1 nature and tribe), `grim`
  (−1 bright … +1 gloomy), `bulk` (−1 small and agile … +1 big and strong), `whimsy` (−1 serious …
  +1 playful), `mystic` (−1 practical and technical … +1 arcane and ancient).

The builder may merge or split axes if a profile cannot tell two classes or races apart; record the
change here.

### Scoring

- Player vector: sum of the chosen answers' weights, divided per axis by the maximum reachable
  absolute sum on that axis for the chosen depth. Axes no question touched are left out of every
  comparison.
- Role weights are never negative: an answer adds to the roles it points at and leaves the rest.
- Role: the four role axes, ranked; percent = share of the role's value in the total of positive
  role values.
- Class: `score = 0.5 × roleFit + 0.5 × playstyleSimilarity`. `roleFit` is the player's role
  distribution dotted with the class's role capability (1 for a full spec, 0.5 for a partial one:
  Shaman tank, Hunter melee). `playstyleSimilarity` is cosine similarity on playstyle axes, mapped
  to 0…1; a zero vector counts as 0.5. The weights 0.5/0.5 are a starting point;
  tune them with the reachability check below.
- Spec shown: the class's spec for the player's top role; if the class has none, its spec for the
  player's best role it does have.
- Race: cosine similarity on race axes among the valid races, mapped to 0…1. A race profile may
  also carry a small class-theme bonus (e.g. Таурен and `nature`) — only if the check shows it is
  needed.
- Percent: the score mapped to 0–100 and rounded; shown for ranking only, no claim of precision.
- "Почему": per answer, the contribution to the winner is the dot product of its weights with the
  winner's profile. Each answer carries a short Russian reason phrase («любишь держать удар за
  других»); show the phrases of the 2–3 top contributors.

### Question bank

- Each question has a stable ID, Russian text, 3–4 answers; each answer has weights on a few axes
  and a reason phrase.
- Easy (10) is a subset of Normal (30): Normal = Easy + 20. Easy touches every axis at least once;
  Normal touches each class axis at least 3 times and each race axis at least 2 times.
- The answers within a question pull in different directions; no answer is the obvious "good" one.
- Questions are indirect: daily habits, tastes, how you act in a group, what you like in films and
  games. Never «хочешь ли ты танковать?».
- The agent writes the bank in Russian; Roman reviews the text before the UI is built on it.

## Visual design

New UI with no reference yet, so the first design goes to a Claude Fable 5.1 subagent at `medium`
effort. This is a personal project, so it runs on the Personal subscription
(`claudeAgent_personal`), not on a corporate seat. The brief gives: goal and users, the flow above,
phone-first, one HTML file, real Russian copy from the question bank, no Blizzard art assets, and
asks for one complete HTML mock of start, question and result screens.

### Accepted visual decisions

None yet. After each review round, list the accepted values here (e.g. "background #…; answer card
radius 12px"). A new request that reverses a value names the old value in one line.

## Exclusions (v1)

- Depth levels Hard (60) and Extra deep (100).
- Party comparison (several friends' links on one screen).
- Racial abilities as a matching factor: race matching is look, vibe and lore only.
- Blizzard images, icons or fonts.
- Analytics, accounts, saving between visits.
- English UI.

## Assumptions

- The race × class table and the roles above match the live game on 2026-11-04. Recheck both after
  launch; a change is a data edit, not a logic change.
- The Skyborne variant names in Russian need a check on the ruRU character creation screen (see
  `docs/forever-race-class.md`, "Open").
- GitHub Pages for this repo needs it public; Roman accepted making it public on 2026-11-01.

## Acceptance criteria

1. The page is one HTML file and works from `file://` and from GitHub Pages, on a phone-width
   screen (360px) and on desktop.
2. Easy asks 10 questions, Normal 30; "назад" changes an answer and the result follows it.
3. Every class can come out as top-1, and every race variant as top-1 race for at least one class
   and faction (reachability check, below).
4. No result ever shows a race that cannot play the selected class in the chosen faction, with the
   Skyborne toggle on or off.
5. The faction switch and the Skyborne toggle recalculate races at once.
6. A share link opens the same result in a fresh browser.
7. All visible text is Russian; class and race names match `docs/forever-race-class.md`.

## Verification

- **Reachability check:** a function in the page, run from the browser console, that samples
  random answer sets (at least 10,000 per depth) and prints how often each class and race is top-1.
  No class may be below 3% or above 25% on Normal; tune the profiles until it holds. No test stack;
  this check is the test.
- **Archetype runs:** 4 hand-written answer sets (a clear tank, healer, melee and ranged player) land
  on the expected role.
- **Rule check:** the console function also asserts criterion 4 for every class × faction × toggle.
- Browser checks follow `~/.agents/browser.md`; Roman reviews the visuals himself.

## Sequencing

1. Data: class and race profiles, role capability table, as JS data in the page.
2. Question bank Easy + Normal, in Russian → Roman reviews the text.
3. Design mock by the Fable subagent → Roman's review rounds → accepted decisions above.
4. Build the page on the accepted design; run the checks.
5. Make the repo public, turn on GitHub Pages, send the link.
