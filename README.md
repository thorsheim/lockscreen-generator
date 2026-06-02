# Lockscreen Generator

A tiny, private, browser-based tool to turn any photo into a phone lockscreen that shows
**who the phone belongs to** — so a lost or misplaced phone can find its way back to you.

Upload a picture, crop it to your phone's screen, add your contact details (and optionally a
scannable QR code), and download a full-resolution image to set as your lockscreen
wallpaper.

Everything runs locally in your browser. **No image is ever uploaded** — there is no server.

## Use it

Open `index.html` in any modern browser. Either:

- **Double-click** `index.html`, or
- Serve it (recommended, and required for the optional web fonts):

  ```sh
  cd lockscreen-generator
  python3 -m http.server 8000
  # then open http://localhost:8000
  ```

## How to make a lockscreen

1. **Image** — click the box (or drag a photo in) to upload. Drag on the preview to pan;
   scroll or pinch to zoom. "Re-center" resets the crop.
2. **Screen size** — pick your phone (generic 9:16 / 9:19.5 / …, common iPhone and Android
   resolutions) or enter a custom width × height.
3. **Text** — choose **Contact** (name, phone, email, address — blank fields are hidden) or
   **Free text** (type anything; line breaks are kept).
4. **Text style** — font, size, color, opacity, alignment, bold, and a drop shadow for
   legibility.
5. **Text box** — toggle the background panel behind your text and set its color, opacity,
   corner rounding, and padding. Use the Top / Center / Bottom buttons, or just **drag the
   box on the preview** to place it anywhere.
6. **QR code** *(optional)* — add a scannable code containing your contact card (vCard) or
   any custom text/URL. Drag it to position it; adjust size and colors.
7. **Export** — choose PNG (sharpest) or JPEG (smaller; quality slider), then **Download**.
   The image is saved at your phone's exact pixel resolution as `lockscreen-<W>x<H>.png`.

Set the downloaded image as your phone's lockscreen wallpaper (not the home screen) so your
details are visible without unlocking.

## Notes

- Your settings (text, colors, fonts, device, layout) are remembered in the browser between
  visits. The photo itself is not stored — re-pick it next time.
- Works fully **offline**. Extra display fonts are pulled from Google Fonts when online; with
  no connection they fall back to built-in fonts and everything else still works.
- Light and dark themes — toggle in the top-right.

## Privacy

100% client-side. The photo and your contact details never leave your device.
