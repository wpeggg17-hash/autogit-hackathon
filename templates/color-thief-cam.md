---
title: Color Thief Cam
app_type: color-thief-cam
wallet: 0xCD3f91c0114890AA15F68b385DC2C7103Be012D3
---

Point a camera at anything. The page samples the frame, finds the six most representative colors in the image through a median cut palette, and prints them as a row of swatches with hex codes underneath. Tap a swatch and the hex copies to the clipboard. Tap shoot and the whole frame plus its palette saves as a card the visitor can keep. The page never sends frames anywhere. Everything happens on the device.

# camera, palette, card

### viewfinder

A studio dark page, almost coffee black. Soft warm white for body type. A single magenta accent reserved for the shoot button and the small recording dot above the viewfinder. Dim graphite for the muted labels. Anton Display at 36px for the single top header reading camera, palette, card on one line, slightly tracked, lowercase, no period. Sora at 16px 400 for body and 13px 500 for the small section labels. DM Mono at 12px for the hex codes and the small image dimensions in the corner of saved cards. Three typefaces, three roles, no overlap. The layout is one page, the viewfinder fills the top half on desktop and the full width on phones, the saved cards stack below.

### feed

The camera feed is a single video element with autoplay, muted, playsInline, srcObject set to a MediaStream from navigator.mediaDevices.getUserMedia. The page asks for camera permission on the first tap of a small text button below the empty viewfinder reading allow the camera. The constraints requested are below.

```
{
  audio: false,
  video: {
    facingMode: { ideal: 'environment' },
    width:      { ideal: 1920 },
    height:     { ideal: 1080 },
    frameRate:  { ideal: 30 }
  }
}
```

If the environment facing camera is unavailable the browser falls back to whatever camera the visitor has. If neither is available, the viewfinder shows a single text button reading pick an image file, which opens a file picker through showOpenFilePicker (with anchor based fallback when the api is missing) and treats the picked image as a still frame. The rest of the page behaves identically. To the right of the viewfinder a small DM Mono 12px line shows the live frame resolution, like 1920 by 1080.

### palette

Below the viewfinder a horizontal row of six swatches, each 80 by 96 pixels on desktop and 56 by 72 on phones. Each swatch is a single colored block with the hex code in DM Mono 12px directly under it, soft warm white text on a strip that picks up the swatch color at twenty percent opacity. Tapping a swatch writes the hex to the clipboard with a small toast saying copied to clipboard for 1200ms. The swatches update live as the camera frame changes, throttled to one update every 300ms to keep the page calm, with a small graceful crossfade between the old and the new colors over 180ms so the row does not flicker. A small DM Mono 11px line below the swatch row shows the current update rate as updated 3.3 times per second.

The palette is computed by a median cut algorithm running on a small downsampled version of the current frame. The frame is drawn into an OffscreenCanvas at 96 by 54 (sixteen by nine aspect, low resolution enough that the algorithm is fast and stable). The pixels are read as a Uint8ClampedArray of RGBA values. The algorithm builds a box around all pixels in RGB space, finds the longest axis (red, green, or blue), sorts the pixels along that axis, splits the box at the median, and recurses on each half until the desired number of leaves is reached (six in this case). The average color of each leaf is the final swatch.

```
{ leaf:   3,
  count:  982,
  r:      214,
  g:      78,
  b:      62,
  hex:    '#D64E3E' }
```

The algorithm sorts the leaves by pixel count descending so the most dominant color is leftmost. The whole pass runs in around two milliseconds on a recent laptop, well inside the throttled budget.

### shoot

A single round button in the center under the swatches, 64 pixels round, magenta with no border, with a smaller dark grey inner circle in the middle. Pressing it triggers the shoot action. The page captures the current frame at the camera's native resolution by drawing the video into a separate canvas, then captures the current palette as it stands, and saves a card containing both. A small magenta dot blinks twice above the viewfinder on each successful shoot.

### cards

Below the shoot button, a vertical list of cards, most recent on top, separated by 16px of vertical space. Each card holds three regions.

The first region is the saved image, scaled to the card width with the original aspect preserved. The image is drawn from a single blob the App stored when shooting.

The second region is the palette as a thin band of six colored stripes across the bottom of the image, each stripe an equal width sixth, with the hex codes printed in DM Mono 10px directly under each stripe.

