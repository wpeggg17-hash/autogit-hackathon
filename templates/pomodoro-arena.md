---
title: Pomodoro Arena
app_type: pomodoro-arena
wallet: 0x22c07247391D55Fa481ACd805C125c07b0e38A18
---

Twenty five minutes of work, five minutes of rest, repeat for four rounds, then a longer rest. The page is a single full screen timer with a big jade ring counting down around a single bold number. The page vibrates the device when a round ends, rumbles a connected game controller when one is paired, and refuses to let two tabs run the timer at the same time. The page is intentionally loud only at the transitions, quiet everywhere else.

· · · SURFACE · · ·

A dojo black background, the kind of black that absorbs the light from a single overhead lamp. Rice paper for the type, the warm cream of a folded handkerchief. Sumi red reserved for the active break state and the small alert dot when the timer is paused. Jade green reserved for the work state ring and the big number while a work round is running. Big Shoulders Display at 160px for the central time number, the only place this typeface appears. Inter at 14px 500 for buttons and at 11px 400 for the small status line at the bottom. JetBrains Mono at 12px for the round counter and any small numeric badges. The layout is one full viewport, no scroll, centered on the time number.

A small inline svg circle drawn behind the number is the timer ring, 360 pixels across on desktop, 240 on phones, with a thin 4 pixel jade stroke during work and a thin sumi red stroke during break. The ring is drawn with a stroke dasharray equal to its circumference and a stroke dashoffset that animates from 0 to the full circumference over the round duration, creating the visual countdown. The ring rotates negative 90 degrees so the empty section starts at the top.

· · · ROUNDS · · ·

A pomodoro arena round is one of four states. Work (twenty five minutes), short break (five minutes), long break (fifteen minutes after the fourth work round), and idle (nothing running). The state machine is straightforward, work goes to short break, short break goes to work, after four work rounds the state goes to long break, then back to work. A small in source state machine in src/lib/cycle.ts holds the transitions. The lengths are configurable through a small settings drawer at the top right, but the defaults are the canonical values.

The big number shows the remaining minutes and seconds in the form 24:38, recomputed every second through a setInterval keyed to a rigorous absolute end time stored in localStorage, so the timer stays accurate even when the tab is backgrounded by the browser and the setInterval fires irregularly. The end time is recomputed at start and pause. The interval is purely a render trigger, the truth is the end time minus the current performance.now offset.

Below the big number, a JetBrains Mono 12px line shows the round count as round 2 of 4 work. Below that a Inter 14px line shows the next state in muted rice paper, like next, short break (5 min). To the right of the big number, two small text buttons in Inter 14px 500 stacked vertically, the first reads start when idle and pause when running, the second reads skip ahead.

· · · VIBRATION · · ·

Every transition fires a small vibration pattern through navigator.vibrate. The patterns are tuned so a worker with the phone in their pocket can recognize the state change without looking.

```
{
  workEnd:  [180, 80, 180, 80, 240],          // three buzzes, the last one longer
  breakEnd: [120, 60, 120, 60, 120, 60, 120], // four short, the all clear
  pause:    [60],                               // single tick
  resume:   [60, 40, 60]                        // two ticks
}
```

The patterns are short on purpose. A vibration that lasts longer than a second is rude. A vibration that does not arrive at all is a missed transition. The patterns are calibrated against both ios and android devices in source comments next to the pattern array.

If the Vibration api is unavailable, the page still flashes the ring color briefly at every transition, the visual feedback is the fallback. The page never plays audio without an explicit user gesture, the visitor decides whether to add a sound.

· · · CONTROLLER · · ·

The page polls navigator.getGamepads every frame through requestAnimationFrame when at least one gamepad has connected through the gamepadconnected event. The first connected gamepad becomes the active one. The button mapping is fixed and small.

```
{
  0:  'start or pause',     // a or cross
  1:  'skip ahead',          // b or circle
  3:  'reset the cycle',     // y or triangle
  8:  'open settings',       // select
  9:  'close settings',      // start
  12: 'longer round',        // dpad up
  13: 'shorter round'        // dpad down
}
```

A small JetBrains Mono 11px line at the bottom of the page reads gamepad connected when a controller is paired. If the controller exposes a vibrationActuator (most modern xbox and playstation controllers), the page rumbles it briefly through the dual rumble effect on every transition, slightly longer and stronger than the phone vibration. The rumble durations match the vibration patterns above with a small offset for the controller motor spin up.

· · · LOCKS · · ·

