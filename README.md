# Clock (`cmux/clock`): a lock-screen style clock widget

An example `cmux.widget/1` widget: install it, then copy this repository to make your own.

## Install

```sh
cmux ext add manaflow-ai/cmux-widget-clock
```

Then open Edit Widgets on the New Tab page or in the sidebar and add it.

## Make your own from it

```sh
git clone https://github.com/manaflow-ai/cmux-widget-clock my-widget
git clone https://github.com/manaflow-ai/cmux     # the extension SDK, next to it (../cmux)
cd my-widget && cmux ext dev                       # live in a running cmux, reloaded on save
bun test                                           # the tests resolve the SDK from ../cmux
```

Change `id` and `name` in `cmux-app.json` before you share it. `dist/main.js` is the built widget;
rebuild it with `bun ../cmux/cmux-tui/crates/cmux-app-host/tools/pack.ts .`.


A `cmux.widget/1` store app. The person installs it from the App Store with one click (it is not
installed by default: no `BUNDLED` marker; the `STORE` marker lists it), then adds it to the New Tab
page or the sidebar from Edit Widgets. It shows something good with zero setup.

| Size | What it shows |
| --- | --- |
| small (1x1) | the weekday in red over the date, the time big and heavy at the bottom |
| medium (2x1) | the small layout, and two cities: their time, how far they are from you, Tomorrow or Yesterday |
| large (2x2) | a lock-screen style date line ("Monday, November 16") over the time, then four cities |
| small, analog | a dial with twelve ticks and the hour and minute hands, the day under it |
| medium, analog | a world-clock row: a dial for here and three cities, each named underneath |

## Settings

The tile's own settings (from the widget host), else the app's (Settings > Apps > Clock):

- `hourCycle`: `system` (default: the locale's clock; the sandbox has no `Intl`, so by region),
  `12h` or `24h`.
- `cities`: IANA zone names separated by commas. Empty (default) means Los Angeles, New York,
  London, Tokyo and Sydney, skipping the ones at your own offset so you never see your time twice;
  a list you write is shown as written. An unknown name shows "Unknown zone" in its row.
- `face`: `digital` (default) or `analog` (small and medium).
- `font`: `rounded` (default, the lock-screen look), `system`, `serif`, `mono` or `condensed`, for the time.
- `weight`: `thin`, `light`, `regular`, `medium`, `semibold`, `bold` or `heavy` (default).
- `color`: `text` (default, the theme's), `accent`, `muted`, `glass` (the theme's text, translucent)
  or a `#RRGGBB` color. Anything else reads as `text`.
- `date`: the date over the time (default on).

The look settings carry the options of the New Tab clock this widget replaced (cmux#44388).

## Cost

Nothing between minutes. The manifest asks the host for `refresh.everySeconds: 60`; the widget host
ticks the mount context's `now()` on the minute only while the tile is shown, and the widget reads
it. On a host without `now()` (an older cmux) it arms one one-shot timer to just after the next
minute, which arms the next and is cleared with the view. A tick sends one scene update: the time's
Text node (and the cities' times and the hands). No reads, no network, no scopes: zone rules come
from `cmux.time.zone`. The person's own zone and locale come from the widget context (`timeZone`,
`locale`): their offset is `cmux.time.zone(timeZone, now())`, and the analog row names their city.

## Files

- `src/clock.ts`: the model (times, dates, cities, hand angles), no view and no host.
- `src/main.ts`: `renderClock`, the scene for each size and face, public SDK only.
- `test/`: `bun test ./test` (the model, and the packed app in the fake host).
- `preview/build.ts` writes `preview/scenes.json`, the scene ops for each gallery variant at 9:41 AM
  on Monday, November 16, 2026 in Los Angeles; `--check` verifies it. The webviews gallery entry
  `apps.clock-widget` renders them with the real scene renderer.
- `screenshots/`: captures of that gallery entry, listed in the store.
- `STORE` marks it for the App Store. The store package is build output, generated from this folder at build time; nothing of it is committed here.

## Platform gaps

- Font design (rounded, serif) needs the scene's `fontDesign`; on a host without it the
  time draws in the system font (mono uses `monospaced()`). `condensed` has no scene font width yet: system.
- Glass is a translucent text color; the New Tab clock's soft rim of light needs a scene text shadow.
- `HStack` has no baseline alignment: AM/PM sits centered beside the big time, not on its baseline.
- Shapes are square (`size` only; `frame` is ignored on `Circle`/`Capsule`/`Rectangle`): clock hands
  are drawn as rotated stacks with a background instead.
- No seconds: a seconds hand or `:SS` would need a host-drawn ticking text or rotation, which the scene vocabulary does not have; per-second scene updates
  would spend the refresh budget.
- English only: the sandbox has no `Intl` to name weekdays and months in the person's language.
