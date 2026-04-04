# AGENTS.md - AI Assistant Guide for Coreus

> This document provides comprehensive guidance for AI assistants working on the Coreus codebase. Read this entirely before making any modifications.

---

## Project Overview

**Coreus** is a fully self-contained, single-file "Unblocked Browser Game" (UBG) platform designed to be completely unblockable. The entire application logic lives in a single `index.html` file (~4,100+ lines), making it portable and deployable anywhere - from data URLs to GitHub Pages.

| Attribute | Value |
|-----------|-------|
| **License** | AGPL v3.0 |
| **Author** | n00dle (noodlelover1) |
| **Live Demo** | https://n00dle.is-a.dev/coreus/ |
| **Repository** | https://github.com/noodlelover1/coreus |

---

## Architecture Philosophy

### Single-File Design Principle

The core design principle is **maximum portability**. The entire application must remain functional as a single HTML file that can be:
- Opened directly in a browser
- Pasted into online HTML editors
- Embedded as a data URL
- Served from any static hosting

**CRITICAL**: When making changes, you MUST keep the single-file architecture intact. Do NOT split `index.html` into separate files unless explicitly requested.

### File Structure

```
coreus/
├── index.html              # THE MAIN APPLICATION (~4,100 lines) - contains ALL app logic
├── server.js               # Optional Express.js server (simple static file server)
├── app.json                # Heroku deployment configuration
├── amplify.yml             # AWS Amplify deployment configuration
├── package.json            # Node.js dependencies (express only)
├── icon.svg                # Application logo
├── LICENSE                 # AGPL v3.0 license
├── README.md               # User documentation
├── AGENTS.md               # This file - AI assistant guide
├── games.json          # Games catalog metadata (113 VRTX games)
├── img/                # Game thumbnail images (*.webp, *.png, *.jpg, *.jpeg, *.avif)
└── games/              # Individual game folders (100+ games)
    ├── gdash/          # Geometry Dash
    ├── amongus/        # Among Us
    ├── cookieclicker/  # Cookie Clicker
    └── ...             # Many more games
```

---

## Technical Stack

### Frontend (index.html)

| Component | Technology | CDN Source |
|-----------|------------|------------|
| **Icons** | Font Awesome 6.4.0 | cdnjs.cloudflare.com |
| **Markdown** | Marked.js | cdn.jsdelivr.net |
| **jQuery** | jQuery 3.6.0 | code.jquery.com |
| **AI Integration** | Puter.js v2 | js.puter.com |
| **Debug Console** | Eruda | cdn.jsdelivr.net |
| **Styling** | CSS Variables + Inline CSS | N/A |

### Backend (server.js)

```javascript
// Simple Express.js static file server
const express = require('express');
const app = express();
app.use(express.static(__dirname));
app.listen(process.env.PORT || 3000);
```

The server is optional - the app works fully client-side.

---

## Core Features Deep Dive

### 1. Games Library

**Two Game Backends:**

| Backend | Description | Games Count | Mobile Friendly |
|---------|-------------|-------------|-----------------|
| **VRTX** | Local games stored in `assets/games/` | 113+ | Yes |
| **Selenite** | External | 500+ | Partial |

**Games Data Structure** (`assets/games.json`):
```json
{
  "name": "Geometry Dash",
  "tags": "geometry, dash, rhythm",
  "image": "img/gdash.png",
  "href": "games/gdash/",
  "category": "Rhythm"
}
```

**Categories**: Action, Racing, Puzzle, Idle, Sports, Strategy, Casual, Adventure, Rhythm, Simulation, Social

**Key Functions:**
- `loadGames()` - Fetches and renders game cards
- `loadGameFromExternalApi(gameSlug)` - Loads Selenite games
- `openGame(gameName, gameUrl)` - Opens game in popup iframe
- `closeGame()` - Closes game popup
- `filterGames(category)` - Filters by category
- `searchGames(query)` - Search functionality

### 2. Web Proxy (Ultraviolet)

