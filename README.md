# Gaelic Football Stats App

A mobile-first, frontend-only Gaelic football live match tracking app built with React and Vite.

## What it includes

- Match setup for Team A and Team B
- Live score tracking for both teams
- Tackles counter for both teams
- Kickout logging with:
  - kicking team
  - long / short
  - possession winner
- Attack logging with:
  - attacking team
  - lost possession / shot / recycle
  - if shot: score / wide / drop short
  - if score: point / goal
- Automatic scoring attribution to the correct team
- Derived percentages:
  - attacks that become shots
  - shots that become scores
  - kickout win rate
- LocalStorage persistence
- Offline support via service worker
- Plain-text match summary copy/export

## Run locally

```bash
npm install
npm run dev
```

## Build for production

```bash
npm run build
```

## Deploy on Netlify

- Push this folder to GitHub
- Create a new Netlify site from that repo
- Build command: `npm run build`
- Publish directory: `dist`

## Notes

- Data is stored only in the browser
- No backend or user accounts are required
- The service worker provides lightweight offline support for static usage
