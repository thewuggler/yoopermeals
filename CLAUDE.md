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

Below the hero, all six nights render as a list. Past nights dim to 38% opacity, tonight gets the accent color and bold weight. Every night shows who's at the table that night — see below; the crowd isn't the same all week.

The week list sits under the first of three tabs — **The week**, **Kids menu**, **Shopping**. A night that has recipes shows them as `<details>` disclosures inside its row, closed by default so the page still answers "what's for dinner" without a tap.

## Editing data

Everything editable is in one `<script>` block at the top of `index.html`, fenced by comment banners.

```js
const START   = [2026, 8, 2];   // the Sunday, 1-indexed month
const NIGHTS  = [ { meal, who, adults, kids, side1…side4, note, cook }, ... ];  // exactly 6, Sunday→Friday
const KID_APPETITE = [ { under, eats }, ... ];  // age → share of an adult portion
const KID_EATS = 0.5;                           // when a night gives a count, not ages
const KIDS    = [ "Chicken Nuggets", ... ];     // no schedule, any length
const RECIPES = { key: { title, source, url, time, covers, serves, makes, ingredients, steps } };
const EXTRAS  = [ { qty, unit, item, group, covers }, ... ];   // on the list, not from a recipe
```

`side1` through `side4` and `note` may each be an empty string or left off entirely and will be omitted from the render — a night with one side renders one, a night with none drops the line. Four is the ceiling because Sunday needs four; the renderer reads exactly those keys, so a fifth side means teaching `sidesInto` and `looseEnds` about it. `NIGHTS` must stay at length 6; the day labels and the "of 6" counter assume it.

Sides render as `side1 · side2 · …` directly under the meal name, in both the hero and the week list. The separator dot is `--lake` so `--lantern` stays reserved for tonight, the cook name and the eyebrow.

`who` is the exception to the empty-string rule: left as `""` it renders **Up for grabs** rather than disappearing, so an unclaimed night looks unclaimed.

Not every night has a person cooking, so `AS_IS_COOKS` lists the cook lines that are statements about the night rather than names — currently "Up for grabs", "Every person for themselves" and "Order out". Those print exactly as written, with no "Cooked by" in front, and are styled as a phrase: muted in the hero, italic in the week list. Anything not in that list is treated as a name and gets the prefix and the accent colour. Add a phrase to the list rather than teaching the renderer to guess.

Meals and sides are real, and four nights have cooks — Tuesday is a free-for-all and Friday is ordered out. `START` is the real trip date, confirmed by the owner: Sunday August 2nd 2026 through Friday the 7th.

Sunday carries four sides, which is the most any night has: coleslaw, green beans with bacon, tater tots and cornbread.

## Who's at the table

The crowd changes through the week, so each night carries its own `adults` and `kids`. It prints on the night — a bordered pill under the cook line in the hero, a quiet line in the week row — and it sets how far the recipes are scaled. Currently: Sunday 7 adults and 7 kids, Monday and Tuesday 9 and 7, Wednesday through Friday 7 adults and 4 kids.

`kids` is either a plain count or a list of ages. Ages are better and Wednesday onward has them (`[13, 11, 7, 3.5]`), because a thirteen-year-old eats an adult dinner and a three-year-old does not. `KID_APPETITE` maps an age to the share of an adult portion it actually eats — 0.35 under 5, 0.55 under 10, 0.75 under 13, then a full share. A count with no ages uses `KID_EATS`, a flat half. Those numbers are a judgement call, not a fact; edit them if the appetites are wrong.

Adding up adults plus each kid's share gives the night's **portions**, which is the only thing the scaling cares about. Sunday is 10½, Monday and Tuesday 12½, Wednesday through Friday 9.65. Head *count* and portions are deliberately different numbers: seven kids is seven place settings but nothing like seven dinners, and only one of those two facts helps you buy meat.

Leave `adults` off a night and the indicator disappears and its recipes stay at their published size — no guessing.

## Recipes and the shopping list

A night lists `RECIPES` keys in its `cook` array and renders those recipes; leave `cook` off and the night is just a name and its sides. Every night now has at least one — eighteen recipes across the week, one per meal and per side except the things nobody cooks (Tuesday's free-for-all, Friday's pizza and garlic knots, the tater tots).

A recipe's `covers` names the meal or side it answers for, written **exactly** as the night writes it — `"Coleslaw"`, `"Meatballs & Pasta"`. That string, and nothing else, is what takes a line off "Still to work out". Two recipes may cover the same thing; the meatballs and the marinara both cover Monday's dinner. Matching is case-insensitive, but the words have to line up, so renaming a side means renaming its `covers` too — otherwise the side quietly reappears in the loose ends, which is the failure mode you want over a silent drop.

`EXTRAS` take a `covers` as well, for the things you buy instead of cook: the tater tots and the kids' standbys are all bought outright, so they carry the line rather than a recipe. An extra's `covers` applies all week — it's bought once, not per night.

Ingredients are `{ qty, unit, item, prep, group }`. `qty` is a **number** — never a string like `"1½"` — because the shopping list scales it and adds it up, and only turns it back into a fraction when it prints. `prep` ("minced", "chopped") shows in the recipe and is dropped from the shopping list. `group` is the aisle: Produce, Meat, Dairy, Bakery, Pantry, in that order. Anything else — a typo like `"produce"`, or a new aisle — gets its own section after those five rather than being dropped. That's deliberate: a silently missing ingredient is the one failure this tab can't have, and an odd heading is how you notice.

