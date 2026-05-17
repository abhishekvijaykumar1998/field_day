# Field Day 2026 🏆

A single-page web app for managing Field Day 2026 — teams, events, live brackets, and a real-time scoreboard powered by Firebase.

---

## Project Structure

```
field_day/
├── index.html       # The entire app — HTML, CSS, and JS in one file
├── preview.png      # Open Graph preview image for link sharing
└── README.md        # This file
```

---

## Hosting on Netlify

The site is hosted as a **static site** on Netlify — no build step, no server, no framework. Netlify just serves `index.html` directly.

### How it's connected

1. The GitHub repo (`abhishekvijaykumar1998/field_day`) is linked to Netlify
2. Every `git push` to the `main` branch triggers an automatic redeploy
3. The site is live at: **https://my-field-day.netlify.app**

### Deploying changes

```bash
# Make your changes to index.html, then:
git add index.html
git commit -m "describe your change"
git push
```

Netlify picks it up automatically within ~30 seconds.

### Adding new static files (e.g. images)

Drop any new files in the repo root and push. They'll be accessible at:
```
https://my-field-day.netlify.app/filename.ext
```

---

## Firebase Realtime Database

Firebase is used **only for the live scoreboard**. It stores and syncs event results across all devices in real time — no refresh needed.

### What Firebase stores

All data lives under one path in the Realtime Database:

```
event_results/
  dasher_m/
    1: "Placid"       ← 1st place player name
    2: "Michael"      ← 2nd place
    3: "Jimmy"        ← 3rd place
  dasher_w/
    1: "Ashley"
    2: "Kenzie"
    3: "Anita"
  caterpillar/
    1: "Red"          ← team events store team names
    2: "Blue"
    3: "Green"
  ... (one entry per event)
```

### Event keys

| Key | Event | Type |
|-----|-------|------|
| `dasher_m` | Dasher (Men) | Individual |
| `dasher_w` | Dasher (Women) | Individual |
| `caterpillar` | Caterpillar | Team |
| `tater_sac` | Tater Sac | Individual |
| `mellow` | Grape Shuffle | Team |
| `juggle` | Juggle Train | Individual |
| `jenga` | Jenga Smash | Team |
| `tug` | Tug of War | Team |
| `pins` | Pins | Team |

### How scores are calculated

- **Individual events** — winner's name is stored. The app looks up which team that player belongs to using the `playerTeam` map in the JS, then adds points to that team's total.
- **Team events** — winning team name is stored directly. Points go straight to that team.
- **Points system**: 1st = 3pts, 2nd = 2pts, 3rd = 1pt

### Real-time sync

```javascript
// This listener fires instantly whenever any score changes in Firebase
onValue(ref(db, 'event_results'), snapshot => {
  scores = snapshot.val() || {};
  renderScoreboard(); // re-renders the entire scoreboard UI
});
```

Every device with the page open receives updates within ~1 second of a score being saved — no polling, no refresh.

### Writing scores

```javascript
// When a leader saves results for an event:
await set(ref(db, `event_results/${eventKey}`), {
  1: 'PlayerOrTeamName',
  2: 'PlayerOrTeamName',
  3: 'PlayerOrTeamName'
});
```

Firebase Realtime Database uses **last-write-wins** — whoever saves last wins. To avoid conflicts, designate one person per event to enter scores.

### Clearing scores

```javascript
// Removes all results for a single event:
await remove(ref(db, `event_results/${eventKey}`));
```

This is triggered by the "🗑 Clear this event's scores" button in the score entry modal.

---

## Firebase Project Details

| Setting | Value |
|---------|-------|
| Project | `FieldDay` |
| Project ID | `fieldday-dda6a` |
| Database URL | `https://fieldday-dda6a-default-rtdb.firebaseio.com` |
| Plan | Spark (Free) |
| Database region | us-central1 |
| Security rules | Test mode (open read/write) |

### Free tier limits (Spark plan)

| Resource | Limit | Estimated usage |
|----------|-------|-----------------|
| Simultaneous connections | 100 | ~20 on event day |
| Storage | 1 GB | < 1 KB |
| Downloads | 10 GB/month | < 1 MB |

The app will comfortably stay within free tier limits indefinitely.

---

## Player & Team Configuration

These are defined in the Firebase script block in `index.html` and must be updated if players or teams change.

### Team roster (`playerTeam` map)

```javascript
const playerTeam = {
  Placid:'Red',  Ashley:'Red',   Abby:'Red',     Coen:'Red',    Anika:'Red',
  Michael:'Blue', Kenzie:'Blue', Nick:'Blue',    Leah:'Blue',   Grant:'Blue',
  Jimmy:'Green', Anita:'Green', Steven:'Green', Kayla:'Green', Devin:'Green',
  Abhi:'White',  Xavier:'White', Brianna:'White', Jack:'White', Lauren:'White'
};
```

### Gender groupings (for Dasher heats)

```javascript
const menPlayers   = ['Placid','Michael','Jimmy','Abhi','Nick','Coen','Grant','Steven','Xavier','Jack','Devin'];
const womenPlayers = ['Ashley','Abby','Anika','Kenzie','Leah','Anita','Kayla','Brianna','Lauren'];
```

---

## Making Changes

### Updating a player name
Search for the old name in `index.html` and replace all occurrences — it will appear in the team card HTML, `playerTeam` map, and the relevant gender array.

### Adding/removing an event
1. Add/remove the event object from the `events` array (for the event card + rules modal)
2. Add/remove the corresponding entry in `scoreEvents` (for the scoreboard)

### Changing team colors
Update both the inline HTML team cards and the `teamColors` object in the JS:
```javascript
const teamColors = { Red:'#e63946', Blue:'#7ab5d4', Green:'#52b788', White:'#f1faee' };
```
