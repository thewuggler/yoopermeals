# yoopermeals

Static one-page site that answers a single question for a family on vacation: **what's for dinner tonight?**

Live at GitHub Pages off `main` / root. The entire site is `index.html`. There is no build step, no package manager, no framework, and no dependency other than a Google Fonts stylesheet link.

## Who reads it

Five households on a shared trip — the owner's family, his three siblings' families, and his mom. Mixed ages and mixed technical comfort. Everyone opens the link on a phone, usually standing in a kitchen deciding whether to start cooking.

## What it is and isn't

**Is:** read-only. Information flows one direction. The owner edits the file; everyone else reads it.

**Isn't:** interactive. No sign-ups, no claiming nights, no editing in the browser, no backend, no storage. This was decided explicitly — schedule changes get texted in the family group thread and the owner updates the file. Do not add collaborative features, forms, or persistence unless asked directly.

## Structure

Six dinners, Sunday through Friday. Main meals only — no breakfast or lunch.

The page reads the device's local date, computes how many days have elapsed since `START`, and uses that index into `NIGHTS`. Three states:

| Condition | Renders |
|---|---|
| Before the trip | Countdown plus a preview of night one |
| During (index 0–5) | Tonight's meal as the hero |
| After Friday | A sign-off |

Below the hero, all six nights render as a list. Past nights dim to 38% opacity, tonight gets the accent color and bold weight.

The week list sits under the first of three tabs — **The week**, **Kids menu**, **Shopping**. A night that has recipes shows them as `<details>` disclosures inside its row, closed by default so the page still answers "what's for dinner" without a tap.

## Editing data

Everything editable is in one `<script>` block at the top of `index.html`, fenced by comment banners.

```js
const START   = [2026, 8, 2];   // the Sunday, 1-indexed month
const NIGHTS  = [ { meal, who, side1, side2, note, cook }, ... ];  // exactly 6, Sunday→Friday
const KIDS    = [ "Chicken Nuggets", ... ];     // no schedule, any length
const RECIPES = { key: { title, source, url, time, scale, makes, ingredients, steps } };
const EXTRAS  = [ { qty, unit, item, group }, ... ];   // on the list, not from a recipe
```

`side1`, `side2` and `note` may each be an empty string and will be omitted from the render — a night with one side renders one, a night with none drops the line. `NIGHTS` must stay at length 6; the day labels and the "of 6" counter assume it.

Sides render as `side1 · side2` directly under the meal name, in both the hero and the week list. The separator dot is `--lake` so `--lantern` stays reserved for tonight, the cook name and the eyebrow.

`who` is the exception to the empty-string rule: left as `""` it renders **Up for grabs** rather than disappearing, so an unclaimed night looks unclaimed.

Not every night has a person cooking, so `AS_IS_COOKS` lists the cook lines that are statements about the night rather than names — currently "Up for grabs" and "Every person for themselves". Those print exactly as written, with no "Cooked by" in front, and are styled as a phrase: muted in the hero, italic in the week list. Anything not in that list is treated as a name and gets the prefix and the accent colour. Add a phrase to the list rather than teaching the renderer to guess.

Meals and sides are real, and two nights have cooks. `START` is still a guess, pending from the owner.

## Recipes and the shopping list

A night lists `RECIPES` keys in its `cook` array and renders those recipes; leave `cook` off and the night is just a name and two sides, as before. Monday points at `meatballs` and `marinara`, both Rao's.

Ingredients are `{ qty, unit, item, prep, group }`. `qty` is a **number** — never a string like `"1½"` — because the shopping list scales it and adds it up, and only turns it back into a fraction when it prints. `prep` ("minced", "chopped") shows in the recipe and is dropped from the shopping list. `group` is the aisle: Produce, Meat, Dairy, Bakery, Pantry, in that order.

`scale` multiplies the whole recipe — every quantity and the yield. **Store recipes at their published amounts and change `scale`**, so the original stays checkable against the source. The party is 9 adults and 7 kids, so meatballs run at 2 (28 meatballs) and marinara at 3 (~3 quarts, since it sauces the pasta as well).

`shop` overrides what an ingredient contributes to the list:

- `shop: false` keeps it off entirely — tap water is the only current case.
- `shop: { qty, unit, item }` substitutes what you actually buy: "9 tbsp onion" is useless in a store, "1 large yellow onion" is not. **This override is taken as written and is not multiplied by `scale`** — it's the real amount for the cart, so revisit these by hand if `scale` changes.

Two lines merge on the list only when item *and* unit match exactly — cups are never silently added to tablespoons. That's why olive oil combines across both recipes (2 cups + ¾ cup → 2¾ cups) but the Pecorino in the meatballs stays separate from the Pecorino for the table.

Below the aisles, **Still to work out** lists every night and side with no recipe behind it, plus the kids' standbys. It's generated, not hand-maintained: fill in a recipe and the night leaves that section on its own. Keep it — the gaps are the reason to open this tab before the trip.

**Copy the list** writes the whole thing out as plain text for the group thread. It's an export, not storage, and it's the only button on the site that does anything.

## Design

The direction is Upper Peninsula camp signage — painted plank lettering, lantern light, treeline. It is deliberately not a generic dashboard.

```
--pine:      #0F2E28   page ground
--pine-lift: #16403A   the week section
--birch:     #EDE4D3   primary text
--lantern:   #F2A93B   accent — tonight, cook name, eyebrow
--lake:      #3C8079   night numbers, counter
--ash:       #7E8F89   secondary text
```

Display face is **Ultra** (woodtype slab, used only for the meal name). Body and UI is **Space Grotesk**. The hero meal name scales with `clamp(2.9rem, 13.5vw, 5.5rem)` — it should stay enormous, that's the point of the page.

The one signature element is the hero: lantern-glow radial behind an oversized meal name, with the hand-cut SVG treeline separating it from the week list. Keep the boldness concentrated there and everything else quiet.

## Rules for changes

- Stay single-file. No bundler, no npm, no imports beyond the fonts link.
- Mobile-first. Test narrow viewports before wide ones; desktop is the afterthought here, not the reverse.
- Preserve the reduced-motion block and visible focus states.
- Keep the config block at the top and clearly fenced — a non-developer edits it from the GitHub web UI on a phone.
- Don't reach for a date library. The current local-midnight arithmetic is correct and dependency-free.
- Open pull requests ready for review, never as drafts.

## Open items

- Real trip dates — `START` is currently a placeholder guess
- Cooks for three of the six nights — Wednesday, Thursday and Friday are "Up for grabs"
- Sides are recommendations, not confirmed with whoever's cooking
- Recipes for the other five nights, and for the sides — everything without one shows under "Still to work out" on the shopping tab
- The shopping list has no quantities for anything outside a recipe; `EXTRAS` currently covers only the spaghetti and the table cheese
