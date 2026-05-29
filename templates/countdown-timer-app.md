---
title: Countdown Timer App
app_type: countdown-timer-app
wallet: 0xc8344eac1b84c0e6c8e436e279c8c4e197c2440e
---

You are an expert React developer. Generate a complete, production-ready React application for a multi-event countdown timer app.

Requirements:

- Header with app title "CountDown" and a tagline "Never miss a moment"
- Ability to display 4 hardcoded upcoming events: New Year 2027, Christmas 2026, Summer Solstice 2026, and a custom "My Birthday" set to December 1 2026
- Each event card shows: event name, a large countdown display broken into days, hours, minutes, seconds
- Countdown updates every second using setInterval
- Cards use different accent colors (blue, green, orange, purple) for visual variety
- A "Featured" badge on the nearest upcoming event
- Simple top navigation with app title on left and a clock icon on right
- Footer with current date and time updating every second

Technical:

- React 18 with TypeScript
- Tailwind CSS for all styling
- Use useState and useEffect with setInterval for live countdown
- Export a single default App component
- No external UI libraries or icon packs
- Use inline SVG for any icons needed
- Fully responsive, 2-column grid on desktop, single column on mobile
