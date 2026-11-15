# 🎄 Advent Calendar — Customization Guide

🇫🇷 [Version française](README.fr.md)

This calendar is **a single self-contained `index.html` file**: HTML, CSS, JavaScript and every image (base64-encoded) live in the same file. No external dependency, no server: you can open it locally, host it anywhere (GitHub Pages, an intranet…) or publish it as an artifact.

To adapt it to another calendar (different theme, year or content), you change four things:

1. the **texts** of the 24 doors,
2. the **images** (starting tower, revealed picture, decorations),
3. the **date / storage key**,
4. optionally the **behavior** (door order, animations, languages).

---

## 1. How it works

- A background image (the `tower`) is repeated 6 times: it forms a grid of **6 columns × 4 rows = 24 doors**.
- Each door shows a day number. When opened, the **matching piece** of a large image (the "scene") appears over the door. Once all 24 doors are open, the scene is fully visible.
- Clicking an open door (or one that has just opened) shows a **window** with the title, the text, a source link, and a copy button if the door contains code.
- Opened doors are remembered in the browser (`localStorage`); the language is stored in a **cookie**.
- A door only opens **on its own day** (December 1 to 24 of the configured year).

---

## 2. Content (the 24 door texts)

In the `<script>`, find the arrays:

```js
const TIPS_FR = [ {tag, title, text, src, url, code?}, ... ];   // 24 entries
const TIPS_EN = [ {tag, title, text, src}, ... ];               // 24 entries
const TIPS_URL = TIPS_FR.map(x => x.url);                       // URLs come from the French array
```

Entry #1 of the array is **day 1**, entry #24 is day 24 (array order = day order, not position on the grid).

| Field   | Where | Purpose |
|---------|-------|---------|
| `tag`   | FR + EN | Label shown next to "Day N" (here `year · category`). |
| `title` | FR + EN | Window title. |
| `text`  | FR + EN | Main text (plain text, no HTML). |
| `src`   | FR + EN | Label of the source button (e.g. `Wikipedia — CICS`). |
| `url`   | FR only | Link of the source button, opened in a new tab. |
| `code`  | FR only, optional | If present, shows a code block and a "Copy code" button. |

Tips:
- Use **verified** links, ideally primary sources. The link is also shown as a tooltip when hovering the button.
- If you only need one language, remove `TIPS_EN` and the language logic (see §6), or simply duplicate the French content.
- In this calendar, door 24 is a closing card ("Merry Christmas"): it is the one that triggers the sleigh animation (see §7).

---

## 3. Images

All images are **data URIs** (`data:image/…;base64,…`) inside the file. Here is the inventory, in order of appearance, so you can find them:

| # | Format | Role | Where to find it |
|---|--------|------|------------------|
| 0 | WebP | Title panel (wood, garland, bow) | `.panel{ background:url(...) }` |
| 1 | JPEG | **Starting tower** (the "mainframe" repeated 6 times) | `.board{ background:... url(...) }` |
| 2–4 | WebP | Top snow: left end / repeatable pattern / right end | `.snowcap .sl`, `.sm`, `.sr` |
| 5 | GIF | Santa in his sleigh (animation) | `.sleigh i, .patrol i` |
| 6 | JPEG | **Revealed scene** (final picture) | `.tile{ background-image:url(...) }` |
| 7 | WebP | Frame of the tip window | `.modal::before{ border-image:url(...) }` |

### 3.1 Changing the revealed scene (final picture)

This is the most common customization.

- **Ideal format: 3:2** (e.g. 1536×1024). The board is set to `aspect-ratio: 3/2` (see `.board`). An image with another ratio will be stretched: either crop it to 3:2, or change the `.board` `aspect-ratio` (and accept that the starting tower will be stretched accordingly).
- Replace URI #6 with the new image (JPEG recommended, quality 85–90).
- The scene is split into 6×4 tiles through `background-size:600% 400%`. Nothing else to configure.

Python script to replace the n-th image of the file:

```python
import re, base64
s = open('index.html', encoding='utf-8').read()
new = 'data:image/jpeg;base64,' + base64.b64encode(open('scene.jpg','rb').read()).decode()
matches = list(re.finditer(r'data:image/\w+;base64,[A-Za-z0-9+/=]+', s))
m = matches[6]                      # index from the table above
s = s[:m.start()] + new + s[m.end():]
open('index.html', 'w', encoding='utf-8').write(s)
```

### 3.2 Changing the starting tower (the image that hides the scene)

- Image #1 is **a single tower** (here 254×790 px) repeated 6 times horizontally: `repeat-x 0 0 / calc(100% / 6) 100%`.
- It must contain **4 vertical panels** (one per row). If your image has a different geometry, adjust the JS constants:

