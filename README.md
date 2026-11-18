# AccessiRide (DevDash)

**First place, Challenge #9: Accessible Rides.** Our five-person team built AccessiRide at [Pitt’s 2025 Wireless Innovation Hackathon for Accessibility](https://www.smarttech.pitt.edu/news/2025hackathon). The mobile prototype helps wheelchair users plan a trip and call a Pittsburgh transportation provider. The team also demonstrated automated provider inquiries in a separate web prototype.

> _“As a motorized wheelchair user, I imagine an app that uses my location to show companies with wheelchair vans, lifts, tie-downs, and seat belts — with hours, fees, and notice required.”_ — Mentor: **Monica Chupko**

## Table of contents
- [Demo](#demo)
- [Why this matters](#why-this-matters)
- [Key features](#key-features)
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

A route can tell a rider how to reach a destination. For someone traveling with a wheelchair, taking that trip can also depend on a provider having a suitable vehicle at the right time. Our mentor described having to look up equipment, hours and fees across multiple companies. We explored how much of that work an app could take off the rider’s hands.

---

## Key features

- **Plan a trip.** Use the phone’s current location or select an address with Google Places. Choose transit, driving or cycling to see step-by-step directions. Transit searches use the selected departure time.
- **Save a trip on the device.** Save origin and destination addresses, tap a saved trip to refill the form, or remove it. The app also remembers preferences for steps, curb cuts and rest stops. Route searches do not currently filter on those preferences.
- **Hear confirmations.** The app announces a route search and speaks when the rider saves a trip or changes a travel preference.
- **Contact a provider.** The phone button opens the system dialer for a listed company. Riders can call to confirm equipment, availability and fare.

### What the demos show

| Feature | Data source and behavior |
| --- | --- |
| Mobile address lookup and directions | Google Places supplies address suggestions and coordinates. Google Directions supplies the steps, distance and duration. |
| Mobile cab options | Four preset Pittsburgh providers. The app generates sample fares and pickup times locally. Selecting a sample simulates booking. |
| Other fares on mobile | The transit fare is a fixed sample. Ride-share estimates use the route distance and a local formula. |
| Web provider inquiries | The team tested phone and email inquiries in the separate web prototype. In an email test, the assistant asked for a missing fare and clarified a pickup time before recording the reply. |

---

## Architecture

The [mobile screen](AccessiRideApp/app/%28tabs%29/index.tsx) calls Google Places and Directions. It uses the device for location, speech, saved addresses, preferences and dialing. The cab demo runs locally.

The web prototype used a separate backend for provider inquiries. The `/email_cab` and `/status` examples below describe that workflow. This repository contains the Expo mobile app. The web client and Python server are hosted or maintained separately.

---

## Tech stack

- **Mobile:** TypeScript, Expo React Native and Expo Router
- **Location and routes:** Google Places, Google Directions, `expo-location` and `react-native-maps`
- **Speech and storage:** `expo-speech` and AsyncStorage

---

## Getting started

### 1) Mobile app (Expo)

From a terminal with Node.js and npm installed:

```bash
git clone https://github.com/mic175/AccessiRide.git
cd AccessiRide/AccessiRideApp
npm install
cp .env.example .env.local
```

On Windows PowerShell, use `Copy-Item .env.example .env.local`. Fill in the values for the environment you will run:

| Variable | Use |
| --- | --- |
| `EXPO_PUBLIC_GOOGLE_MAPS_API_KEY` | Address autocomplete, place details and route requests. The prototype uses the Places API and Directions API legacy web-service endpoints. |
| `GOOGLE_MAPS_IOS_API_KEY` | Google Maps SDK for iOS in custom native builds. Restrict the key to the iOS app’s bundle identifier and SDK. |
| `GOOGLE_MAPS_ANDROID_API_KEY` | Google Maps SDK for Android in custom native builds. Restrict the key to the Android package, signing certificate and SDK. |

Start the development server with `npx expo start`. Rebuild a custom native app after changing its SDK key. The local environment file is ignored by Git. Expo embeds public variables in the client bundle. Limit the web-service key to the required APIs and use a backend proxy for production web-service calls. See [Google Maps Platform’s guidance](https://developers.google.com/maps/api-security-best-practices) and [Expo’s environment variable documentation](https://docs.expo.dev/guides/environment-variables/).

---

### 2) Backend API

These examples assume you have the separate hackathon backend running at `http://127.0.0.1:5000`. The mobile app can run independently.

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

#### Check request status
- curl 'http://127.0.0.1:5000/status?EmailFrom=provider@example.com'

#### Web frontend fetch examples (JS)

```js
const BACKEND_URL = 'http://127.0.0.1:5000';
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
| `/email_cab` | `POST` | `{ to, source, destination, cab_company }` | Asks a provider about a trip’s availability, pickup time and fare. |
| `/status?EmailFrom=<email>` | `GET` | — | Returns the latest status for the request thread. |

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

The mobile app offers light and dark themes, spoken confirmations and direct dialing. Riders can record needs such as curb cuts in their profile. A provider can confirm whether its vehicle will accommodate a particular wheelchair.

---

## Roadmap

- Connect the mobile cab screen to provider replies for actual quotes and pickup times.
- Verify provider equipment and service areas, then use those records to filter results.
- Keep a history of provider inquiries and notify the rider when a reply arrives.
- Expand coverage beyond Pittsburgh and support community-verified updates.

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
