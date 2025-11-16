CoaST – Calculator of All Structural Tuning
==========================================

CoaST (Calculator of All Structural Tuning) is a client-side web app for Magic: the Gathering players who care about how their decks *actually* behave: land counts, curve hills, ramp, cantrips, colour sources, and game plans are all treated as one connected system.

It is designed to be understandable and useful to:
- Long-term committed players
- Returning players
- New players
- Number-minded deckbuilders (think Karsten-style)
- Game designers / theorists (Garfield / Rosewater mindset)
- And, specifically, its creator: Foozi


1. What CoaST does
------------------

At a high level, CoaST is:

- A **deck builder + importer**
  - Build from scratch using a Scryfall-powered search box.
  - Or paste a text decklist in classic format: `4 Lightning Bolt` / `4 Lightning Bolt (2XM) 123`, etc.
  - Automatically fetches card data (name, mana value, type, colour identity, mana cost, art).

- A **structural analyzer**
  - Computes total **deck size**, **land count**, and **nonland weight**.
  - Computes **WPL (Weight Per Land)**: total nonland MV / land count.
    - This is the main “how heavy is this deck per land?” metric.
  - Builds a **curve hill**: how many cards at each effective mana value (MV 0–10).
  - Compares the curve against **anchor curves** for 40/60/99-card decks and common land counts
    and shows the difference per MV (Δ column).

- A **land consistency tool**
  - Uses hypergeometric math to estimate, for each turn 1–5:
    - P(≥1 land), P(≥2), P(≥3), P(≥4)
  - Does this twice:
    - For your literal land count.
    - For a “virtual land” count that treats tagged ramp pieces as +1 land each.

- A **colour balance checker**
  - Parses coloured mana symbols from card mana costs (W/U/B/R/G).
  - Sums total pips per colour.
  - Applies a simple rule of thumb:
    - Every ~3 coloured pips in the deck wants ~1 land source of that colour.
  - Compares **needed sources** vs **actual coloured land sources** (from colour identity on lands).
  - Shows per-colour: pips, needed sources, actual sources, and Δ.

- A **plan evaluator**
  - Lets you tag cards as being part of Plan A/B/C/D/E.
  - For a chosen plan and target turn (e.g. Plan A by turn 3):
    - Estimates P(“at least one copy of each Plan card is seen by that turn”).
    - Shows per-card “P(seen by T)” inside the plan.
    - Generates a sample “wishlist hand” of 7 cards that includes plan cards when possible.
    - Shows whole-deck composition as the likely follow-up draws:
      - Land %
      - 1–2 MV spells %
      - 3–4 MV spells %
      - 5+ MV spells %
    - Highlights likely bottlenecks: the cards in the plan that are least likely to show up by that turn.

- A **standalone land odds calculator**
  - Separate screen for quick math without a decklist.
  - Inputs:
    - Deck size
    - Land count
    - Max turn
    - On the play / on the draw
  - Outputs a simple table:
    - Turn, cards seen, P(≥1 land), P(≥2), P(≥3), P(≥4)


2. Files in this repo
---------------------

Right now the app is intentionally simple:

- `coast.html`
  - A single, self-contained HTML file.
  - Contains:
    - All HTML structure
    - All CSS (including responsive/mobile behavior)
    - All JavaScript logic
  - No build step, no bundler, no npm dependencies.

You can rename or add other files (e.g. `README.md`), but CoaST itself runs entirely from `coast.html`.


3. Tech stack / dependencies
----------------------------

- **Pure frontend**: HTML, CSS, vanilla JavaScript.
- **No backend**: all logic runs in the browser.
- **External dependency**:
  - Scryfall API (public):
    - `https://api.scryfall.com/cards/named?fuzzy=...`
    - `https://api.scryfall.com/cards/named?exact=...`
  - Used to:
    - Resolve card names.
    - Get mana value, type line, mana cost, colour identity.
    - Load card images for the grid view and the card modal.

Because of this, CoaST requires an **active internet connection** to:
- Search for cards
- Resolve imported card names to real card data

Once data is loaded, all calculations are done locally in the browser.


4. How to run CoaST locally
---------------------------

There is no installation step.

1. Download / clone the repo.
2. Open `coast.html` in any modern browser (Chrome, Firefox, Edge, Safari).
3. That’s it.

If you want to avoid CORS / local file issues:
- You can also serve the file using a simple HTTP server, e.g.:
  - `python -m http.server` (then open `http://localhost:8000/coast.html`)


5. How to deploy CoaST (e.g. GitHub Pages, Vercel)
--------------------------------------------------

