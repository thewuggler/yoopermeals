# yoopermeals

What's for dinner tonight. Six dinners, Sunday through Friday, on one page.

The whole site is [`index.html`](index.html). No build step, nothing to install —
GitHub Pages serves it straight off `main`.

## Changing the meals

Open `index.html` here on GitHub, tap the pencil, and edit the block at the very
top marked **EDIT HERE**. It's the first thing in the file, so you don't have to
scroll. Commit, and the page updates in a minute or so.

```js
const START = [2026, 8, 2];   // the Sunday the trip starts — month is 1-12

const NIGHTS = [
  { meal: "Pasties", who: "Mom", note: "Rutabaga in half of them." },
  ...
];
```

Three things worth knowing:

- `NIGHTS` has to stay at six entries, Sunday first.
- `note` can be `""` — it just won't show up.
- The page works out what day it is from the phone that's reading it. Nothing to
  switch over each morning.

Everything else — colours, type, the treeline — is documented in
[`CLAUDE.md`](CLAUDE.md).
