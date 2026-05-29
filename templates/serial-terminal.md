---
title: Serial Terminal
app_type: serial-terminal
wallet: 0x7049a50a44A9E26e26BA68E68A4146e20D69Ea23
---

Plug a usb to serial dongle into the laptop. Open the page. Tap connect, pick the port from the browser's chooser, the page opens a stream at the chosen baud rate and prints every byte the device sends. Type a line, hit return, the line flies down the wire. The terminal scrolls up forever, the line buffer is bounded, and the visitor can pause the output and copy the buffer to the clipboard. No driver, no library, no electron, no native code. Web Serial in a browser tab.

# serial terminal

❯ surface

A terminal black background. Phosphor green for the body type, the color of a vintage crt set to monochrome. Retro amber for the title bar and the small connection status dot. Error magenta for any failure line or rejected message. IBM Plex Mono at 14px 400 for the live output and at 13px for the small status line, weight 600 for the prompt prefix. The optional VT323 web font is used for the header at the top of the page, with IBM Plex Mono as the fallback. Mono only, no other typeface. No icons except a small inline svg of a circular plug in the top left of the page, twelve pixels across, two paths. The page is one full viewport, the output buffer fills the screen, the input line sits at the bottom, the title bar at the top.

❯ status

A thin status bar across the top of the page, 32px tall, terminal black with a 1px phosphor green hairline at twenty percent opacity at the bottom. Left to right inside the bar.

The plug glyph in retro amber when connected, in dim phosphor green at thirty percent when disconnected.

The title in 14px VT323 (or IBM Plex Mono) retro amber, reading serial terminal.

The port descriptor when connected, in 12px IBM Plex Mono phosphor green, in the form usb id 067b: 2303 at 115200 baud, 8n1. The descriptor is read from the port's getInfo call and the connection options the visitor chose.

A small text button on the right in 12px IBM Plex Mono reads connect when disconnected, disconnect when connected. A second small button reads settings, which opens the baud and framing dialog.

❯ output

Below the status bar, the output buffer fills the rest of the page minus the bottom input line. The buffer is a single scrollable region, terminal black background, phosphor green text, padded 12px left and right and 8px between lines. Each incoming chunk is decoded through TextDecoderStream and appended. The page splits the decoded text on newlines and renders one logical line per row, with the most recent line at the bottom and older lines scrolling up. The output buffer holds at most 5000 lines, older lines drop off silently. The current scroll position is anchored to the bottom by default. When the visitor scrolls up to inspect older lines, the page detaches the anchor, a small italic 11px line at the bottom of the buffer reads scroll lock on, scroll back to the bottom or tap resume to follow.

A small text button in the top right corner of the output area in 11px IBM Plex Mono reads pause when running, resume when paused. Tapping pause stops appending incoming lines to the visible buffer (incoming bytes are still queued in memory, capped at the same 5000 line ring). Tapping resume drains the queue and re anchors to the bottom.

Below the output, a single thin 1px phosphor green hairline at twenty percent opacity, then the input row.

❯ input

The input row is a single line of IBM Plex Mono 14px phosphor green, with a 600 weight prompt prefix in retro amber reading `> ` followed by the visitor's cursor. Typing inside the input edits the line. Pressing the enter key sends the line to the port, terminated by the line ending chosen in the settings dialog (default carriage return plus line feed), and appends the sent line to the output buffer with the prefix `> ` so the visitor can see what they sent. Pressing the up and down arrows walks through the history of sent lines, capped at 200 entries, also persisted across sessions to localStorage under the key serial.terminal.history.v1.

A small text button to the right of the input reads send file, which opens a file picker and streams the picked file's bytes through the writer. The file is sent in chunks of 64 bytes with a small 5 millisecond pause between chunks to avoid overrunning the device, and a small italic 11px line under the input reads sending file, 1284 of 4096 bytes, with the progress updating live.

❯ port

The Web Serial api is the only path. On the first connect press, the App calls navigator.serial.requestPort, which opens the browser's chooser. The visitor picks the port, the App calls port.open with the visitor's chosen options.

```
await port.open({
  baudRate: 115200,
  dataBits: 8,
  parity:   'none',
  stopBits: 1,
  flowControl: 'none'
});
```

The reader and writer are then created from the port's readable and writable streams. The reader is piped through a TextDecoderStream with the visitor's chosen encoding (default utf 8) so the App reads decoded text rather than raw bytes. The writer is fed strings encoded through a TextEncoder.

