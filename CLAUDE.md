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

## Editing data

Everything editable is in one `<script>` block at the top of `index.html`, fenced by comment banners.

```js
const START = [2026, 8, 2];   // the Sunday, 1-indexed month
const NIGHTS = [ { meal, who, side1, side2, note }, ... ];  // exactly 6, Sunday→Friday
```

`side1`, `side2` and `note` may each be an empty string and will be omitted from the render — a night with one side renders one, a night with none drops the line. `NIGHTS` must stay at length 6; the day labels and the "of 6" counter assume it.

Sides render as `side1 · side2` directly under the meal name, in both the hero and the week list. The separator dot is `--lake` so `--lantern` stays reserved for tonight, the cook name and the eyebrow.

`who` is the exception to the empty-string rule: left as `""` it renders **Up for grabs** rather than disappearing, so an unclaimed night looks unclaimed. It's muted in the hero and italic in the week list — it isn't a name, so it isn't styled like one. Fill in a name and it goes back to normal.

Meals and sides are real, and two nights have cooks. `START` is still a guess, pending from the owner.

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
- Cooks for four of the six nights — Tuesday through Friday are "Up for grabs"
- Sides are recommendations, not confirmed with whoever's cooking
