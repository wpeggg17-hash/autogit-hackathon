---
title: Event Schedule App
app_type: event-schedule-app
wallet: 0x76e9ac012eb6ac4938f6f9e9caa126d8824d77f6
---

You are an expert React developer. Generate a complete, production-ready React application for a daily event schedule and agenda app.

Requirements:

- Header with app title "AgendaQ" and today's date displayed prominently
- A horizontal day selector showing 7 days of the current week, clicking a day switches the schedule view
- A timeline view for the selected day showing hourly slots from 07:00 to 22:00
- Pre-populate Monday with 4 hardcoded events: Morning Standup (09:00), Lunch Break (12:00), Team Review (14:00), Gym Session (18:00)
- Each event block shows: event name, time, duration, and a color-coded left border by category (work=blue, personal=green, health=orange)
- An Add Event button that opens a simple inline form to add a new event with name, time, and category
- Empty hour slots show a faint dashed line
- Clean minimal design with white cards on a light gray background

Technical:

- React 18 with TypeScript
- Tailwind CSS for all styling
- Use useState for selected day and events list
- Export a single default App component
- No external UI libraries or icon packs
- Use inline SVG for any icons needed
- Fully responsive layout