The port object is an EventTarget. The App attaches connect and disconnect listeners to the navigator.serial object itself for hot plug detection, and a close listener on the port for the disconnect path. The port closes cleanly when the visitor presses disconnect, when the page is hidden for more than thirty seconds, and when the browser sends a page lifecycle freeze event.

```
const reader = port.readable.pipeThrough(new TextDecoderStream()).getReader();
while (true) {
  const { value, done } = await reader.read();
  if (done) { break; }
  bufferAppend(value);
}
```

The reader loop runs inside an async function inside src/lib/serial.ts. The loop exits when reader.read returns done, which happens on port close or on a device unplug.

❯ settings

The settings dialog (an HTMLDialogElement modal) holds the connection options.

baud rate, dropdown, 9600, 19200, 38400, 57600, 115200, 230400, 921600.

data bits, dropdown, 5, 6, 7, 8.

parity, dropdown, none, even, odd.

stop bits, dropdown, 1, 2.

flow control, dropdown, none, hardware.

line ending sent on enter, dropdown, lf, cr, crlf.

encoding for the decoder, dropdown, utf 8, latin 1, ascii.

Each setting persists in localStorage under the key serial.terminal.options.v1. Changes apply on the next connect. A small italic 11px line at the bottom of the dialog reads these options are kept only on this device.

❯ buffer

A small text button below the output in 12px IBM Plex Mono reads copy the buffer to the clipboard, which writes the current buffer (up to 5000 lines) as plain text. A second small text button reads clear the buffer, which empties the buffer with no confirm (the visitor is presumed to know what they want at a terminal). A third small text button reads save as text file, which downloads the current buffer through a temporary anchor with a filename of serial buffer and the current timestamp.

A small status line in 11px IBM Plex Mono at the bottom right of the page shows live counters in the form 412 lines, 18234 bytes, 1.2 lines per second, updated every second. The counter for lines per second is a rolling average of the last 10 seconds.

❯ edges

A port that disconnects mid stream (the dongle was unplugged) fires the close event on the port and the disconnect event on navigator.serial. The App swaps the connect button back to its idle state, the status bar plug glyph fades, and the output buffer gets a small italic 11px line reading the device went away. The lines already in the buffer are preserved.

A connect attempt that fails (the visitor cancelled the chooser, the port refused to open, the device is in use by another process) shows a small italic 11px line under the connect button in error magenta reading the port could not be opened, with the specific error message. The button remains pressable.

A send to a port that has just disconnected throws, the App catches the error, shows the same error magenta line, and the visitor must reconnect.

If the Web Serial api is unavailable (firefox, safari at most versions), the page shows a single error magenta line at the top reading this browser does not support web serial, the page is read only here and a small text button below it reads see which browsers support web serial that links to a small static reference page bundled in the source.

❯ build

The build is a single Vite project. React 18 with TypeScript. Tailwind CSS for layout and color, custom palette in tailwind.config.ts adding terminal black, phosphor green, phosphor green muted, retro amber, error magenta. Vite as the build tool. State is plain useState and one useReducer for the connection state machine and the input history. No router, no global store, no context provider, no serial library, no terminal emulator library, no icon pack. navigator.serial, TextDecoderStream, TextEncoder, the standard Streams api, and HTMLDialogElement are all used directly. Every glyph including the small plug and the dropdown carets is inline svg in the component that uses it.

Files. index.html with the VT323 and IBM Plex Mono links in the head. src/main.tsx as the React entry. src/App.tsx exporting one default component holding the port handle, the connection state, the output buffer, the input value, and the history. src/components/StatusBar.tsx for the top bar. src/components/OutputBuffer.tsx for the scrollable output region. src/components/InputRow.tsx for the bottom line. src/components/SettingsDialog.tsx for the HTMLDialogElement modal. src/lib/serial.ts wrapping navigator.serial.requestPort, port.open, the reader loop, and the writer. src/lib/history.ts for the line history persistence. src/lib/options.ts for the settings shape and the localStorage. tailwind.config.ts. package.json with react, react dom, vite, tailwind only. README.md with two short paragraphs.

A first time visitor opens the page in chrome with a usb to serial dongle plugged in. The terminal black surface paints, the plug is dim, the status bar reads serial terminal in retro amber. They press connect, pick the dongle from the chooser, the plug lights up retro amber, the status bar shows the port id and the baud. A few lines of phosphor green text start scrolling up the buffer as the device sends its boot message. They type a command at the prompt, press enter, the command flies down the wire, the device's response prints below. They press pause to inspect a line, scroll up, find what they need, press resume. They press copy the buffer to the clipboard, paste into a notes app, the full session lands as plain text.