### GitHub Pages

1. Create a new GitHub repo.
2. Commit `coast.html` and `readme.txt`.
3. Enable GitHub Pages (main branch, root or `/docs` depending on your preference).
4. Visit the published URL; CoaST should load in the browser.

### Vercel

1. Create a new project linked to your GitHub repo.
2. Framework preset: “Other” / “Static”.
3. Build command: *(leave empty)*  
   Output directory: `.` (root).
4. Deploy. Vercel will serve `coast.html` as a static file.


6. Using CoaST: main flows
--------------------------

### 6.1 Start screen

When you open CoaST you will see three main choices:

- **Build a deck**
  - Opens the deck builder with Scryfall search.
- **Import a deck**
  - Opens the deck importer so you can paste an existing list.
- **Land odds only**
  - Opens the standalone land odds calculator.

You can always go back to the start screen with the “← Home” button.


### 6.2 Deck builder / importer

The main deck screen has:

- **Deck header**
  - Deck name
  - Format:
    - Auto / Limited / Constructed / Commander / Custom
  - Target size:
    - Used to judge “off by how many cards” from your intended size.
  - Land count:
    - You can set this manually.
    - If you leave it at 0, CoaST will infer land count from any cards whose type line includes “Land”.

- **Builder mode**
  - “Build with search”:
    - Scryfall search box for adding cards:
      - Type the name, press Enter or click Add.
    - Quantity field for how many copies to add.
  - “Paste deck text”:
    - A textarea where you can paste your deck in text form (one card per line).
    - All parsing uses the standard `QTY Card Name (SET) #` style, but only quantity + name are required.

- **Deck view**
  - View:
    - Grid (card images, quantity, type, mana value, colour pips)
    - List (rows with qty, name, type, MV)
  - Group:
    - None (all together)
    - By Type (Creature, Sorcery, Land, etc.)
    - By Mana Value
    - By Colour (W, U, B, R, G, colourless)

- **Auto analysis**
  - CoaST re-analyzes the deck automatically whenever:
    - You add/remove cards
    - You change land count
    - You change format/target size
    - You change card tuning in the modal

You do **not** need to press an “Analyze” button; it updates on change.


### 6.3 Card detail modal (per-card tuning / agency)

Click any card (grid or list view) to open its detail modal.

You can configure:

- **Effective mana value (MV override)**
  - If a card is *usually* played differently from its printed mana value (e.g. a 7-drop routinely cast for 4), you can:
    - Input an “effective MV”.
  - CoaST uses this effective MV for:
    - Curve hills
    - Deck weight / WPL
  - If left blank, CoaST uses printed MV.

- **Ramp profile**
  - Optional tags to teach CoaST about structural ramp:
    - Ramp type:
      - Dork (creature)
      - Rock (artifact)
      - Land (land tutor / growth)
      - Burst (ritual / temporary)
      - Cheat (reanimate / sneak, etc.)
    - Online turn:
      - When this ramp effect realistically starts mattering (e.g. Birds of Paradise could be T2 ramp).
    - Net mana:
      - Approximate long-term net mana advantage (e.g. +1, +2).
  - Current version uses ramp tags primarily to:
    - Count “virtual lands” (each ramp copy ≈ +1 land source).

- **Cantrip / selection**
  - Extra cards seen (average):
    - For example:
      - Scry 2 → pick 1 might be ~0.6
      - Impulse-style look at 4 choose 1 might be ~1.2–1.5
  - This is stored for future deeper plan math; currently it is displayed in per-card summary and aggregated in the deck summary.

- **Plan tags**
  - Checkboxes for Plan A / B / C / D / E.
  - CoaST uses these in the Plan panel to:
    - Group cards by plan.
    - Compute how likely it is to have all necessary pieces by a given turn.
    - Build a sample “wishlist hand”.

Each card also shows a **mini summary** describing:
- Its contribution to total deck weight.
- Whether an effective MV override is in use.
- Whether it is considered ramp or a cantrip.
- Which plans it belongs to.


### 6.4 Summary panel (right side)

The summary panel presents:

1. **Deck snapshot**
   - Deck size, lands, nonlands.
   - WPL (Weight Per Land).
   - Count of ramp copies.
   - Aggregate cantrip “extra cards” sum.

2. **Curve vs anchor hill**
   - Table of MV → Cards in deck vs Anchor vs Δ.
   - Anchor curves come from “hill” shapes for common deck sizes and land counts, scaled to match your nonland count.
   - Δ highlights where you are over/under the hill:
     - Big positive:
       - Heavier than the anchor.
     - Big negative:
       - Lighter than the anchor.