The third region is the metadata strip, a single DM Mono 11px line below the band showing the date the card was shot in long form like Sat 29 May 14:22, the camera that took the shot (the deviceId resolved to the visitor's chosen camera label when the browser exposes it), and the resolution of the saved image. To the right of the metadata strip a small text button in Sora 12px magenta reading remove this card with a confirm prompt.

The card is the smallest unit the page knows about. Tapping the image inside a card opens a small lightbox that fills the viewport with the saved image, dim graphite background, a single close text button in the corner. Tapping anywhere outside the image closes the lightbox.

### storage

Saved cards live in IndexedDB rather than localStorage so the page can handle several full resolution images without bumping the quota. The store is named cards with a keyPath of id and a single index on shotAt for the descending sort. Each card stores the id, the shotAt iso timestamp, the camera label, the resolution as a two number tuple, the image blob, and the six swatches as an array of objects with r, g, b, hex, and count fields. The page uses structuredClone for any value it duplicates between memory and disk, and uses createObjectURL with revokeObjectURL discipline to render the image blobs without leaking memory.

A small text button below the card list in 12px Sora reads remove every card, with a two stage confirm, which clears the IndexedDB store after the visitor confirms.

If IndexedDB is unavailable the cards live in memory and a small italic 11px line at the bottom of the page reads these cards are not being kept on this device.

### keys

Pressing the space key at any time triggers the shoot button. Pressing the escape key inside the lightbox closes it. Pressing the c key copies the leftmost swatch hex to the clipboard. Pressing one through six copies the matching swatch hex.

### edges

A camera that suddenly stops (the visitor unplugged the webcam) is detected through the ended event on the MediaStreamTrack and the viewfinder shows a small italic 14px line reading the camera went away with a small text button to ask again. A camera permission denied at first use shows the same line plus a small italic 13px hint reading you can still use the page with a picked image file.

A swatch whose count is very small (less than one percent of pixels) is dropped and the algorithm tries to produce one more leaf to fill the six slot row. If the image has fewer than six distinct colors (rare, but possible for a deliberately uniform frame), the empty slots show as dim graphite with a small en placeholder dash.

The page does not zoom the camera, does not flash a torch, does not toggle a filter. The viewfinder is the camera's native view. The whole page is the smallest tool that does the thing.

### build

The build is a single Vite project. React 18 with TypeScript. Tailwind CSS for layout and color, custom palette in tailwind.config.ts adding studio, warm white, warm white muted, magenta, graphite. Vite as the build tool. State is plain useState and one useReducer for the camera state machine (idle, asking, streaming, denied, gone, file mode) and the card list. No router, no global store, no context provider, no camera library, no image processing library, no clustering library, no icon pack. getUserMedia, IndexedDB, OffscreenCanvas, and the standard Canvas2D context are used directly. Every glyph including the small shoot icon and the lightbox close x is inline svg in the component that uses it.

Files. index.html with the Anton, Sora, and DM Mono links in the head. src/main.tsx as the React entry. src/App.tsx exporting one default component holding the camera state, the live swatches, and the saved cards. src/components/Header.tsx for the top display line. src/components/Viewfinder.tsx wrapping the video element and the resolution line. src/components/Palette.tsx for the six swatch row. src/components/ShootButton.tsx for the magenta round button. src/components/CardList.tsx for the saved list. src/components/Card.tsx for one saved shot. src/components/Lightbox.tsx for the full screen view. src/lib/camera.ts for the getUserMedia wrapper and the device label resolution. src/lib/sampler.ts for the OffscreenCanvas downsample and the pixel read. src/lib/median.ts for the median cut algorithm. src/lib/cards.ts for the IndexedDB store wrapper. src/lib/file.ts for the showOpenFilePicker fallback. tailwind.config.ts. package.json with react, react dom, vite, tailwind only. README.md with two short paragraphs.

A first time visitor opens the page on a laptop. The dark page paints, the empty viewfinder waits with a small allow the camera button below it. They allow access, the studio dark surface fills with their cluttered desk in 1920 by 1080. Six swatches appear under the viewfinder, the dominant color is a warm wood tan that matches their desk. They tap the leftmost swatch, the hex copies to the clipboard. They press space, the magenta dot blinks, a new card slides into the list below with the desk image and the six color stripes. They close the tab, reopen the next day, the saved card is still in the list.
