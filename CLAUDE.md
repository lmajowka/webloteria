# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Web Loteria is a client-side Bitcoin address finder that generates random private keys and checks them against target puzzle addresses. It's a single-file HTML application (index.html) with no build system, no package manager, and no external dependencies at build time. Deployed as a static site via GitHub Pages at https://lmajowka.github.io/webloteria/.

## Development

There is no build step, no linting, and no test suite. The entire application lives in `index.html` (~850 lines including embedded minified libraries). To develop, open the file directly in a browser or serve it with any static file server.

## Architecture

**Single-file structure** — `index.html` contains everything:
- Inline CSS (neon cyberpunk theme with CSS variables)
- Embedded minified libraries: **elliptic.js** (secp256k1 curve) and **CryptoJS** (SHA-256, RIPEMD-160)
- Application JavaScript (unminified, at the bottom)

**Core flow:**
1. User selects a wallet (puzzle number) from dropdown and clicks "Jogar" (Play)
2. `start()` → `loop()` initializes wallet config (target address + key range)
3. `processNextBatch()` generates 1000 random keys per batch via `requestAnimationFrame`
4. Each key: `generateRandomNumber()` (using `crypto.getRandomValues`) → `generateAddress()` (secp256k1 → SHA-256 → RIPEMD-160 → Base58Check) → compare with target
5. Match found: `saveFoundKey()` persists to localStorage, `sendPuzzleResult()` POSTs to external API
6. User clicks "Parar" (Stop) to halt

**Key functions:** `generateRandomNumber(min, max)`, `generateAddress(privateKeyInt)`, `hexToBase58(hex)`, `saveFoundKey()`, `sendPuzzleResult()`, `loop()`, `processNextBatch()`

**State:** All persistence uses browser `localStorage` under the key `foundKeys` (JSON array of `{walletId, address, privateKey, timestamp}`).

**External API:** `POST https://bitcoinpuzzles.io/api/puzzle-results` — reports found keys with `{puzzleNumber, reducedPoolToken, privateKey}`.

## Conventions

- UI text is in **Portuguese** (button labels, log messages, table headers)
- Wallet addresses and key ranges are hardcoded in the `loop()` function's wallet initialization
- Console logging uses `log(message)` which appends HTML to `#console` div; supports color classes: `warn`, `error`, and inline `style="color: green"` for success
- DOM IDs: `wallet-dropdown`, `reduced-token`, `start`, `stop-btn`, `console`, `found-keys-list`