3. **Land availability (real lands)**
   - Per turn row:
     - Turn
     - Cards seen (opening hand + draw steps)
     - P(≥1 land)
     - P(≥2)
     - P(≥3)
     - P(≥4)

4. **Virtual lands (with ramp)**
   - Same table, but using:
     - Virtual land count = actual lands + total ramp copies.
   - This is a deliberately conservative model that says:
     - “Each ramp piece roughly behaves like +1 land source over time.”

5. **Colour pips vs land sources**
   - Per colour:
     - Pips (from mana costs)
     - Needed sources (≈ pips / 3)
     - Actual sources (coloured lands by colour identity)
     - Δ (have – need)


### 6.5 Plan panel (Plans A–E)

Under the summary panel is the Plan section.

- **Plan tabs**
  - Plan A, B, C, D, E.
  - Each plan has its own target turn.

- **Target turn**
  - Input box for “when this plan should be online” (e.g. 3, 4, 6…).

- **Plan calculations**
  - For all cards tagged with that plan:
    - Calculates for the deck:
      - Cards seen by the target turn (7 + draw steps).
      - P(seen ≥1 copy) for each plan card.
      - Approximate “all pieces online by T” probability (product of these per-card probabilities).
  - Displays:
    - Per-card table: qty and P(seen ≥1 by T).
    - Sample “wishlist hand”:
      - A 7-card sample that includes plan cards where possible.
    - Whole deck composition for follow-ups:
      - Land, 1–2 MV, 3–4 MV, 5+ MV.
    - Bottlenecks:
      - Which plan cards are least likely to show up by the target turn.


7. Standalone land odds screen
------------------------------

If you only need land math, choose “Land odds only” on the home screen.

Inputs:
- Deck size
- Land count
- Max turn to calculate to
- On the play or on the draw

Output:
- Text table showing, for each turn:
  - Cards seen
  - P(≥1 land)
  - P(≥2)
  - P(≥3)
  - P(≥4)

This is a pure hypergeometric calculator and does not depend on any decklist.


8. Saving / loading decks
-------------------------

CoaST uses `localStorage` in your browser to save decks.

- **Save locally**
  - Saves:
    - Deck name
    - Card list
    - Per-card tuning (MV overrides, ramp, cantrip, plans)
    - Format
    - Target size
    - Land count
    - Plan target turns
- **Saved decks dropdown**
  - Select a saved name and click “Load” to restore it.
  - “Delete” removes it from localStorage.

This data is stored **only** on the device / browser you used. There is no backend or sync service.


9. Limitations / known behavior
-------------------------------

- No account system or server:
  - All data is local to your browser.
- No actual game engine:
  - CoaST is a *structural calculator*, not a rules engine or full game.
- Ramp / cantrip modeling:
  - Current version uses simple heuristics:
    - Each ramp copy ≈ +1 virtual land.
    - “Extra cards seen” from cantrips is stored but not yet deeply integrated into draw-by-turn calculations.
- Anchor curves:
  - Current anchor hills are a skeleton based on 40/60/99-card baselines and can be extended later from more complete tables.
- Scryfall dependency:
  - If Scryfall is down or blocked, card details and images may fail to load; the app will still try to fall back on basic info.


10. Future directions (not yet implemented)
-------------------------------------------

These are ideas intentionally left for future expansions:

- **CoaSTBot (Discord integration)**
  - A bot that:
    - Accepts decklists via chat.
    - Replies with core CoaST metrics (WPL, land odds, curve hill).
    - Potentially integrates plan calculations.
- **Multiplayer playtest / goldfishing**
  - A real-time or async playtest environment where players can:
    - Import decks and goldfish solo.
    - Play 1v1, Two-Headed, or 4-player Commander test games.
  - This will require:
    - A backend server.
    - Game state management.
    - Possibly WebSockets or similar.

Right now this repository focuses on the **core structural calculator**:
- All the math around curves, land odds, colours, and plans.
- In a single, inspectable, forkable HTML file.


11. License / credit
--------------------

- Card data and images via **Scryfall**:
  - Scryfall is unaffiliated with this project.
  - Please respect Scryfall’s API rules and terms of service.
- Magic: the Gathering, card names, and mana symbols are property of Wizards of the Coast.
- CoaST is a fan-made tool meant to help players understand deck structure and tuning.

Creator / concept driving force: **Foozi (Foozinacci)**.

If you fork or extend CoaST, please keep attribution and respect Scryfall and WotC policies.