**Proxy APIs with Fallback System:**
```javascript
const proxyApis = [
    'https://embeddr.rhw.one/?url=',      // Primary
    'https://embeddr.pages.dev/?url=',    // Fallback 1
    'https://ev2.rhw.one/?url=',          // Fallback 2
    'https://api.allorigins.win/raw?url=', // Fallback 3
    'https://cors.lol/?url=',             // Fallback 4
    'https://yacdn.org/proxy/'            // Fallback 5
];
```

**Key Functions:**
- `proxyNavigate()` - Navigate to URL or search query
- `proxyLoadUrl(targetUrl)` - Load URL through proxy
- `proxyGoBack()` / `proxyGoForward()` - Browser history navigation
- `proxyReload()` - Refresh current page
- `proxyClearUrl()` - Clear and reset proxy state

**URL Detection**: Automatically detects domains vs search queries using regex pattern for TLDs.

### 3. AI Chatbot

**Integration**: Uses Puter.js SDK for AI capabilities.

**Available Models:**
- `gpt-5-nano` (default)
- `gpt-4o-mini`
- `claude-3-5-sonnet`
- `meta-llama/Meta-Llama-3.1-8B-Instruct-Turbo`

**Authentication Flow:**
1. User sends message
2. Check `isPuterLoggedIn()`
3. If not logged in, show `puterLoginModal`
4. After auth, retry with `pendingAiMessage`

**Key Functions:**
- `sendAiMessage(messageText)` - Send message to AI
- `addMessageToChat(sender, text)` - Render message bubble
- `convertMarkdownToHtml(text)` - Parse AI response markdown
- `handlePuterSignIn()` - OAuth flow with Puter

### 4. Achievements System

**Total Achievements**: 18

**Achievement Categories:**
- Games played milestones (1, 5, 10, 20, 50)
- Proxy usage milestones
- Chat room participation
- Tool usage
- Time spent on site
- Favorites system
- First visit/explorer achievements

**Key Functions:**
- `checkAchievements()` - Evaluate all achievement conditions
- `unlockAchievement(id)` - Grant achievement
- `getUnlockedAchievements()` - Return array of unlocked IDs
- `updateAchievementsCard()` - Update UI counter

### 5. Account System

**Data Synced:**
- Achievements
- Played games history
- Used tools
- Proxy visit history
- Search count
- Chat visits
- Favorites (games & tools)
- Total time on site
- Theme preference

**Storage**: LocalStorage with custom encoding for sync codes.

**Key Functions:**
- `encodeAccount(obj)` - Create shareable account code
- `decodeAccount(hash)` - Restore from account code
- `saveAccount(acc)` - Persist to localStorage
- `loadAccount()` - Load on page init
- `applyAccount()` - Apply synced data

### 6. Tab Cloaking

**Cloaking Options:**
- No cloaking
- Google Docs
- Google Drive
- Google Classroom
- Canvas
- Schoology
- Khan Academy
- Wikipedia
- Custom (user-defined title/favicon)

**Key Functions:**
- `applyCloaking(option)` - Set tab title and favicon
- `openInAboutBlank()` - Open site in about:blank for stealth

### 7. Theming System

**CSS Variables (`:root`):**
```css
--primary-bg       /* Main background */
--secondary-bg     /* Secondary surfaces */
--tertiary-bg      /* Input backgrounds */
--primary-text     /* Main text color */
--secondary-text   /* Muted text */
--border-color     /* Border color */
--accent-color     /* Brand color */
--accent-light     /* Lighter accent */
--navbar-bg        /* Bottom navbar bg */
--card-bg          /* Card backgrounds */
```

**Available Themes:**
- Default (green accent on black)
- Blue (`theme-blue`)
- Purple (`theme-purple`)
- Green (`theme-green`)

**Apply Theme:**
```javascript
document.body.classList.add('theme-blue');
```

---

## View System Architecture

### View Management Pattern

The app uses a single-page application (SPA) pattern with view switching:

```javascript
function hideAllViews() {
    document.querySelector('.container').style.display = 'none';
    ['gamesView', 'toolsView', 'aboutView', 'settingsView', 
     'chatView', 'aiView', 'accountView', 'proxyView', 
     'achievementsView', 'statsView', 'gamePopup'].forEach(id => {
        const el = document.getElementById(id);
        if (el) el.style.display = 'none';
    });
}

function showGamesView() {
    hideAllViews();
    document.getElementById('gamesView').style.display = 'block';
    previousView = 'games';
}
```