**Store recipes at their published amounts** so the original stays checkable against the source, and say how many grown-up portions that published recipe feeds in `serves`. The batch size is then derived, per night: the portions at that night's table divided by `serves`, **rounded to the nearest half** — half a recipe is a thing you can stand at a counter and cook, 1.79 of one is not. Everything follows from it, every quantity and the yield, on the page and in the shopping list.

Monday's 12½ portions against meatballs at `serves: 7` (14 meatballs, two apiece) gives 2× — 28 meatballs. Against marinara at `serves: 4` (a quart sauces four, pasta and all) it gives 3× — ~3 quarts. Those are the same amounts the page shipped with when the multipliers were hand-set, which is the check to re-run if `serves` or the head counts move.

The batches the week currently lands on, which is the table to re-check whenever `serves` or a head count changes:

| Night | Portions | Batches |
|---|---|---|
| Sunday | 10½ | ribs 2×, slaw 1×, greenbeans 1×, cornbread 1× |
| Monday | 12½ | meatballs 2×, marinara 3×, garlicbread 2×, italiansalad 2× |
| Tuesday | 12½ | pastasalad 1×, veggietray 1× |
| Wednesday | 9.65 | tacos 2½×, spanishrice 1×, guacamole 2× |
| Thursday | 9.65 | grillnight 1½×, hotdogs 1×, cornonthecob 1½×, potatosalad 1× |
| Friday | 9.65 | gardensalad 1× |

Several `serves` values were picked so the night lands on a whole batch rather than an awkward one — the cornbread is `serves: 12` because a 9×13 pan is what you actually bake, not because twelve people eat it alone. Changing a head count will move these.

A literal `scale` on a recipe pins the batch and ignores the head count entirely. Nothing uses it now; reach for it only when a recipe genuinely isn't about how many people are eating.

The recipe's meta line prints the multiplier *and* the crowd it was sized for — "2× the original · For 9 adults & 7 kids" — so the number is checkable against the night rather than taken on faith. Don't drop that.

`shop` overrides what an ingredient contributes to the list:

- `shop: false` keeps it off entirely — the meatballs' tap water, and the cornbread's gluten-free flour, which the owner is bringing from home rather than buying up there.
- `shop: { qty, unit, item }` substitutes what you actually buy: "9 tbsp onion" is useless in a store, "1 large yellow onion" is not. **This override is taken as written and is never multiplied** — it's the real amount for the cart. Since the batch size now moves on its own when a night's heads change, these are the lines to re-check by hand afterwards.

Two lines merge on the list only when item *and* unit match exactly — cups are never silently added to tablespoons. That's why olive oil combines across both recipes (2 cups + ¾ cup → 2¾ cups) but the Pecorino in the meatballs stays separate from the Pecorino for the table. Ingredient names are deliberately spelled the same across recipes where they should merge — "butter" not "salted butter", "Parmesan" not "shaved Parmesan", "black pepper" not "coarse black pepper" — so that eggs, butter, garlic and cherry tomatoes come out as one line each instead of five. The distinguishing word goes in `prep`, which the list drops.

An unit-less countable item carries its own plurality in the name: "large eggs", "hard taco shells", "roma tomatoes". The renderer pluralizes units, never item names, so an item written singular will read "4 small red onion" once quantities merge past one. That's a handwritten-list idiom and it's left alone rather than taught to the renderer to guess at.

Groups outside the five aisles get their own section after them, and the list currently uses one on purpose: **Frozen**, for the tater tots and the chicken nuggets.

Below the aisles, **Still to work out** lists every meal and side with nothing behind it — no recipe covering it, no extra buying it — plus any of the kids' standbys nobody's shopping for. It's generated, not hand-maintained: fill in a recipe or an extra with the right `covers` and the line leaves that section on its own. Keep it — the gaps are the reason to open this tab before the trip. Each night there carries its head count, since with nothing to size it that's the only quantity the line can give you.

What's left there now is honest and short: Tuesday's Free-for-All, and Friday's pizza and garlic knots, which are ordered rather than shopped.

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

- Every night is now spoken for — nothing is "Up for grabs"
- Sides are recommendations, not confirmed with whoever's cooking
- Head counts are confirmed, including the Wednesday-onward drop from 9 adults to 7. Every recipe amount on the page moves with those numbers, so a late change to who's coming is a re-check of the whole shopping list
- `KID_APPETITE` is a guess at how much kids eat, and now the whole week's quantities rest on it
- The recipes the owner didn't dictate are sensible, well-known versions of each dish rather than a specific family recipe, and several cite "Adapted from" a page that couldn't be opened from here — the quantities are sound but not transcribed. Worth a look before anyone cooks from them. The green beans, the coleslaw and the cornbread came from the owner directly and say "House recipe"
- **Grill Night** is burgers and hot dogs, as two recipes both covering the night: 9 patties and 8 dogs for a table of eleven. That's deliberately generous — the owner asked for the burgers back at full size after they were briefly trimmed. Both recipes read straight, `serves` meaning what it means everywhere else on the page
- Friday's pizza and garlic knots sit under "Still to work out" because they're ordered, not shopped. That reads correctly now that everything else has left the section, so the old question of whether an ordered-out night should drop off it is answered: leave it
- Nobody has checked the list against what's already in the cabin, so staples like oil, salt and sugar are on it whether or not they need buying