The page uses the Web Locks api to make sure two open tabs do not run the timer in parallel. On mount, the App requests a single named lock called pomodoro.arena.timer with mode exclusive and steal false. The request returns immediately when the lock is granted, the timer starts running. If a second tab is opened, the request waits and the second tab shows a small italic 14px Inter line in the center reading the timer is already running in another tab. The visitor can choose to take over by tapping a small Inter 12px text button below the line reading run it in this tab instead, which posts a release request to the first tab through a BroadcastChannel named pomodoro.arena.takeover, the first tab releases the lock, the second tab acquires it, and the timer continues from the same end time without a break.

The lock is released on visibilitychange to hidden after thirty seconds of being hidden, so a backgrounded tab does not block a foreground tab forever, but the timer state stays in localStorage so resuming from the lock is fast.

· · · SETTINGS · · ·

A small text button in the top right corner of the page in 12px JetBrains Mono rice paper reads settings. Tapping it opens a small panel sliding down with four number inputs labelled work minutes (default 25), short break minutes (default 5), long break minutes (default 15), rounds before long break (default 4), plus a small text button reading restore defaults. The settings persist in localStorage under the key pomodoro.arena.settings.v1. Changes take effect on the next round, the current round runs to completion at its original duration to keep the visitor's expectation intact.

A small Inter 12px italic line at the bottom of the panel reads these settings are kept only on this device.

· · · STATUS · · ·

The very bottom of the page holds a thin Inter 11px italic status line that summarizes the current session in one phrase. Examples.

```
running work, 14:22 left, lock held in this tab
on a short break, 03:51 left, vibration ready, controller paired
paused, 24:38 saved, ready to resume
the timer is in another tab, this view is read only
```

The line updates on every state change and on every second. The line is the only place the page tells the visitor what is happening through words rather than numbers and colors.

· · · EDGES · · ·

A device that does not expose vibration shows a small italic 11px line in the status line reading vibration is not available on this device, and the visual ring flash takes over. A gamepad that disconnects mid round fires the gamepaddisconnected event, the line updates to controller is gone, the rumble stops, the timer continues. A second gamepad connecting while the first is still active is logged but not promoted to active, the visitor can swap controllers through a small swap text button in the settings panel.

A lock that fails to acquire (because the first tab is unreachable or hung) shows a small text button in the center of the page reading force this tab to run, which calls navigator.locks.request with steal true after a five second confirm.

If the visitor closes the tab mid round, the end time stays in localStorage with a small flag, and the next time the page is opened it asks resume the round that was running, in 16px Inter, with two text buttons resume and start fresh.

· · · BUILD · · ·

The build is a single Vite project. React 18 with TypeScript. Tailwind CSS for layout and color, custom palette in tailwind.config.ts adding dojo black, rice paper, rice paper muted, sumi red, jade. Vite as the build tool. State is plain useState and one useReducer for the state machine, the end time, the round count, and the gamepad pairing. No router, no global store, no context provider, no timer library, no gamepad library, no svg animation library, no icon pack. navigator.vibrate, navigator.getGamepads, navigator.locks, and BroadcastChannel are all used directly. Every glyph including the small settings gear and the inline svg ring is in the component that renders it.

Files. index.html with the Big Shoulders Display, Inter, and JetBrains Mono links in the head. src/main.tsx as the React entry. src/App.tsx exporting one default component holding the cycle state, the end time, the round count, and the settings. src/components/Ring.tsx for the svg circle and the stroke dashoffset animation. src/components/Number.tsx for the big Big Shoulders Display countdown. src/components/Controls.tsx for the start, skip, and other text buttons. src/components/Settings.tsx for the top right panel. src/components/Status.tsx for the bottom italic line. src/lib/cycle.ts for the state machine and the absolute end time math. src/lib/vibrate.ts for the navigator.vibrate patterns. src/lib/gamepad.ts for the polling loop, button mapping, and rumble. src/lib/locks.ts for the navigator.locks request and the BroadcastChannel takeover. src/lib/storage.ts for the localStorage try catch. tailwind.config.ts. package.json with react, react dom, vite, tailwind only. README.md with two short paragraphs.

A first time visitor opens the page on a laptop with a controller paired through bluetooth. The dojo black page paints, the big number reads 25:00 in Big Shoulders Display jade. They press the a button on the controller. The ring starts to drain, the number ticks down, the controller rumbles briefly. They walk away to make tea. Twenty five minutes later the controller buzzes three times and the page swaps to a short break, the ring redraws in sumi red, the number reads 05:00. They open a second tab on the same machine, the second tab tells them the timer is already in another tab. They close the second tab, finish the break, the controller buzzes four times for the all clear, the next work round begins.
