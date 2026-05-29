---
title: Pomodoro Timer
app_type: pomodoro-timer
wallet: 0xbf5cb14280d86a922fea9a9a14430c410bb58501
---

You are an expert React developer. Generate a complete, production-ready React application for a Pomodoro productivity timer.

Requirements:

- Header with app title "FocusFlow" and a tagline "Work smarter, not harder"
- A large circular timer display showing minutes and seconds remaining, styled as an SVG circle progress ring that depletes as time counts down
- Three mode buttons: Focus (25 min), Short Break (5 min), Long Break (15 min) — clicking switches the timer and resets it
- Start, Pause, and Reset buttons below the timer
- A session counter showing how many Pomodoros completed today (e.g. "3 / 4 sessions")
- After every 4 focus sessions, automatically suggest a long break by highlighting the Long Break button
- A task input field where user can type what they are working on, displayed below the timer during focus mode
- Soft color theme: red/tomato for focus mode, green for short break, blue for long break

Technical:

- React 18 with TypeScript
- Tailwind CSS for all styling
- Use useState and useEffect with setInterval for countdown logic
- Export a single default App component
- No external UI libraries or icon packs
- Use inline SVG for the circular progress ring
- Fully responsive layout