### All Views

| View ID | Description | Show Function |
|---------|-------------|---------------|
| `.container` | Home/landing page | `location.reload()` |
| `gamesView` | Games library | `showGamesView()` |
| `toolsView` | Tools section | `showToolsView()` |
| `proxyView` | Web proxy browser | `showProxyView()` |
| `aiView` | AI chatbot | `showAiView()` |
| `chatView` | Live chatroom | `showChatView()` |
| `aboutView` | About page | `showAboutView()` |
| `settingsView` | Settings | `showSettingsView()` |
| `accountView` | Account/profile | `showAccountView()` |
| `achievementsView` | Achievements list | `showAchievementsView()` |
| `statsView` | User statistics | `showStatsView()` |
| `gamePopup` | Full-screen game iframe | `openGame()` |

---

## UI Components

### Bottom Navigation Bar

Fixed position navbar at bottom of screen with icon buttons:
- Home (logo + house icon)
- Proxy (globe)
- Games (gamepad)
- Tools (wrench)
- AI (robot)
- Chat (comments)
- About (info-circle)
- Achievements (trophy)
- Stats (chart-bar)
- Settings (cog)

### Custom Cursor

The app implements a custom cursor system:
- `.cursor-dot` - White dot cursor that follows mouse
- `.click-ripple` - Expanding ring animation on click
- All elements have `cursor: none !important`

### Cards

- **Profile Card**: Top-left, shows logged-in username
- **Achievements Card**: Top-right, shows unlocked count (e.g., "5/18")
- **Game Cards**: Grid of game thumbnails with hover effects
- **Tool Cards**: Similar to game cards but for tools

### Modals

- `puterLoginModal` - Puter OAuth login prompt
- `backendSelectorModal` - VRTX/Selenite backend selection
- Settings modal sections for theme, cloaking, etc.

---

## Data Storage

### LocalStorage Keys

| Key | Type | Description |
|-----|------|-------------|
| `coreus_account` | JSON | Full account object |
| `coreus_achievements` | JSON Array | Unlocked achievement IDs |
| `coreus_played_games` | JSON Array | Played game slugs |
| `coreus_used_tools` | JSON Array | Used tool IDs |
| `coreus_proxy_visits` | JSON Array | Visited proxy URLs |
| `coreus_search_count` | Number | Total searches |
| `coreus_chat_visits` | Number | Chat room visit count |
| `coreus_favorite_count` | Number | Total favorites |
| `coreus_days_used` | Number | Days active |
| `coreus_total_time` | Number | Seconds on site |
| `coreus_game_favorites_vrtx` | JSON Array | VRTX favorite games |
| `coreus_game_favorites_selenite` | JSON Array | Selenite favorites |
| `coreus_tool_favorites` | JSON Array | Favorite tools |
| `cloakingOption` | String | Current cloaking preset |

---

## Adding New Features

### Adding a New Game (VRTX)

1. Create game folder in `assets/games/[game-slug]/`
2. Add game files (index.html or game assets)
3. Add thumbnail to `assets/img/[game-slug].png`
4. Add entry to `assets/games.json`:
```json
{
  "name": "Game Name",
  "tags": "keyword1, keyword2",
  "image": "img/game-slug.png",
  "href": "games/game-slug/",
  "category": "Category"
}
```

### Adding a New View

1. Add HTML structure in `index.html`:
```html
<div id="newView" style="display: none; position: fixed; ...">
    <!-- View content -->
</div>
```

2. Add show/hide functions:
```javascript
function showNewView() {
    hideAllViews();
    document.getElementById('newView').style.display = 'block';
    previousView = 'new';
}
```

3. Add navbar button if needed:
```html
<button class="nav-icon-btn" onclick="showNewView()">
    <i class="fas fa-icon-name" style="color: white; font-size: 20px;"></i>
</button>
```

4. Add to `hideAllViews()` function.

### Adding a New Theme

