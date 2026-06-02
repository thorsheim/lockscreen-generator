# CLAUDE.md

Orientation notes for future Claude sessions working on this project.

## What it is

A static, single-page **Lockscreen Generator**. You upload a photo, crop/zoom it to a
phone screen aspect ratio, overlay your contact details (and optionally a scannable QR
code), and download a full-resolution image to set as a phone lockscreen — so a lost phone
shows who it belongs to.

No build step. `index.html` is opened directly (`file://`) or served via
`python3 -m http.server`. Everything — HTML, CSS, JS, and the QR encoder — lives in that
one file. Nothing is uploaded anywhere; all processing is client-side in `<canvas>`.

## Files

- `index.html` — the entire app:
  - `:root` / `[data-theme="light"]` — CSS variables for dark/light themes.
  - Static HTML: header → controls column (7 cards: Image, Screen size, Text,
    Text style, Text box, QR code, Export) → sticky preview "stage" with the `<canvas>`.
  - **First `<script>`** — the bundled QR encoder (qrcode-generator v1.4.4, Kazuhiko
    Arase, MIT). Inlined verbatim and exposed as `window.qrcode`. Don't hand-edit it; to
    update, re-fetch the upstream file and re-splice (see "QR encoder" below).
  - **Second `<script>`** — the app itself, one IIFE. Key pieces: `FONTS` list, the
    `state` object + `defaults`, `loadState`/`saveState` (localStorage), text model
    (`textLines`, `buildVCard`, `getQrModel`), image transform
    (`baseScale`/`clampView`/`imageRect`), the shared `draw(ctx, W, H)`, pointer/wheel/
    pinch handlers, control wiring, `doExport`, `initUI`.
- `README.md` — end-user usage.

## The one idea that makes it work: normalized coordinates + one `draw`

Everything that has a position or size is stored as a **fraction (0–1) of the frame**, never
in pixels: image pan (`view.offX/offY`), zoom (`view.scale`, a multiplier on the cover-fit
base scale), text box center (`box.cx/cy`), padding/corner radius (`box.padRel/radiusRel`),
font size (`text.fontSizeRel` = fraction of frame **height**), and QR size/position
(`qr.sizeRel`, `qr.cx/cy`).

Because of that, a **single `draw(ctx, W, H)`** function renders identically at any
resolution. The on-screen preview calls it at a scaled-down canvas size (`render()`); export
calls the *same* function on an offscreen canvas at the device's real pixel resolution
(`doExport`). Preview and download are therefore pixel-faithful. **If you add any new
overlay/size, store it normalized and render it inside `draw` — don't special-case export.**

The uploaded image is held in the module-level `img` variable and is **not** persisted
(localStorage holds only settings). Reloading keeps your text/colors/format but you re-pick
the photo.

## Key behaviours

- **Cover-fit + clamp**: the image always covers the frame (`baseScale` = max ratio).
  `clampView` prevents panning/zooming past the edges, so there are never empty borders.
  Wheel zoom is cursor-anchored; two-finger pinch zooms via the `pointers` Map.
- **Hit-testing priority** on pointer-down: QR box → text box → otherwise pan the image.
  `draw` records `lastBoxRect`/`lastQrRect` each frame for this. Dragging the box or QR sets
  `layout="custom"`.
- **Layout presets** (`top`/`center`/`bottom`) only set the vertical anchor `cy` for both
  the box and QR; horizontal stays wherever it was. Dragging switches to `custom`.
- **Text modes**: `contact` builds lines from prefix/name/phone/email/address (blank fields
  skipped); `free` is a verbatim textarea (line breaks kept). Both are word-wrapped to 86%
  of frame width inside `draw`.
- **QR**: `vcard` mode builds a VCARD 3.0 string from the contact fields; `text` mode uses a
  raw string/URL. `getQrModel` caches by payload and overrides `qrcode.stringToBytes` with a
  `TextEncoder` so UTF-8 (accents, etc.) encodes correctly. If the payload is empty or too
  long for a QR, a labelled placeholder square is drawn instead (still draggable).
- **Export**: PNG (lossless) or JPEG (quality slider). Filename is
  `lockscreen-<W>x<H>.<ext>`. Object URL is revoked after the click.
- **Persistence**: `localStorage` keys `lockscreen-state-v1` (settings, debounced 250 ms)
  and `lockscreen-theme`. Reads/writes are wrapped in try/catch for private-mode browsers.
  Bump the `-v1` suffix if the state shape changes incompatibly.

## Gotchas

- **No runtime `innerHTML`** — a project hook blocks it. All static structure is authored
  HTML markup (fine); anything dynamic uses `document.createElement` + `textContent` (e.g.
  the font dropdown is built that way, and messages use `el.textContent`). Don't introduce
  `innerHTML`.
- **QR encoder is inlined on purpose.** The sibling `word-pin-converter` documents that
  Chrome silently fails to load `file://` scripts from subdirectories. Inlining keeps the
  tool single-file and fully offline. Don't move it to a separate `<script src>`.
- **Web fonts need the network.** The Google Fonts `<link>` (Inter, Poppins, Montserrat,
  Nunito, Oswald, Bebas Neue, Playfair Display) only loads when online; offline, those
  dropdown entries fall back to their generic stacks and the rest of the tool (including QR)
  still works. `document.fonts.ready` triggers a re-render once fonts arrive.
- **Preview vs export size**: the preview canvas is sized to fit the viewport
  (`PREVIEW_MAX_H`), the export canvas to `state.format.{w,h}`. Never assume preview pixels
  equal export pixels — always go through normalized coords.

## Common changes

- **Add a device preset**: add an `<option value="WIDTHxHEIGHT">` to the right `<optgroup>`
  in the Screen-size card. The `format` change handler parses `WxH` automatically.
- **Add a font**: add `{ name, stack }` to the `FONTS` array, and (if it's a web font) add
  its family to the Google Fonts `<link>` href. The dropdown is populated from `FONTS`.
- **Add a new styling control**: add the input to the relevant card, store its value
  normalized on `state`, wire an `input`/`change` listener that updates state +
  `render()` + `saveState()`, render it inside `draw`, and set its initial value in
  `initUI`.

## Verify after changes

`cd lockscreen-generator && python3 -m http.server 8000`, open `http://localhost:8000`,
upload a photo, exercise the cards, drag the box/QR, and download both a PNG and a JPEG —
confirm the file dimensions equal the selected device resolution and the image matches the
preview. A headless Playwright smoke test was used during initial development (uploads an
image, fills contact fields, zooms, enables QR, switches format, exports PNG+JPEG, checks
the PNG IHDR dimensions and localStorage) — replicate that flow when touching `draw`,
export, or the QR path.
