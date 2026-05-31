# CLAUDE.md — VitayKo Project Guide

## Project Overview

**VitayKo** is a Telegram Mini App (WebApp) — a single-page gift experience where a user picks an occasion, optionally enters a recipient's name, listens to music, and sends it as a personal gift.

**Slogan:** "Подарунок уваги. Для тих, кому є що сказати серцем."

- **Telegram bot:** `@vitayko_ua_bot`
- **GitHub repo:** `github.com/vitalismel/vitayko_ua_bot`
- **Live URL (GitHub Pages):** `https://vitalismel.github.io/vitayko_ua_bot/`
- **Owner GitHub:** `vitalismel`

---

## Architecture

This is a **zero-build, zero-dependency** project. Everything lives in a single HTML file (`index.html`) with embedded CSS and JavaScript. There is no package.json, no bundler, no framework.

### File structure (target state)

```
vitayko_ua_bot/
├── index.html          # Main app (all CSS + JS inline)
├── bg.jpg              # Background image (~47 KB JPEG)
├── CLAUDE.md           # This file
├── README.md
└── songs/              # MP3 files (to be added)
    ├── birthday.mp3
    ├── anniversary.mp3
    ├── love.mp3
    ├── gratitude.mp3
    ├── achievement.mp3
    ├── newborn.mp3
    ├── support.mp3
    ├── recovery.mp3
    ├── newstart.mp3
    └── proud.mp3
```

### Deployment

The app is deployed via **GitHub Pages** (branch: `main`, root `/`). Changes pushed to `main` go live at the GitHub Pages URL above. No CI/CD pipeline — push = deploy.

---

## Screen Flow

```
scr-home → scr-occasion → scr-name → scr-player → scr-creating → scr-final
```

| Screen ID      | Purpose                                   |
|----------------|-------------------------------------------|
| `scr-home`     | Landing — two CTA buttons                 |
| `scr-occasion` | Overlay — pick one of 10 occasions        |
| `scr-name`     | Overlay — optional recipient name input   |
| `scr-player`   | Overlay — music player + track list       |
| `scr-creating` | Overlay — 4-second animated transition    |
| `scr-final`    | Overlay — share / download / start over   |

The home screen (`scr-home`) uses class `screen active`. All other screens are `.overlay` elements toggled with `showOverlay(id)` / `hideAllOverlays()`. Back navigation is handled by `goBack(id)`.

---

## JavaScript Architecture

All JS is inline at the bottom of `index.html`. No modules, no imports.

### Global state object

```javascript
var state = {
  occasion: '',       // e.g. 'birthday'
  occasionLabel: '',  // e.g. 'День народження'
  name: '',           // recipient name (optional)
  songIdx: -1,        // currently selected song index (-1 = none)
  playing: false
};
```

### Key functions

| Function | Description |
|---|---|
| `showOverlay(id)` | Activate overlay screen; shows Telegram BackButton |
| `hideAllOverlays()` | Return to home; hides Telegram BackButton |
| `goBack(id)` | Navigate back one step in the flow |
| `goToOccasion(e)` | Home → Occasion; spawns particles on click |
| `selectOccasion(key, label)` | Occasion → Name (380ms delay) |
| `goToPlayer()` | Name → Player; calls `buildSongList()` |
| `goToFinal()` | Player → Creating (4s animation) → Final |
| `buildSongList()` | Renders `.song-list` from `SONGS` array |
| `loadSong(idx)` | Loads track into `<audio>` element |
| `togglePlay()` / `playAudio()` / `pauseAudio()` | Audio control |
| `prevSong()` / `nextSong()` | Cycle through tracks |
| `seekAudio(e)` | Click-to-seek on progress bar |
| `sendToTelegram()` | Share via `tg.openTelegramLink` or `navigator.share` |
| `downloadAudio()` | Trigger MP3 download |
| `shareApp()` | Share bot link |
| `startOver()` | Reset all state, return to home |
| `spawnParticles(x, y)` | Decorative particle burst on button click |
| `haptic(type)` | Wraps `tg.HapticFeedback.impactOccurred` |
| `showToast(msg)` | 2.6s toast notification |

### Telegram WebApp integration

```javascript
var tg = (window.Telegram && window.Telegram.WebApp) ? window.Telegram.WebApp : null;
if (tg) {
  tg.ready();
  tg.expand();
  tg.BackButton.onClick(function() {
    var a = document.querySelector('.overlay.active');
    if (a) goBack(a.id);
  });
}
```

The app works in both Telegram (full WebApp features) and regular browser (fallback to `navigator.share` / `navigator.clipboard`).

---

## Music / SONGS Array

```javascript
var SONGS = [
  { id:0, title:'Тепло серця',   src:'...' },
  { id:1, title:'Промінь уваги', src:'...' },
  { id:2, title:'Момент щастя',  src:'...' },
  { id:3, title:"З любов'ю",    src:'...' },
  { id:4, title:'Тихий вечір',   src:'...' }
];
```

