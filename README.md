# Dairy Ledger

A lightweight, installable web app for running the day-to-day records of a small dairy farm — milk supply to clients, herd production, expenses, and full per-animal health & breeding tracking.

**Live app:** https://aryanthakur20021-byte.github.io/dairy-ledger/

## Features

- **Milk supply & billing** — log daily supply per client, track dues, print statements.
- **Herd production** — record morning/evening yield per animal.
- **Expenses** — feed, labour, and other costs with monthly summaries.
- **Herd health & breeding log** — per animal: artificial insemination, heat cycles, pregnancy checks with due-date estimates (cow vs. buffalo gestation), deworming, vaccination, and calving history, with a calf-to-mother lineage link and Milking / Dry / Heifer status.
- **Reminders** — a Home screen card surfaces upcoming/overdue pregnancy checks, vaccinations, deworming, expected calving, and expected heat.
- **Works fully offline** — data is stored locally in the browser (local-first), so the app works with no internet connection and installs to your home screen like a native app (PWA).
- **Optional account sync** — sign in to sync data across devices via Firebase; guest mode (no sign-in) never touches the network.

## Tech

Single-file vanilla HTML/CSS/JavaScript — no build step, no framework, no dependencies for guest use. A service worker (`sw.js`) caches the app for offline use and installability.

## Deployment

Hosted on GitHub Pages, served directly from `main`. Pushing to `main` updates the live site (the service worker's cache version in `sw.js` should be bumped on any meaningful update, so installed devices pick up the change cleanly).
