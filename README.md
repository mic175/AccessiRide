 # AccessiRide (DevDash) — Accessible Rides, Faster

**Challenge #9 (SmartTech Hackathon) • Winner — 1st Place 🏆**  
Helping wheelchair users quickly find **nearby, truly** accessible transportation — without hours of research and phone calls.

> _“As a motorized wheelchair user, I imagine an app that uses my location to show companies with wheelchair vans, lifts, tie-downs, and seat belts — with hours, fees, and notice required.”_ — Mentor: **Monica Chupko**

## Table of contents
- [Demo](#demo)
- [Why this matters](#why-this-matters)
- [Key features](#key-features)
- [Mobile-first advantages](#mobile-first-advantages)
- [Architecture](#architecture)
- [Tech stack](#tech-stack)
- [Getting started](#getting-started)
  - [1) Mobile app (Expo)](#1-mobile-app-expo)
  - [2) Backend API](#2-backend-api)
  - [3) Web app](#3-web-app)
- [API reference](#api-reference)
- [Data](#data)
- [Accessibility](#accessibility)
- [Roadmap](#roadmap)
- [Contributing](#contributing)
- [Acknowledgements](#acknowledgements)
- [Maintainers / Contact](#maintainers--contact)

---

## Demo

- **Live web demo**: [https://boisterous-lolly-461c30.netlify.app/](https://boisterous-lolly-461c30.netlify.app/)  
- **Videos:**
  - Fundamental functions (demo 1): https://www.youtube.com/watch?v=PtLKyoecgmk&t=197s
  - AI assistant interaction (demo 2): https://www.youtube.com/watch?v=i4f0H3GSSWs
  - Mobile version walkthrough (demo 3): https://www.youtube.com/shorts/PzHxlr54sC0

---

## Why this matters

Finding accessible transportation often takes **hours** of searching, calls, and dead ends. Information is **scattered and outdated**. AccessiRide centralizes providers and adds a simple, accessible workflow to **search → contact → confirm** — saving time and reducing stress.

---

## Key features

- **Use my location** → instantly find providers near you  
- **Filter by accessibility** (wheelchair vans, lifts, tie-downs, seat belts)  
- **One-tap Call** or **Request via Email**  
- **Clear status flow**: _Checking… → In progress → Confirmed/ETA_  
- **Assistive tech–friendly UI**: large tap targets, high contrast, voice/TTS support  
- **AI assistant** (prototype): guides the user and triggers requests

---

## Mobile-first advantages

- **True portability**: always one tap away on the phone  
- **Direct hardware integration**: high-accuracy GPS and one-tap calling  
- **Focused, distraction-free UI**  
- **Future-ready**: push notifications, offline saved trips, background updates

---

## Architecture

```bash
Client (Expo React Native)         Backend (API)
+--------------------------+       +---------------------------------+
| AccessiRideApp           |       |                                 |
| - Tabs & screens         |----POST--> /email_cab (send request)    |
| - GPS, TTS, Voice        |<---GET---- /status (poll status)        |
| - Call providers         |       |                                 |
+--------------------------+       | + (planned) /providers/* |
                                   +---------------------------------+
```

---

**Repo layout (core)**

- `AccessiRideApp/` — Expo React Native app (tabs, components, hooks)
- `assets/` — icons, images, etc.
- (Backend lives separately; sample endpoints documented below)

---

## Tech stack

- **Mobile:** Expo (React Native), `expo-router`, React Navigation  
- **Integrations:** `expo-location`, `react-native-maps`, `expo-speech`, `react-native-voice`, `@react-native-async-storage/async-storage`  
- **Backend:** simple HTTP API for provider contact & status (email-based flow)  
- **Data:** provider directory (CSV/Excel) + future sync/verification fields

---

## Getting started

### 1) Mobile app (Expo)

**Prereqs**

- Node 18+ and npm  
- Expo CLI (`npm i -g expo`)  
- iOS Simulator / Android Emulator set up (or Expo Go on your device)

**Run**

```bash
git clone <this-repo>
cd AccessiRideApp
npm install
npx expo start
# press i (iOS) or a (Android) or scan QR with Expo Go
```

**Environment**

- GOOGLE_MAPS_API_KEY=xxxx
- EMAIL_SERVICE_API_KEY=xxxx
- BACKEND_URL=http://127.0.0.1:5000

---

### 2) Backend API

Minimal prototype used during the hackathon. Run your local server at `http://127.0.0.1:5000`.

#### Send request to a provider

```bash
curl --location 'http://127.0.0.1:5000/email_cab' \
--header 'Content-Type: application/json' \
--data-raw '{
  "to": "provider@example.com",
  "source": "Sennot Hall, Pittsburgh",
  "destination": "Mervis, Pittsburgh",
  "cab_company": "DevDash accessible cabs"
}'
```

#### Check request statusCheck request status
- curl 'http://127.0.0.1:5000/status?EmailFrom=provider@example.com'

#### Frontend fetch examples (JS)

```bash
// Send
await fetch(`${BACKEND_URL}/email_cab`, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    to: 'provider@example.com',
    source: 'Sennot Hall, Pittsburgh',
    destination: 'Mervis, Pittsburgh',
    cab_company: 'DevDash accessible cabs'
  })
});

// Poll
const res = await fetch(`${BACKEND_URL}/status?EmailFrom=provider@example.com`);
const status = await res.text();
```
**Hardening to add (recommended)**

- Token auth for client requests
- Rate limiting and input validation
- Move provider emails & secrets to env/secure storage

---

### 3) Web app
The public demo site is deployed at Netlify:
https://boisterous-lolly-461c30.netlify.app/

---

## API Reference
| Endpoint | Method | Body | Description |
| :--- | :--- | :--- | :--- |
| `/email_cab` | `POST` | `{ to, source, destination, cab_company }` | Sends an email to the provider to initiate a ride request. |
| `/status?emailFrom=<email>` | `GET` | — | Returns the latest status for the request thread. |

### Planned

* `GET /providers/search?lat&lng&radius&wheelchair=true&same_day=true`
* `GET /providers/:id`
* `POST /requests` → returns request ID; status lifecycle: `sent` → `in_progress` → `confirmed`/`cancelled`

---

## Data

- Source spreadsheet: Accessible Transportation.xlsx (normalized to CSV during import).
- Suggested canonical fields:
  - name, region_served, phone, email, booking_method, vehicle_features (lift, tie_downs, belts), wheelchair_supported, hours, notice_required, last_verified
- Add a data freshness indicator in the UI (last verified date).

---

## Accessibility

We designed for access-first:

- High-contrast themes & reduced motion support
- Large tap targets and consistent layouts
- Voice input for addresses; Text-to-Speech confirmations
- “One-tap Call” for minimal effort actions
- Clear progress states with haptic/aural feedback

Please open issues for additional accessibility improvements — we prioritize them.

---

## Roadmap

- Provider directory service with verification workflow
- In-app saved places and recent requests
- Push notifications: “Your accessible cab is 5 minutes away”
- Offline access for saved providers and past trips
- Multi-city coverage & community-verified updates
- Privacy & security review (PII minimization, audit logs)

---

## Contributing

Contributions are welcome!

- Open an issue for bugs, features, or accessibility requests.
- For PRs, include screenshots when changing UI and note any accessibility impact.

Local dev tips

- Don’t commit secrets or node_modules/
- Use feature branches and small, focused PRs

---

## Acknowledgements

- **SmartTech @ Pitt** — Hackathon organizers
- **Challenge #9:** Accessible Rides
- **Mentor:** Monica Chupko — for lived experience, guidance, and validation
- **Team DevDash** — design, frontend, backend, and integration work

---

## Maintainers / Contact

- **Project:** AccessiRide (DevDash)
- **Issues:** please file through GitHub Issues