**Current state:** `src` values point to `soundhelix.com` demo tracks.

**Target state:** Replace `src` with GitHub Pages URLs once MP3s are uploaded:
```
https://vitalismel.github.io/vitayko_ua_bot/songs/<filename>.mp3
```

Each of the 10 occasions maps to one MP3 file (see file structure above).

---

## Design System

### Colors (CSS custom properties)

```css
--olive:      #5C6B45
--olive-dark: #3D4A2E
--gold:       #C8A84B
--gold-light: #E2C97A
--gold-dark:  #9A7A2E
--cream:      #F5F0E8
--text-soft:  #7A8C5E
--text-muted: #A8B898
```

### Fonts (Google Fonts)

- **Cormorant Garamond** — headings, titles, slogans
- **Raleway** — labels, buttons, small text

### Symbols (use these, never emoji)

```
♡  heart
✦  star
♪  note
♫  double note
·  separator dot
›  arrow
```

### Strict prohibitions in UI

- No standard emoji (😊🎂🎁 etc.)
- No words: AI, генерація, нейромережа, аудіофайл, MP3

---

## Background Image

The app uses `.bg-image` (position: absolute, inset: 0) with a `.bg-overlay` on top for the frosted/tinted effect.

**Current state:** `background-image` in `.bg-image` CSS contains a base64-encoded JPEG (~47 KB) — this is the known bug causing the image not to display.

**Fix required:**
1. Extract the base64 from HTML and save as `bg.jpg`
2. Replace the inline `url('data:image/jpeg;base64,...')` with `url('bg.jpg')`
3. Commit and push both `index.html` and `bg.jpg`

```python
# Extract bg.jpg from base64 in index.html
import re, base64
with open('index.html') as f:
    html = f.read()
match = re.search(r'base64,([^\']+)', html)
if match:
    data = base64.b64decode(match.group(1))
    open('bg.jpg', 'wb').write(data)

# Replace base64 reference with file reference
html = re.sub(r"url\('data:image/jpeg;base64,[^']+'\)", "url('bg.jpg')", html)
with open('index.html', 'w') as f:
    f.write(html)
```

---

## Occasions (10 total)

| Key           | Ukrainian label        | Symbol |
|---------------|------------------------|--------|
| `birthday`    | День народження        | ♡      |
| `anniversary` | Річниця                | ✦      |
| `love`        | Просто люблю тебе      | ♡      |
| `gratitude`   | Подяка                 | ✦      |
| `achievement` | Досягнення             | ♪      |
| `newborn`     | Народження дитини      | ♡      |
| `support`     | Підтримка              | ♡      |
| `recovery`    | Одужання               | ✦      |
| `newstart`    | Новий початок          | ♪      |
| `proud`       | Пишаюсь тобою          | ✦      |

---

## Development Workflow

### Local testing

```bash
# From the repo root
python3 -m http.server 8080
# Open: http://localhost:8080
```

No build step needed. Edit `index.html` directly and reload the browser.

### Deploying

```bash
git add index.html bg.jpg
git commit -m "describe the change"
git push origin main
```

GitHub Pages auto-deploys from `main`. Allow ~1 minute for changes to go live.

### Connecting to Telegram

After GitHub Pages is live, configure via BotFather:
```
/setmenubutton → URL: https://vitalismel.github.io/vitayko_ua_bot/
/setdomain     → https://vitalismel.github.io/vitayko_ua_bot/
```

---

## Outstanding Tasks

- [ ] **CRITICAL:** Fix background image — extract base64 to `bg.jpg`, update CSS reference
- [ ] Upload `index.html` to GitHub `main` branch
- [ ] Upload `bg.jpg` to GitHub `main` branch
- [ ] Enable GitHub Pages (Settings → Pages → Branch: main → Save)
- [ ] Configure bot via BotFather (setmenubutton + setdomain)
- [ ] Replace demo soundhelix tracks with actual MP3s from suno.com
- [ ] Upload MP3 files to `songs/` directory

---

## Coding Conventions

- All user-visible text is **Ukrainian only**
- Style is warm and human — no technical jargon in the UI
- Keep JS vanilla (no libraries). The entire app must remain a single deployable HTML file (until the bg.jpg split is done)
- When adding a new occasion: add to `SONGS` array, add `.occ-item` in `scr-occasion`, add corresponding MP3 to `songs/`
- Haptic feedback on every meaningful user interaction (`haptic('light')` default, `haptic('medium')` for primary actions)
- Toast notifications (`showToast`) for async feedback (download started, link copied, etc.)

---

## Future Roadmap

- Suno API integration for personalized AI-generated music per occasion
- Per-occasion song mapping (currently all occasions share the same 5-track pool)