```js
const TW=254, TH=790;          // width / height of the tower image (px)
const PX=29,  PW=183, PH=184;  // X position, width and height of a panel (px)
const PYS=[6,203,399,595];     // Y position of each of the 4 panels
```

  These values place the clickable doors **exactly** on the panels (they are converted to percentages). Measure them in an image editor on your own tower.
- Alternative: a single background image (no repetition) — replace `repeat-x … / calc(100% / 6) 100%` with `no-repeat center / 100% 100%` and recompute the positions.

### 3.3 Changing the decorations (title, snow, window frame)

- **Title panel** (#0): original ratio 1400×406 (`aspect-ratio:1400/406`). The text is centered in the light area through the `.panel` `padding` (in `cqw`, proportional to the width). If you change the image, adjust `padding:10.5cqw 9cqw 4.6cqw 10cqw` (top, right, bottom, left) so the title stays in the usable area.
- **Snow** (#2–4): three pieces of the **same height** and same band thickness at the joins. The middle pattern is repeated (`repeat-x`). The ratios are written in the CSS (`aspect-ratio:1399/684` and `1075/684`): update them if you change the images. Position and height: `.snowcap{ top:-3.8cqw; height:12cqw; left/right:-2.2cqw }`.
- **Window frame** (#7): used as a `border-image` cut into 3 bands (top 330 px, stretchable middle, bottom 350 px in the original image). If you change the image, update the `330` / `350` values in `.modal::before` and the `.modal` `padding` to keep the text in the light area.
- To **remove** a decoration: delete the matching element (`.snowcap`, `.panel`…) and its CSS.

Tip: for any image whose background must be transparent, cut it out first (PNG/WebP with an alpha channel) before encoding; the current decorations are transparent WebP files.

---

## 4. Dates and saving

```js
const YEAR = 2026;                  // calendar year
const KEY  = "avent-dino-" + YEAR;  // storage key in localStorage
```

- Doors only open from **December 1 of `YEAR`**: door N opens on December N. After December 25, all doors can be opened.
- For another month or another rhythm, edit the `currentDay()` function (it returns the last openable day, from 0 to 24):

```js
function currentDay(){
  const now = new Date();
  if (now < new Date(YEAR, 11, 1))  return 0;   // 11 = December (months start at 0)
  if (now >= new Date(YEAR, 11, 25)) return 24;
  return Math.min(24, now.getDate());
}
```

- If you reuse the same file year after year, just change `YEAR`: the storage key changes too, so all doors start closed again.
- The "Patience! Door N opens on December N" message uses `STR.fr.month` / `STR.en.month`.

---

## 5. Door layout

```js
const ORDER = [7,19,2,14,23,10, 17,4,12,24,1,21, 9,15,6,20,13,3, 22,11,18,5,16,8];
const COLS  = 6;
```

- `ORDER` lists the day numbers **row by row**, left to right (4 rows of 6). Shuffle as you like: each number from 1 to 24 must appear **exactly once**.
- For another grid (e.g. 4×6), change `COLS`, the number of rows (`row/4`, `25%`, `row/3` in the tile creation code and in `.tile`), `PYS` and `background-size:600% 400%`.

---

## 6. Languages and interface texts

The interface labels live in the `STR` object:

```js
const STR = {
  fr: { title, h1, sub, count, reset, foot, day, src, month, wait, testOn, testOff, resetT, copy, copied, copyFail, open, again, close, test },
  en: { … }
};
```

- `h1`: calendar title; `sub`: subtitle; `foot`: footer; `day`: prefix ("Jour" / "Day"); `src`: prefix of the source button (empty by default → only the source name is shown).
- **Adding a language**: (1) add a key in `STR` and a `TIPS_XX` array, (2) adapt the `tips(day)` function to return it, (3) add a button in the `<span class="lang">` block and its `onclick`, (4) complete `applyLang()`.
- **Persistence**: the language is stored in the `avent-lang` cookie (1 year) with a fallback on `localStorage`. On first visit the browser language is used (French, otherwise English).
- **Removing the language switch**: delete the `.lang` block, force `LANG="fr"` and remove the cookie reading.

---

## 6 bis. General appearance (CSS)

Color variables at the top of the stylesheet:

```css
:root{ --night:#0b1a45; --night2:#16307a; --gold:#ffd24a; --red:#c9202e; --green:#1f7a3d; --snow:#f4f8ff; }
```

- Page background: `body{ background: radial-gradient(...) }`.
- Frame around the board: `.frame{ border:6px solid #7a4a1d; box-shadow:0 0 0 3px var(--gold) … }`.
- Numbered badge on the doors: `.door .n` (font size, width, colors).
- Source button: `.modal .srcbtn` (color `#1f7a3d`, raised shadow).
- Title: `h1{ color:#8f1420; font-size:min(6cqw,3rem) }`; subtitle: `.sub`.
- Falling snow: the `/* neige */` block at the bottom of the script (number of flakes `40`, durations, sizes). Automatically disabled when the device asks for reduced motion.

---

## 7. Animations and interactions

### Santa's sleigh (after door 24 is opened)

```css
.patrol{ width:min(58vw,420px); animation:patrolX 17s linear infinite; z-index:30; }
@keyframes patrolWave{ … }      /* vertical sway (sinusoidal) */
```

- **Trigger**: door `24` opened (`opened.has(24)` in `refresh()`). For another trigger, change this test (e.g. another door, or all doors opened).
- **Size**: `width` of `.patrol`. **Speed**: the `17s` duration (smaller = faster).
- **Sway**: `patrolWave` (amplitude `7vh`, tilt `-6deg → 8deg`, duration `3.2s`).
- **Randomness**: on every new pass, the `randPatrol()` function picks a starting height and an ending height (between 2% and 78% of the screen height). The direction is always left to right. Change the `lo` / `hi` bounds.
- **Image**: replace the GIF (#5). Adjust `aspect-ratio:724/241` to the new GIF's ratio.
- **Disable**: delete the `<div class="patrol">` element and the `$("patrol").classList.toggle(...)` line in `refresh()`.
- The file still contains a `.sleigh` element and a `flySleigh()` function from an earlier version (a one-off diagonal flight); they are no longer called and can be removed with no effect.

### Test mode (hidden)

An invisible button sits on the **green gift** of the title panel. Clicking it toggles test mode: all doors become openable without waiting for the date, with a golden halo around the gift. The "Reset" button clears the opened doors (and stops the sleigh).

- Button position: `.giftbtn{ left:10.5%; top:75%; width:8%; height:18% }` (as a percentage of the panel). **Re-adjust it** if you change the title panel.
- To disable it completely: delete the `<button class="giftbtn">` and the `$("testBtn").onclick` handler.

### Tip window

- Size: `--W:min(760px,94vw)` in `.modal` (maximum width).
- Closing: cross button, click outside, Escape key.
- Code copy button: only appears if the door has a `code` field.

---

## 8. Publishing / hosting

- **Locally**: double-click `index.html`.
- **Static hosting** (GitHub Pages, Netlify, web server): upload `index.html` as is.
- **Cookies**: the language cookie may be ignored if the page is shown in a frame that blocks cookies; a `localStorage` fallback is built in, and otherwise the browser language is used.
- **Size**: the file weighs around 2 MB because of the images. To slim it down, recompress the images (WebP quality ~80, JPEG quality ~80, realistic display size).

---

## 9. Checklist for creating a new calendar

1. Duplicate `index.html` under another name.
2. Change `YEAR` and, if needed, `KEY`.
3. Replace the 24 entries of `TIPS_FR` (and `TIPS_EN`): `tag`, `title`, `text`, `src`, `url`.
4. Change the `STR` texts (`title`, `h1`, `sub`, `foot`).
5. Replace the **scene** (#6, 3:2 ratio) and the **tower** (#1); adjust `TW/TH/PX/PW/PH/PYS` if the tower changes.
6. (Optional) Replace the decorations: title panel, snow, window frame, sleigh GIF.
7. Re-adjust the hidden test-mode button (`.giftbtn`) on the new element of the title panel.
8. Test: enable test mode, open all doors, check the source links, switch the language, resize the window (mobile and desktop).
9. Reset the calendar before publishing, then turn test mode off.

---

## 10. License

- **Code** (HTML, CSS, JavaScript): **MIT** license — see the `LICENSE` file.
- **Content created by the author** (door texts, original illustrations): **CC BY 4.0** license — see the `LICENSE-CONTENT` file. You may reuse and modify it, including commercially, as long as you credit the author ("zarafa-dev-io") and indicate any changes.
- **IBM trademarks and visuals**: the z17 image (the mainframe tower), the IBM logo and the "Z" logo belong to IBM. IBM and z17 are trademarks of IBM. This project is independent and is not affiliated with, endorsed by or sponsored by IBM; these elements are not covered by the licenses above.
- **Not covered**: any third-party element (for example the IBM mainframe photograph and IBM marks, or any image / animation whose rights you do not hold) is **not** covered by these licenses. Replace them with your own visuals before redistributing the calendar.

Copyright (c) 2026 zarafa-dev-io
