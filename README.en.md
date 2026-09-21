# Card Design Generator

[中文说明](./README.md)

A single-file, dependency-free tool for laying out card artwork to the ISO 7810 ID-1 size. Drop in an image, place a logo, export a static PNG.

Live demo: https://susie-meow.github.io/card-design-generator/

## What it does

- Upload a background image — it is scaled and centred to fill the ID-1 card ratio
- Crop with the ratio locked at 1.586 : 1, so however you cut it you still get a valid card
- Add a reference image — a second picture run through the same fitting and cropping, laid over the card and beneath the logos at an opacity you control, for alignment only; it never reaches the export
- Add logos — click to pick several at once, or drag a batch straight onto the card
- Move, scale, rotate, and fade each layer, and key out a flat background colour
- Nudge the selected logo with the arrow keys — one export pixel at a time, accelerating while you hold
- Give a logo an outline so it stays readable when its colour is close to the background; let the tool pick a contrasting colour or choose your own
- Export a 1036 × 640 PNG — about 635 DPI at card width, enough to print
- Switch the interface between English and Chinese

No logos or templates ship with the project. Every asset you place is one you upload yourself.

## Getting started

Open `index.html` in a browser. That is the entire install — no dependencies, no build step.

If you would rather serve it over HTTP:

```
python3 -m http.server 8000
```

then visit http://localhost:8000

`package.json` only records the card spec the app renders against (standard, dimensions in millimetres, ratio, export resolution). It is not a dependency manifest and nothing needs installing.

## How to use it

1. **Accept the terms.** They appear once on load; the tool stays locked until you agree.
2. **Add your background image** to the Card image panel on the right, or press *Choose image* in the middle of the empty card. It is fitted to the card ratio straight away.
3. **Crop** if you want to reframe it. Drag inside the selection to reposition, drag a corner to resize — the ratio stays locked.
4. **Add a reference image** if you want something to align against. It uses the same fitting and the same crop dialog, and sits between the card and the logos. See *About the reference image* below.
5. **Add logos** to the Logo panel, or drag them anywhere onto the window. If no background is set yet, a dropped image becomes the background instead.
6. **Click a logo on the card** to select it. The Adjust panel then drives that layer:

   | Control | Range | Notes |
   | --- | --- | --- |
   | Size | 5 – 300% | Scroll over the card to scale the selected logo directly |
   | Opacity | 0 – 100% | |
   | Rotation | −180° to 180° | |
   | Remove background colour | on / off | Keys out the flat colour averaged from the four corners |
   | Tolerance | 1 – 120 | Widens or narrows what counts as background. Default 24 |
   | Outline | on / off | An outer stroke — see below |

   Drag a logo on the card to move it. Use the arrows in the selection bar to reorder layers, the trash icon to delete one, or press Delete / Backspace.
7. **Export.** *Export PNG* stays greyed out until you add a background image; once active it writes a `card-design.png` to your downloads.
8. **Decide on corners.** *Show 3.18 mm corner radius* is on by default and applies to the export. Uncheck it for a square, bleed-friendly file.

Image handling runs entirely in your browser. Nothing you upload leaves your machine.

### Keyboard

Every control is reachable with Tab, including the three drop zones and the layer rows. Enter or Space activates whichever of those has focus, and Delete / Backspace removes the selected logo.

Arrow keys nudge the selected logo. A single press moves it one export-canvas pixel — enough to land on exact pixel values — and holding a key repeats with a shrinking interval and a step that doubles up to 16 px, so you can cross the whole card without letting go. The repeat curve is implemented in the app rather than relying on the operating system's key repeat, so it feels the same on every platform.

## Dimensions

ISO 7810 ID-1 is the spec behind bank and transit cards: 85.60 × 53.98 mm, a ratio of about 1.586 : 1, with 3.18 mm corners.

The export canvas is fixed at 1036 × 640 px, which works out to roughly 635 DPI across the 85.60 mm width. The preview carries those corners and the export matches.

## About the reference image

The reference is a second image you bring in purely to line things up against — a previous revision, a mockup, or a card you are trying to match. It goes through exactly the same fitting and cropping as the card image, with the ratio locked to 1.586 : 1 and the same crop dialog (the dialog title switches to *Crop the reference image* so you always know which one you are cutting).

Layer order is **card → reference → logos**, so the reference reads like a sheet of tracing paper laid over the artwork, under anything you place on top.

It exists on the preview canvas only. Export re-renders to an offscreen canvas without it, so the reference and the selection outlines never appear in the PNG.

| Control | Range | Notes |
| --- | --- | --- |
| Show reference | on / off | Hides it without discarding it |
| Opacity | 0 – 100% | Default 40% |

## About outlines

The outline grows outward from the logo silhouette rather than being painted over it. The logo's alpha channel is treated as a mask, a distance field is computed from every pixel to the nearest opaque one, and colour is filled only in the transparent space beyond it — up to the width you set. A few things follow from that:

- Cut-outs survive. A hollow ring keeps its hole instead of filling in.
- The artwork itself is untouched — same shape, proportions, and crispness.
- Edges get a 1 px coverage ramp, so they stay smooth instead of stepping, even zoomed in.
- The outline follows position, size, rotation, and opacity; fade the logo and the outline fades with it.

| Parameter | Range | Notes |
| --- | --- | --- |
| Outline | on / off | On by default. Turning it off removes the stroke entirely. |
| Outline width | 0 – 40 px | Measured in export-canvas pixels (1036 × 640); 0 means none. The starting value is 3 – 8 px depending on logo size. |
| Outline colour | picker + Auto contrast | With Auto contrast on, the colour comes from the average brightness of the logo's opaque pixels: light logos get #111A2B, dark ones get pure white. The swatch locks and shows what is actually being used. Uncheck it to pick freely. |

Because the unit is the export canvas, it is WYSIWYG — whatever width you see in the preview is what lands in the PNG.

## Notes

- Image processing happens locally in your browser; nothing is sent to a server
- The terms surface once on load and have to be accepted before you continue
- Output is visual artwork only. It is not a payment or transit card issued by anyone, and it must not be used to counterfeit, impersonate, or produce something that could pass as a working credential. Printing design mockups, art cards, collector cards, or board game pieces is unaffected
- This project has no official partnership, licence, affiliation, or endorsement with Apple, Apple Pay, Visa, Mastercard, or any card issuer. Third-party trademarks belong to their respective owners

## Implementation

One `index.html` using plain Canvas 2D — no framework, no dependencies, no build step. Fonts come from Google Fonts and fall back to system faces if they cannot be reached, which does not affect anything functionally.

The outline runs a Felzenszwalb exact Euclidean distance transform over the logo's alpha channel, so the stroke is a true outward dilation with isotropic edges rather than a blurred shadow.

| File | Purpose |
| --- | --- |
| `index.html` | The whole application — markup, styles, and script |
| `README.md` | Documentation in Chinese |
| `README.en.md` | Documentation in English |
| `package.json` | Card spec metadata only |