Add CSS class in `<style>` section:
```css
body.theme-newtheme {
    --primary-bg: #hexcolor;
    --secondary-bg: #hexcolor;
    --tertiary-bg: #hexcolor;
    --primary-text: #hexcolor;
    --secondary-text: #hexcolor;
    --border-color: #hexcolor;
    --accent-color: #hexcolor;
    --accent-light: #hexcolor;
    --navbar-bg: rgba(r, g, b, 0.80);
    --card-bg: #hexcolor;
}
```

Add option to theme selector in settings view.

### Adding a New Achievement

1. Define achievement in achievements array
2. Add unlock condition check in `checkAchievements()`
3. Update total count display (currently 18)

---

## Code Patterns and Conventions

### Inline Styles vs CSS

The codebase uses a mix:
- **CSS Variables**: For theming
- **`<style>` block**: For reusable classes
- **Inline styles**: For one-off styling (very common)

When adding new elements, follow existing patterns in that section.

### Event Handling

- Use `onclick` attributes for simple handlers
- Use `addEventListener` for complex logic in `DOMContentLoaded`

### Error Handling

- Use try/catch for async operations
- Log errors with `console.error()`
- Show user-friendly alerts with `alert()` or status divs

### Async/Await

The codebase uses async/await for:
- Puter AI calls
- Account sync operations
- IndexedDB operations

---

## External Dependencies

### CDN Dependencies (Required)

These MUST remain as CDN links for the single-file design:

```html
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
<script src="https://cdn.jsdelivr.net/npm/marked/marked.min.js"></script>
<script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
<script src="//cdn.jsdelivr.net/npm/eruda"></script>
<script src="https://js.puter.com/v2/"></script>
```

### External APIs

| API | Purpose | URL |
|-----|---------|-----|
| Selenite | External games | `api.selenite.cc` |
| Proxy APIs | Web proxy | Various (see proxy section) |
| Puter | AI chatbot | `puter.com` |
| CountAPI | Visitor counter | `api.countapi.xyz` |

---

## Deployment

### Static Hosting (Recommended)

Just serve `index.html` - no build step required:
- GitHub Pages
- Netlify
- Vercel
- Any static host

### Node.js Server

```bash
npm install
node server.js
# Runs on http://localhost:3000
```

### Python Server

```bash
python3 -m http.server 8000
# Runs on http://localhost:8000
```

---

## Common Tasks

### Fixing a Bug

1. Locate the function in `index.html`
2. Test in browser console first
3. Make minimal changes
4. Test all affected features

### Adding a Feature

1. Find similar existing feature
2. Follow its patterns
3. Add to appropriate section in HTML
4. Test on mobile and desktop

### Updating Games List

1. Edit `assets/games.json`
2. Add thumbnail to `assets/img/`
3. Add game files to `assets/games/[slug]/`

---

## Testing Checklist

Before committing changes:

- [ ] Home page loads correctly
- [ ] Games view shows all games
- [ ] Game search works
- [ ] Game popup opens/closes
- [ ] Proxy navigates correctly
- [ ] AI chat sends/receives (with Puter auth)
- [ ] Settings save properly
- [ ] Theme switching works
- [ ] Tab cloaking works
- [ ] Mobile view is functional
- [ ] Achievements unlock correctly
- [ ] Account sync works

---

## Important Warnings

1. **DO NOT** split `index.html` into multiple files
2. **DO NOT** remove CDN dependencies
3. **DO NOT** add build tools (webpack, etc.)
4. **DO NOT** add authentication that requires server-side code
5. **DO NOT** modify the AGPL license
6. **ALWAYS** test on mobile devices
7. **ALWAYS** maintain backwards compatibility with saved accounts

---

## Contributing Guidelines

From the README:

> AI contributions are accepted BUT you must:
> - Understand what the AI does
> - Review manually all the AI-generated code
> - Manually test the AI generated code

---

## Resources

- [Live Demo](https://n00dle.is-a.dev/coreus/)
- [GitHub Repository](https://github.com/noodlelover1/coreus)
- [Puter.js Documentation](https://docs.puter.com/)
- [Font Awesome Icons](https://fontawesome.com/icons)
- [Marked.js Documentation](https://marked.js.org/)

---
