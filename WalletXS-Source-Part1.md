# WalletXS — GitHub Source Bundle (Part 1 of 3)

Full source for the **WalletXS** wallet-exposure-score app. 100% client-side and stateless: no backend, no database, no accounts, no cookies. All persistence lives in `localStorage` (mirrored to `IndexedDB` for service-worker access). The score is computed fresh in the browser on every check from public RPC + public blocklist data.

> **Part 1** = config / build files / public assets / hooks / lib. **Part 2** = components. **Part 3** = pages + dependencies. The only external dependencies are standard shadcn/ui primitives (`Button`, `Input`, `Dialog`) and the npm packages listed at the end of Part 3.

## File structure

```
WalletXS/
├─ index.html
├─ tailwind.config.js
├─ public/
│  ├─ manifest.json
│  ├─ sw.js
│  └─ icon.svg
└─ src/
   ├─ main.jsx
   ├─ App.jsx
   ├─ index.css
   ├─ hooks/
   │  ├─ useInstallPrompt.js
   │  ├─ useLiveMetrics.js
   │  └─ useScheduledCheck.js
   ├─ lib/
   │  ├─ config.js
   │  ├─ scoring.js
   │  ├─ chain.js
   │  ├─ storage.js
   │  ├─ syncdb.js
   │  ├─ backup.js
   │  ├─ percentile.js
   │  ├─ percentileSample.json
   │  ├─ severity.js
   │  └─ notify.js
   ├─ components/walletxs/   (see Part 2)
   └─ pages/                 (see Part 2)
```

---

## index.html

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <link rel="icon" type="image/svg+xml" href="/icon.svg" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <link rel="manifest" href="/manifest.json" />
    <link rel="apple-touch-icon" href="/icon.svg" />
    <meta name="theme-color" content="#38bdf8" />
    <meta name="description" content="WalletXS — live wallet exposure score from real on-chain and public data, computed fresh in your browser each check." />
    <title>WalletXS — Wallet Exposure Score</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.jsx"></script>
  </body>
</html>
```

## tailwind.config.js

```js
/** @type {import('tailwindcss').Config} */
module.exports = {
    darkMode: ["class"],
    content: ["./index.html", "./src/**/*.{ts,tsx,js,jsx}"],
  theme: {
  	extend: {
  		borderRadius: {
  			lg: 'var(--radius)',
  			md: 'calc(var(--radius) - 2px)',
  			sm: 'calc(var(--radius) - 4px)'
  		},
  		colors: {
  			background: 'hsl(var(--background))',
  			foreground: 'hsl(var(--foreground))',
  			card: {
  				DEFAULT: 'hsl(var(--card))',
  				foreground: 'hsl(var(--card-foreground))'
  			},
  			popover: {
  				DEFAULT: 'hsl(var(--popover))',
  				foreground: 'hsl(var(--popover-foreground))'
  			},
  			primary: {
  				DEFAULT: 'hsl(var(--primary))',
  				foreground: 'hsl(var(--primary-foreground))'
  			},
  			secondary: {
  				DEFAULT: 'hsl(var(--secondary))',
  				foreground: 'hsl(var(--secondary-foreground))'
  			},
  			muted: {
  				DEFAULT: 'hsl(var(--muted))',
  				foreground: 'hsl(var(--muted-foreground))'
  			},
  			accent: {
  				DEFAULT: 'hsl(var(--accent))',
  				foreground: 'hsl(var(--accent-foreground))'
  			},
  			destructive: {
  				DEFAULT: 'hsl(var(--destructive))',
  				foreground: 'hsl(var(--destructive-foreground))'
  			},
  			border: 'hsl(var(--border))',
  			input: 'hsl(var(--input))',
  			ring: 'hsl(var(--ring))',
  			chart: {
  				'1': 'hsl(var(--chart-1))',
  				'2': 'hsl(var(--chart-2))',
  				'3': 'hsl(var(--chart-3))',
  				'4': 'hsl(var(--chart-4))',
  				'5': 'hsl(var(--chart-5))'
  			},
  			sidebar: {
  				DEFAULT: 'hsl(var(--sidebar-background))',
  				foreground: 'hsl(var(--sidebar-foreground))',
  				primary: 'hsl(var(--sidebar-primary))',
  				'primary-foreground': 'hsl(var(--sidebar-primary-foreground))',
  				accent: 'hsl(var(--sidebar-accent))',
  				'accent-foreground': 'hsl(var(--sidebar-accent-foreground))',
  				border: 'hsl(var(--sidebar-border))',
  				ring: 'hsl(var(--sidebar-ring))'
  			}
  		},
  		fontFamily: {
  			heading: ['var(--font-heading)'],
  			body: ['var(--font-body)'],
  			display: ['var(--font-display)'],
  			mono: ['var(--font-mono)']
  		},
  		keyframes: {
  			'accordion-down': {
  				from: {
  					height: '0'
  				},
  				to: {
  					height: 'var(--radix-accordion-content-height)'
  				}
  			},
  			'accordion-up': {
  				from: {
  					height: 'var(--radix-accordion-content-height)'
  				},
  				to: {
  					height: '0'
  				}
  			}
  		},
  		animation: {
  			'accordion-down': 'accordion-down 0.2s ease-out',
  			'accordion-up': 'accordion-up 0.2s ease-out'
  		}
  	}
  },
  plugins: [require("tailwindcss-animate")],
}
```

## public/manifest.json

```json
{
  "name": "WalletXS",
  "short_name": "WalletXS",
  "description": "Live wallet exposure score from real on-chain and public data, computed fresh in your browser each check.",
  "display": "standalone",
  "background_color": "#0b0f19",
  "theme_color": "#38bdf8",
  "orientation": "portrait",
  "start_url": "/",
  "scope": "/",
  "icons": [
    { "src": "/icon.svg", "sizes": "192x192", "type": "image/svg+xml", "purpose": "any" },
    { "src": "/icon.svg", "sizes": "512x512", "type": "image/svg+xml", "purpose": "any" },
    { "src": "/icon.svg", "sizes": "512x512", "type": "image/svg+xml", "purpose": "maskable" }
  ]
}
```

## public/icon.svg

```xml
<svg xmlns="http://www.w3.org/2000/svg" width="512" height="512" viewBox="0 0 512 512">
  <rect width="512" height="512" rx="96" fill="#0b0f19" />
  <path d="M256 88 L392 132 V256 C392 350 332 402 256 430 C180 402 120 350 120 256 V132 Z" fill="#0b0f19" stroke="#38bdf8" stroke-width="20" stroke-linejoin="round" />
  <path d="M206 256 L242 292 L312 214" fill="none" stroke="#38bdf8" stroke-width="26" stroke-linecap="round" stroke-linejoin="round" />
  <text x="256" y="392" text-anchor="middle" font-family="ui-sans-serif, system-ui, sans-serif" font-size="52" font-weight="700" fill="#38bdf8">XS</text>
</svg>
```

## public/sw.js

```js
/**
 * WalletXS service worker.
 *
 * 1) Network-first app-shell caching for offline use.
 * 2) Periodic Background Sync: a stateless daily background recheck. The SW
 *    wakes up (app fully closed) ~once/day, reads the saved wallet + Etherscan
 *    key from IndexedDB, refetches the wallet's active approval surface from
 *    Etherscan, and if it changed since the last background check it shows a
 *    local notification. No push server, no server-side state — the only
 *    stored data is a single approval-signature string in IndexedDB.
 *
 * Limitations (inherent to Periodic Background Sync):
 *   - Chromium/Android only (no iOS Safari).
 *   - The browser — not the app — decides the actual interval based on site
 *     engagement; it is "roughly daily", not guaranteed exactly 24h.
 *   - Requires the PWA to be installed and notifications to be enabled.
 *   - Requires a saved Etherscan API key (the keyless RPC fallback is too
 *     heavy/rate-limited to run reliably inside a background SW wake-up).
 */

const CACHE = 'walletxs-shell-v2';
const SYNC_TAG = 'walletxs-daily';

// --- Etherscan approval discovery (mirrors src/lib/chain.js, minimized) -----
const TOPIC_APPROVAL =
  '0x8c5be1e5ebec7d5bd14f71427d1e84f3dd0314c0f7b2291e5b200ac8c7c3b925';
const TOPIC_APPROVAL_FOR_ALL =
  '0x17307eab39ab6107e8899845ad3d59bd9653f200f220920489ca2b5937696c31';
const UNLIMITED_THRESHOLD = 2n ** 255n;
const ETHERSCAN_BASE = 'https://api.etherscan.io/v2/api';
const BLOCKLIST_URL =
  'https://raw.githubusercontent.com/scamsniffer/scam-database/main/blacklist/address.json';
const MAX_PAGES = 10;

const pad32 = (a) => '0x' + '0'.repeat(24) + a.toLowerCase().replace(/^0x/, '');
const addrFromTopic = (t) => (t && t.length >= 66 ? '0x' + t.slice(26).toLowerCase() : '');

async function fetchBlocklist() {
  try {
    const res = await fetch(BLOCKLIST_URL);
    if (!res.ok) return null;
    const list = await res.json();
    const arr = Array.isArray(list) ? list : Object.keys(list);
    return new Set(arr.map((a) => String(a).toLowerCase()));
  } catch {
    return null;
  }
}

async function etherscanLogs(apiKey, topic0, topic1) {
  const all = [];
  const params = new URLSearchParams({
    chainid: '1',
    module: 'logs',
    action: 'getLogs',
    fromBlock: '0',
    toBlock: 'latest',
    topic0,
    topic1,
    offset: '1000',
    apikey: apiKey,
  });
  for (let page = 1; page <= MAX_PAGES; page++) {
    params.set('page', String(page));
    const res = await fetch(`${ETHERSCAN_BASE}?${params.toString()}`);
    const json = await res.json();
    if (json.status === '0') {
      if (json.message === 'No records found') break;
      throw new Error(typeof json.result === 'string' ? json.result : json.message || 'Etherscan error');
    }
    const rows = Array.isArray(json.result) ? json.result : [];
    all.push(...rows);
    if (rows.length < 1000) break;
  }
  return all;
}

// A stable signature of the wallet's ACTIVE approval surface. Changes only
// when an approval is granted/revoked, an unlimited flag flips, or a spender
// gets newly flagged on the blocklist — i.e. exactly what moves the score.
async function approvalSignature(address, apiKey) {
  const blocklist = await fetchBlocklist();
  const blocklistAvailable = blocklist !== null;
  const owner = pad32(address);
  const [a, b] = await Promise.all([
    etherscanLogs(apiKey, TOPIC_APPROVAL, owner),
    etherscanLogs(apiKey, TOPIC_APPROVAL_FOR_ALL, owner).catch(() => []),
  ]);
  const logs = [...a, ...b];

  // Latest log per (token, spender) defines current state.
  const byPair = new Map();
  for (const log of logs) {
    if (!log.topics || log.topics.length < 3) continue;
    const token = log.address.toLowerCase();
    const spender = addrFromTopic(log.topics[2]);
    if (!spender) continue;
    const key = `${token}:${spender}`;
    const bn = parseInt(log.blockNumber, 16);
    const prev = byPair.get(key);
    if (!prev || bn > prev.bn) byPair.set(key, { log, token, spender, bn });
  }

  const parts = [];
  for (const e of byPair.values()) {
    const isForAll = e.log.topics[0] === TOPIC_APPROVAL_FOR_ALL;
    let active;
    let unlimited;
    if (isForAll) {
      const approvedTopic = e.log.topics[3];
      active = !!approvedTopic && approvedTopic.toLowerCase().endsWith('1');
      unlimited = active;
    } else {
      let value = 0n;
      try {
        value = BigInt(e.log.data && e.log.data !== '0x' ? e.log.data.slice(0, 66) : '0x0');
      } catch {
        value = 0n;
      }
      active = value > 0n;
      unlimited = value >= UNLIMITED_THRESHOLD;
    }
    if (!active) continue;
    const flagged = blocklistAvailable && blocklist.has(e.spender);
    parts.push(`${e.token}:${e.spender}:${unlimited ? 1 : 0}:${flagged ? 1 : 0}`);
  }
  parts.sort();
  return parts.join('|');
}

// --- IndexedDB (shared with the page via src/lib/syncdb.js) ----------------
const SYNC_DB = 'walletxs-sync';
const SYNC_STORE = 'kv';
function openSyncDB() {
  return new Promise((resolve, reject) => {
    const req = indexedDB.open(SYNC_DB, 1);
    req.onupgradeneeded = () => req.result.createObjectStore(SYNC_STORE, { keyPath: 'k' });
    req.onsuccess = () => resolve(req.result);
    req.onerror = () => reject(req.error);
  });
}
async function idbGet(k) {
  try {
    const db = await openSyncDB();
    return await new Promise((resolve) => {
      const tx = db.transaction(SYNC_STORE, 'readonly');
      const req = tx.objectStore(SYNC_STORE).get(k);
      req.onsuccess = () => resolve(req.result ? req.result.v : null);
      req.onerror = () => resolve(null);
      tx.oncomplete = () => db.close();
    });
  } catch {
    return null;
  }
}
async function idbSet(k, v) {
  try {
    const db = await openSyncDB();
    await new Promise((resolve) => {
      const tx = db.transaction(SYNC_STORE, 'readwrite');
      tx.objectStore(SYNC_STORE).put({ k, v });
      tx.oncomplete = () => { db.close(); resolve(); };
      tx.onerror = () => resolve();
    });
  } catch {
    /* noop */
  }
}

// --- SW lifecycle ---------------------------------------------------------
self.addEventListener('install', () => {
  self.skipWaiting();
});

self.addEventListener('activate', (e) => {
  e.waitUntil(
    (async () => {
      const keys = await caches.keys();
      await Promise.all(keys.filter((k) => k !== CACHE).map((k) => caches.delete(k)));
      await self.clients.claim();
    })()
  );
});

// Network-first for the app's own static shell only. Cross-origin GETs
// (Etherscan, the RPC provider, ScamSniffer blocklist, OFAC list, CoinGecko)
// must always hit the network live and be allowed to fail naturally — never
// served from cache. Caching them would (a) mask stale data behind a cached
// response on a live fetch failure, defeating the app's "Unavailable"
// fallback, and (b) persist a local record of queried addresses (the Etherscan
// URL carries the wallet address as a query param) in Cache Storage.
self.addEventListener('fetch', (e) => {
  const req = e.request;
  if (req.method !== 'GET') return;
  const url = new URL(req.url);
  if (url.origin !== self.location.origin) return;
  e.respondWith(
    (async () => {
      try {
        const fresh = await fetch(req);
        const cache = await caches.open(CACHE);
        cache.put(req, fresh.clone());
        return fresh;
      } catch {
        const cached = await caches.match(req);
        if (cached) return cached;
        if (req.mode === 'navigate') {
          const fallback = await caches.match('/');
          if (fallback) return fallback;
        }
        throw new Error('offline and not cached');
      }
    })()
  );
});

// --- Periodic Background Sync: stateless daily recheck --------------------
self.addEventListener('periodicsync', (e) => {
  if (e.tag !== SYNC_TAG) return;
  e.waitUntil(
    (async () => {
      const address = await idbGet('address');
      if (!address) return;
      const apiKey = await idbGet('apiKey');
      if (!apiKey) return; // background full-depth check needs an Etherscan key
      // Only notify if the user has enabled alerts.
      if (typeof Notification === 'undefined' || Notification.permission !== 'granted') return;

      let sig;
      try {
        sig = await approvalSignature(address, apiKey);
      } catch {
        return; // network/Etherscan hiccup — try again next cycle
      }
      const lastSig = await idbGet('lastSig');
      await idbSet('lastSig', sig);
      if (lastSig === null) return; // first run — establish baseline silently
      if (sig === lastSig) return;

      const short = `${address.slice(0, 6)}…${address.slice(-4)}`;
      self.registration.showNotification('WalletXS exposure changed', {
        body: `${short} — your approval surface changed since the last check. Open WalletXS to see your updated score.`,
        tag: `walletxs-${address}`,
        data: { url: '/' },
      });
    })()
  );
});

// Tap the notification → focus an open WalletXS tab, or open one.
self.addEventListener('notificationclick', (e) => {
  e.notification.close();
  e.waitUntil(
    (async () => {
      const all = await clients.matchAll({ type: 'window', includeUncontrolled: true });
      for (const c of all) {
        if ('focus' in c) {
          await c.focus();
          return;
        }
      }
      if (clients.openWindow) await clients.openWindow(e.notification.data?.url || '/');
    })()
  );
});
```

## src/main.jsx

```jsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import App from '@/App.jsx'
import '@/index.css'

// Register the offline app-shell service worker (network-first, no push/sync).
// When an updated SW installs, reload once so the new code takes over immediately.
if ('serviceWorker' in navigator) {
  window.addEventListener('load', () => {
    navigator.serviceWorker
      .register('/sw.js')
      .then((reg) => {
        reg.addEventListener('updatefound', () => {
          const nw = reg.installing;
          if (!nw) return;
          nw.addEventListener('statechange', () => {
            if (nw.state === 'installed' && navigator.serviceWorker.controller) {
              window.location.reload();
            }
          });
        });
      })
      .catch(() => {});

    // Register a stateless daily background recheck via Periodic Background Sync
    // (Chromium/Android only; the browser decides the actual interval). The SW
    // rechecks the approval surface and notifies on a change — no push server.
    navigator.serviceWorker.ready
      .then((reg) => {
        if ('periodicSync' in reg) {
          reg.periodicSync
            .register('walletxs-daily', { minInterval: 24 * 60 * 60 * 1000 })
            .catch(() => {});
        }
      })
      .catch(() => {});
  });
}

ReactDOM.createRoot(document.getElementById('root')).render(
  <App />
)
```

## src/App.jsx

```jsx
import { Toaster } from "@/components/ui/toaster"
import { QueryClientProvider } from '@tanstack/react-query'
import { queryClientInstance } from '@/lib/query-client'
import { BrowserRouter as Router, Route, Routes } from 'react-router-dom';
import PageNotFound from './lib/PageNotFound';
import { AuthProvider, useAuth } from '@/lib/AuthContext';
import UserNotRegisteredError from '@/components/UserNotRegisteredError';
import ScrollToTop from './components/ScrollToTop';
import Home from './pages/Home';
import Methodology from './pages/Methodology';
import PrivacyPolicy from './pages/PrivacyPolicy';

// WalletXS is fully public — zero accounts, ever. AuthProvider is a required
// platform wrapper but is inert here: no loading gate on auth state and no
// login redirect of any kind. Routes render directly.
const AuthenticatedApp = () => {
  const { authError } = useAuth();
  if (authError && authError.type === 'user_not_registered') {
    return <UserNotRegisteredError />;
  }
  return (
    <Routes>
      <Route path="/" element={<Home />} />
      <Route path="/methodology" element={<Methodology />} />
      <Route path="/privacy" element={<PrivacyPolicy />} />
      <Route path="*" element={<PageNotFound />} />
    </Routes>
  );
};

function App() {
  return (
    <AuthProvider>
      <QueryClientProvider client={queryClientInstance}>
        <Router>
          <ScrollToTop />
          <AuthenticatedApp />
        </Router>
        <Toaster />
      </QueryClientProvider>
    </AuthProvider>
  )
}

export default App
```

## src/index.css

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  :root {
    --background: 0 0% 100%;
    --foreground: 0 0% 3.9%;
    --card: 0 0% 100%;
    --card-foreground: 0 0% 3.9%;
    --popover: 0 0% 100%;
    --popover-foreground: 0 0% 3.9%;
    --primary: 0 0% 9%;
    --primary-foreground: 0 0% 98%;
    --secondary: 0 0% 96.1%;
    --secondary-foreground: 0 0% 9%;
    --muted: 0 0% 96.1%;
    --muted-foreground: 0 0% 45.1%;
    --accent: 0 0% 96.1%;
    --accent-foreground: 0 0% 9%;
    --destructive: 0 84.2% 60.2%;
    --destructive-foreground: 0 0% 98%;
    --border: 0 0% 89.8%;
    --input: 0 0% 89.8%;
    --ring: 0 0% 3.9%;
    --chart-1: 12 76% 61%;
    --chart-2: 173 58% 39%;
    --chart-3: 197 37% 24%;
    --chart-4: 43 74% 66%;
    --chart-5: 27 87% 67%;
    --radius: 0.5rem;
    --font-heading: ui-sans-serif, system-ui, sans-serif;
    --font-body: ui-sans-serif, system-ui, sans-serif;
    --font-display: ui-sans-serif, system-ui, sans-serif;
    --font-mono: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace;
    --sidebar-background: 0 0% 98%;
    --sidebar-foreground: 240 5.3% 26.1%;
    --sidebar-primary: 240 5.9% 10%;
    --sidebar-primary-foreground: 0 0% 98%;
    --sidebar-accent: 240 4.8% 95.9%;
    --sidebar-accent-foreground: 240 5.9% 10%;
    --sidebar-border: 220 13% 91%;
    --sidebar-ring: 217.2 91.2% 59.8%;
  }

  .dark {
    --background: 0 0% 3.9%;
    --foreground: 0 0% 98%;
    --card: 0 0% 3.9%;
    --card-foreground: 0 0% 98%;
    --popover: 0 0% 3.9%;
    --popover-foreground: 0 0% 98%;
    --primary: 0 0% 98%;
    --primary-foreground: 0 0% 9%;
    --secondary: 0 0% 14.9%;
    --secondary-foreground: 0 0% 98%;
    --muted: 0 0% 14.9%;
    --muted-foreground: 0 0% 63.9%;
    --accent: 0 0% 14.9%;
    --accent-foreground: 0 0% 98%;
    --destructive: 0 62.8% 30.6%;
    --destructive-foreground: 0 0% 98%;
    --border: 0 0% 14.9%;
    --input: 0 0% 14.9%;
    --ring: 0 0% 83.1%;
    --chart-1: 220 70% 50%;
    --chart-2: 160 60% 45%;
    --chart-3: 30 80% 55%;
    --chart-4: 280 65% 60%;
    --chart-5: 340 75% 55%;
    --sidebar-background: 240 5.9% 10%;
    --sidebar-foreground: 240 4.8% 95.9%;
    --sidebar-primary: 224.3 76.3% 48%;
    --sidebar-primary-foreground: 0 0% 100%;
    --sidebar-accent: 240 3.7% 15.9%;
    --sidebar-accent-foreground: 240 4.8% 95.9%;
    --sidebar-border: 240 3.7% 15.9%;
    --sidebar-ring: 217.2 91.2% 59.8%;
  }
}



@layer base {
  * {
    @apply border-border outline-ring/50;
  }

  body {
    @apply bg-background text-foreground font-body;
  }
}

/* Print / export stylesheet — hides interactive chrome, keeps the report. */
.print-only { display: none; }

@media print {
  body { background: #ffffff !important; }
  .no-print { display: none !important; }
  .print-only { display: block !important; }
  /* Hide all interactive buttons in the print output. */
  button { display: none !important; }
  /* Keep the report's colors (score tiers, borders) in the printout. */
  * { -webkit-print-color-adjust: exact; print-color-adjust: exact; }
}
```

## src/hooks/useInstallPrompt.js

```js
import { useEffect, useState } from 'react';

/**
 * Captures the beforeinstallprompt event so we can trigger the native PWA
 * install dialog on demand. Returns null when install isn't available
 * (already installed, unsupported browser, etc.) — in which case the
 * "Install now" button stays hidden rather than faking a prompt.
 */
export function useInstallPrompt() {
  const [deferred, setDeferred] = useState(null);
  const [installed, setInstalled] = useState(false);

  useEffect(() => {
    const onPrompt = (e) => {
      e.preventDefault();
      setDeferred(e);
    };
    const onInstalled = () => {
      setInstalled(true);
      setDeferred(null);
    };
    window.addEventListener('beforeinstallprompt', onPrompt);
    window.addEventListener('appinstalled', onInstalled);
    return () => {
      window.removeEventListener('beforeinstallprompt', onPrompt);
      window.removeEventListener('appinstalled', onInstalled);
    };
  }, []);

  const prompt = async () => {
    if (!deferred) return false;
    deferred.prompt();
    const { outcome } = await deferred.userChoice;
    setDeferred(null);
    return outcome === 'accepted';
  };

  return { canInstall: !!deferred && !installed, prompt, installed };
}
```

## src/hooks/useLiveMetrics.js

```js
import { useState, useEffect, useCallback } from 'react';

// Live all-time usage total for the hero banner.
//
// Privacy contract (enforced in code):
//   - GET  https://api.walletxs.com/stats   → { totalAudits }
//   - POST https://api.walletxs.com/count   → body "{}", no query params, no
//     cookies (credentials: 'omit'), no wallet address, no IP, no user id.
//
// Only the cumulative all-time total is tracked here — no daily count, since a
// daily figure is volatile and can make the app look unused on a slow day.
//
// If the endpoint is unreachable (ad-blocker, offline, or not yet deployed),
// `available` flips to false so the banner can render nothing.
const STATS_URL = 'https://api.walletxs.com/stats';
const COUNT_URL = 'https://api.walletxs.com/count';

const SAFE_FETCH = { credentials: 'omit', mode: 'cors' };

export function useLiveMetrics() {
  const [totalAudits, setTotalAudits] = useState(null);
  const [available, setAvailable] = useState(true);

  // One read on mount; does not increment anything. Reading the count is not
  // itself a usage event.
  useEffect(() => {
    let cancelled = false;
    fetch(STATS_URL, { ...SAFE_FETCH, method: 'GET' })
      .then((r) => r.json())
      .then((d) => {
        if (cancelled) return;
        if (d && typeof d.totalAudits === 'number') {
          setTotalAudits(d.totalAudits);
        } else {
          setAvailable(false);
        }
      })
      .catch(() => {
        if (!cancelled) setAvailable(false);
      });
    return () => {
      cancelled = true;
    };
  }, []);

  // Fire-and-forget anonymous signal. This is invoked only after a scan
  // actually completes (a real score was produced) — see the guard in Home.jsx.
  // Optimistic local +1 keeps the UI snappy even when the ping is silently
  // blocked; the server response, when it arrives, reconciles to the true
  // global total. If the read never resolved (null), we don't fake a number.
  const increment = useCallback(() => {
    setTotalAudits((n) => (n === null ? n : n + 1));
    fetch(COUNT_URL, {
      ...SAFE_FETCH,
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: '{}',
    })
      .then((r) => r.json())
      .then((d) => {
        if (d && typeof d.totalAudits === 'number') {
          setTotalAudits(d.totalAudits);
        }
      })
      .catch(() => {});
  }, []);

  return { totalAudits, available, increment };
}
```

## src/hooks/useScheduledCheck.js

```js
import { useEffect, useRef } from 'react';

const DAY_MS = 24 * 60 * 60 * 1000;
const TICK_MS = 60 * 60 * 1000; // poll hourly; the actual scan is gated to once/day

// Stateless daily auto-check. No backend, no push tokens — the WalletXS tab
// must be open for a background check to occur. (A check while the app is
// fully closed would require a push server, which we don't run.)
//
// - Opening the app anchors the 24h clock (the on-open scan counts as today's
//   check).
// - While the tab stays open in the background, an hourly tick runs a scan
//   once 24h+ have passed. Because the tab is hidden, a score change fires a
//   browser notification (see notifyScoreChange in lib/notify.js).
// - When the tab becomes visible again after being hidden, if 24h+ have
//   passed it refreshes immediately so a returning user sees a fresh score.
export function useScheduledCheck(address, apiKey, run) {
  const runRef = useRef(run);
  runRef.current = run;
  const keyRef = useRef(apiKey);
  keyRef.current = apiKey;

  useEffect(() => {
    if (!address) return;
    const key = `walletxs.lastAutoCheck.${address.toLowerCase()}`;
    // Anchor the clock to the on-open scan so the first 24h don't auto-check.
    try {
      localStorage.setItem(key, String(Date.now()));
    } catch {
      /* storage unavailable */
    }
    const lastAt = () => Number(localStorage.getItem(key) || Date.now());
    const due = () => Date.now() - lastAt() >= DAY_MS;

    const doCheck = () => {
      if (!due()) return;
      try {
        localStorage.setItem(key, String(Date.now()));
      } catch {
        /* noop */
      }
      runRef.current(address, keyRef.current);
    };

    // Background hourly tick — only while hidden (a visible session already
    // scanned on open and can re-run manually).
    const tick = () => {
      if (document.hidden) doCheck();
    };
    // Refresh on return if stale.
    const onVisible = () => {
      if (!document.hidden) doCheck();
    };

    const id = setInterval(tick, TICK_MS);
    document.addEventListener('visibilitychange', onVisible);
    return () => {
      clearInterval(id);
      document.removeEventListener('visibilitychange', onVisible);
    };
  }, [address]);
}
```

## src/lib/config.js

```js
// WalletXS configurable constants. Edit these values to point the app at your
// own tip address and source repository.

// Public EVM address that receives Tip Jar contributions (Ethereum mainnet).
// Replace with your own address before publishing.
export const DEV_TIP_ADDRESS = '0x4b848Adb6BB9072C7Ef3513D52699AB569291bFf';

// GitHub repository URL for the "Source code" button.
export const GITHUB_REPO_URL = 'https://github.com/officiald2raven-alt/-WalletXS---Source-Code-Bundle.txt';

// Ko-fi page for card/fiat tips (complements the on-chain crypto Tip Jar).
export const KOFI_URL = 'https://ko-fi.com/walletxs';

// Official WalletXS logo (mark + wordmark). Rendered in the header.
export const WALLETXS_LOGO_URL = 'https://media.base44.com/images/public/6a75d38d6ef05e633ef41d17/4fbc7a5e1_generated_image.png';

// OPTIONAL developer Etherscan API key. When set, the app uses it to fetch full
// lifetime approval history for every user (no per-user key needed). It ships in
// the client bundle — this is a developer key, not a user credential, and needs
// no backend. Get a free key at https://etherscan.io/myapikey. Leave empty to
// fall back to a keyless public-RPC scan (~55 days of history).
export const ETHERSCAN_API_KEY = '';

// USDC contract on Ethereum mainnet (6 decimals).
export const USDC_CONTRACT = '0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48';

// The all-time audit counter is hidden below this cumulative total to avoid
// showing a small or volatile number. Once the total crosses it, the counter
// renders and stays visible (the total only ever goes up). Adjust freely.
export const MIN_VISIBLE_COUNT = 250;
```

## src/lib/scoring.js

```js
/**
 * WalletXS scoring engine — pure functions, no UI coupling, no network I/O.
 *
 * DESIGN RULE: no sub-score starts at 100 and only subtracts (or starts at a
 * shared max and only adds). Each sub-score has a mid-range baseline and both
 * positive and negative deltas driven by distinct input conditions, so the full
 * 0-100 range is genuinely reachable and the score can differentiate wallets.
 *
 * Every sub-score returns either a real number or { unavailable: true }.
 * Never substitute a placeholder number for missing data.
 */

export const UNLIMITED_THRESHOLD = (2n ** 255n);

export const WEIGHTS = {
  approvalExposure: 0.35,
  approvalStaleness: 0.2,
  historicalExposure: 0.25,
  publicExposure: 0.2,
};

export const SUB_SCORE_LABELS = {
  approvalExposure: 'Approval exposure',
  approvalStaleness: 'Approval staleness',
  historicalExposure: 'Historical contract exposure',
  publicExposure: 'Public exposure (OSINT)',
};

const clamp = (n) => Math.max(0, Math.min(100, Math.round(n)));

export function categoryFor(score) {
  if (score === null || score === undefined) return null;
  if (score >= 80) return 'Strong';
  if (score >= 60) return 'Fair';
  if (score >= 40) return 'At Risk';
  return 'Critical';
}

/**
 * A. APPROVAL EXPOSURE
 * Baseline 55. Active grants push down (weighted by unlimited-ness and whether
 * the spender is on the blocklist); a genuinely clean approval surface pushes up.
 * Reachable range: ~0 (several unlimited approvals to flagged spenders)
 * to 100 (no active approvals at all).
 */
export function scoreApprovalExposure({ approvals, blocklistAvailable }) {
  if (!approvals || !blocklistAvailable) return { unavailable: true };

  let score = 55;
  const reasons = [];

  if (approvals.length === 0) {
    return { score: 100, reasons: ['No active token approvals found.'] };
  }

  let flagged = 0;
  let unlimited = 0;

  for (const a of approvals) {
    if (a.flagged) flagged += 1;
    if (a.unlimited) unlimited += 1;

    if (a.unlimited && a.flagged) {
      score -= 45;
      reasons.push(`Unlimited approval to flagged contract ${a.spender}.`);
    } else if (a.flagged) {
      score -= 20;
      reasons.push(`Capped approval to flagged contract ${a.spender}.`);
    } else if (a.unlimited) {
      score -= 12;
    } else {
      score -= 3;
    }
  }

  // Positive deltas for real, distinct good conditions.
  if (flagged === 0) score += 12;
  if (unlimited === 0) score += 14;
  if (approvals.length <= 3) score += 8;

  return { score: clamp(score), reasons };
}

/**
 * B. APPROVAL STALENESS
 * Baseline 60. Long-unused but still-active approvals push down hardest when
 * unlimited; recently-exercised or short-lived approvals push up.
 */
export function scoreApprovalStaleness({ approvals, now }) {
  if (!approvals) return { unavailable: true };
  if (approvals.length === 0) {
    return { score: 100, reasons: ['No active approvals to go stale.'] };
  }
  if (approvals.every((a) => a.lastActivityAt == null)) return { unavailable: true };

  let score = 60;
  const reasons = [];
  let stale = 0;
  let fresh = 0;

  for (const a of approvals) {
    if (a.lastActivityAt == null) continue;
    const days = (now - a.lastActivityAt) / 86400000;
    const ageDays = a.grantedAt != null ? (now - a.grantedAt) / 86400000 : days;

    if (days > 365) {
      stale += 1;
      score -= a.unlimited ? 30 : 12;
      reasons.push(
        `Approval to ${a.spender} unused for ${Math.round(days)} days and still active.`
      );
    } else if (days > 180) {
      stale += 1;
      score -= a.unlimited ? 14 : 6;
    } else if (days > 30) {
      score -= a.unlimited ? 5 : 2;
    } else {
      fresh += 1;
      score += 6;
    }

    // An approval granted long ago but exercised recently is a live integration,
    // not abandoned surface area.
    if (ageDays > 180 && days < 30) score += 4;
  }

  if (stale === 0) score += 15;
  if (fresh === approvals.length) score += 8;

  return { score: clamp(score), reasons };
}

/**
 * C. HISTORICAL CONTRACT EXPOSURE
 * Baseline 70. Every distinct contract the address is known to have interacted
 * with that is CURRENTLY on the blocklist drives this down. Monotonic by nature:
 * it can only worsen as more contracts get flagged.
 */
export function scoreHistoricalExposure({ interactedContracts, flaggedContracts, blocklistAvailable }) {
  if (!interactedContracts || !blocklistAvailable) return { unavailable: true };

  let score = 70;
  const reasons = [];
  const hits = flaggedContracts || [];

  for (const c of hits) {
    score -= 22;
    reasons.push(`Address has interacted with blocklisted contract ${c}.`);
  }

  if (hits.length === 0) {
    score += 20;
    if (interactedContracts.length >= 5) score += 8;
  }

  return { score: clamp(score), reasons };
}

/**
 * D. PUBLIC EXPOSURE (OSINT)
 * Currently screens the wallet against the OFAC SDN sanctioned Ethereum
 * address list (fetched whole and matched locally — nothing leaves the device).
 * Sanctions presence is severe and binary, so a single hit drops the score to
 * Critical; a clean screen is neutral (baseline 80) and does not inflate the
 * overall score. Returns unavailable when the list can't be fetched.
 */
export function scorePublicExposure({ osintMatches }) {
  if (!osintMatches) return { unavailable: true };

  let score = 80;
  const reasons = [];
  for (const m of osintMatches) {
    score -= 60;
    reasons.push(`Address appears in public dataset: ${m.source}.`);
  }

  return { score: clamp(score), reasons };
}

/**
 * Weighted overall score. Weights of unavailable sub-scores are renormalized
 * across the available ones. If nothing is computable, the overall is null.
 */
export function computeScore(inputs) {
  const subs = {
    approvalExposure: scoreApprovalExposure(inputs),
    approvalStaleness: scoreApprovalStaleness(inputs),
    historicalExposure: scoreHistoricalExposure(inputs),
    publicExposure: scorePublicExposure(inputs),
  };

  let weighted = 0;
  let totalWeight = 0;
  for (const key of Object.keys(subs)) {
    const s = subs[key];
    if (s.unavailable) continue;
    weighted += s.score * WEIGHTS[key];
    totalWeight += WEIGHTS[key];
  }

  const overall = totalWeight > 0 ? clamp(weighted / totalWeight) : null;
  const findings = Object.values(subs).flatMap((s) => s.reasons || []);

  return { overall, category: categoryFor(overall), subs, findings };
}

/**
 * REAL counterfactual: rerun the full scoring function with one approval
 * removed and diff the overall score. Never estimated, never hardcoded.
 */
export function counterfactualGain(inputs, approvalKey) {
  const base = computeScore(inputs);
  if (base.overall === null) return null;
  const without = computeScore({
    ...inputs,
    approvals: inputs.approvals.filter((a) => a.key !== approvalKey),
  });
  if (without.overall === null) return null;
  return without.overall - base.overall;
}
```

## src/lib/chain.js

```js
/**
 * Live on-chain + public data collection. 100% client-side, stateless.
 *
 * APPROVAL DISCOVERY: Etherscan's "Get Event Logs by Topics" API. Unlike
 * keyless public RPCs (which cap eth_getLogs to a few dozen blocks), Etherscan
 * returns the wallet's FULL lifetime Approval / ApprovalForAll history from
 * block 0, which is what a real exposure score requires.
 *
 * KEY RESOLUTION: an optional developer key (ETHERSCAN_API_KEY in config.js)
 * ships with the app; a user-supplied key (stored in localStorage) takes
 * precedence. Either enables full-depth Etherscan history. With NO key at all,
 * or if the Etherscan request fails, we fall back to a keyless public-RPC
 * windowed scan (~55 days) and flag the result limitedDepth=true so the UI can
 * warn the user instead of silently presenting a partial scan as complete.
 *
 * BLOCKLIST: ScamSniffer's open scam-address database. On fetch failure →
 * unavailable, never guess.
 */

import { UNLIMITED_THRESHOLD } from '@/lib/scoring';
import { ETHERSCAN_API_KEY } from '@/lib/config';

const ETHERSCAN_BASE = 'https://api.etherscan.io/v2/api';
const RPC_URL = 'https://eth.llamarpc.com';
const BLOCKLIST_URL =
  'https://raw.githubusercontent.com/scamsniffer/scam-database/main/blacklist/address.json';

// Approval(address indexed owner, address indexed spender, uint256 value)
const TOPIC_APPROVAL = '0x8c5be1e5ebec7d5bd14f71427d1e84f3dd0314c0f7b2291e5b200ac8c7c3b925';
// ApprovalForAll(address indexed owner, address indexed operator, bool approved)
const TOPIC_APPROVAL_FOR_ALL = '0x17307eab39ab6107e8899845ad3d59bd9653f200f220920489ca2b5937696c31';

const MAX_PAGES = 10; // 10 × 1000 = 10k logs; plenty for approval history.
const RPC_WINDOWS = [400000, 120000, 40000]; // ~55 / 16 / 5 days of mainnet blocks.

const pad32 = (addr) => '0x' + '0'.repeat(24) + addr.toLowerCase().replace(/^0x/, '');
const addrFromTopic = (t) => (t && t.length >= 66 ? '0x' + t.slice(26).toLowerCase() : '');

export async function fetchBlocklist() {
  try {
    const res = await fetch(BLOCKLIST_URL);
    if (!res.ok) throw new Error('blocklist fetch failed');
    const list = await res.json();
    const arr = Array.isArray(list) ? list : Object.keys(list);
    return new Set(arr.map((a) => String(a).toLowerCase()));
  } catch {
    return null; // unavailable — callers must not fabricate
  }
}

async function etherscanLogs(apiKey, topic0, topic1) {
  const all = [];
  const params = new URLSearchParams({
    chainid: '1',
    module: 'logs',
    action: 'getLogs',
    fromBlock: '0',
    toBlock: 'latest',
    topic0,
    topic1,
    offset: '1000',
    apikey: apiKey,
  });
  for (let page = 1; page <= MAX_PAGES; page++) {
    params.set('page', String(page));
    const res = await fetch(`${ETHERSCAN_BASE}?${params.toString()}`);
    const json = await res.json();
    if (json.status === '0') {
      // "No records found" is a normal empty result, not an error.
      if (json.message === 'No records found') break;
      throw new Error(typeof json.result === 'string' ? json.result : json.message || 'Etherscan error');
    }
    const rows = Array.isArray(json.result) ? json.result : [];
    all.push(...rows);
    if (rows.length < 1000) break;
  }
  return all;
}

// --- Keyless public-RPC fallback (~55-day windowed scan) ---------------
async function rpc(method, params) {
  const res = await fetch(RPC_URL, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ jsonrpc: '2.0', id: 1, method, params }),
  });
  const json = await res.json();
  if (json.error) throw new Error(json.error.message || 'RPC error');
  return json.result;
}

async function getLogsWindow(topics, latest) {
  for (const w of RPC_WINDOWS) {
    const fromBlock = '0x' + Math.max(0, latest - w).toString(16);
    try {
      return await rpc('eth_getLogs', [{ fromBlock, toBlock: 'latest', topics }]);
    } catch {
      /* try a narrower window */
    }
  }
  throw new Error('log query failed');
}

async function blockTimestamps(blockHexList) {
  const unique = [...new Set(blockHexList)].slice(0, 30);
  const map = {};
  await Promise.all(
    unique.map(async (bn) => {
      try {
        const b = await rpc('eth_getBlockByNumber', [bn, false]);
        if (b) map[bn] = parseInt(b.timestamp, 16) * 1000;
      } catch {
        /* leave undefined */
      }
    })
  );
  return map;
}

// Normalize a log from either source into a common shape.
function normLog(log, ts) {
  return {
    address: log.address,
    topics: log.topics,
    data: log.data || '0x',
    blockHex: log.blockNumber,
    bn: parseInt(log.blockNumber, 16),
    ts: ts ?? null,
  };
}

/**
 * Reads live approval state for an address and cross-references the blocklist.
 * Returns { approvals, interactedContracts, flaggedContracts, blocklistAvailable, limitedDepth }.
 * limitedDepth is true when the scan fell back to the keyless RPC window
 * (~55 days) — the UI must surface this, never silently treat it as complete.
 */
export async function collectWalletData(address, apiKey) {
  const blocklist = await fetchBlocklist();
  const blocklistAvailable = blocklist !== null;
  const owner = pad32(address);
  const effectiveKey = apiKey || ETHERSCAN_API_KEY;

  let logs = null;
  let limitedDepth = false;

  if (effectiveKey) {
    try {
      const [a, b] = await Promise.all([
        etherscanLogs(effectiveKey, TOPIC_APPROVAL, owner),
        etherscanLogs(effectiveKey, TOPIC_APPROVAL_FOR_ALL, owner).catch(() => []),
      ]);
      logs = [...a, ...b].map((l) => normLog(l, parseInt(l.timeStamp, 16) * 1000));
    } catch {
      logs = null; // fall through to RPC fallback
    }
  }

  if (!logs) {
    limitedDepth = true;
    try {
      const latest = parseInt(await rpc('eth_blockNumber', []), 16);
      const [a, b] = await Promise.all([
        getLogsWindow([TOPIC_APPROVAL, owner], latest),
        getLogsWindow([TOPIC_APPROVAL_FOR_ALL, owner], latest).catch(() => []),
      ]);
      const raw = [...a, ...b];
      const stamps = await blockTimestamps(raw.map((l) => l.blockNumber));
      logs = raw.map((l) => normLog(l, stamps[l.blockNumber] ?? null));
    } catch {
      return {
        approvals: null,
        interactedContracts: null,
        flaggedContracts: null,
        blocklistAvailable,
        limitedDepth,
      };
    }
  }

  // Latest log per (token, spender) pair defines current approval state.
  const byPair = new Map();
  const firstSeen = new Map();
  for (const log of logs) {
    if (!log.topics || log.topics.length < 3) continue;
    const token = log.address.toLowerCase();
    const spender = addrFromTopic(log.topics[2]);
    if (!spender) continue;
    const key = `${token}:${spender}`;
    if (!firstSeen.has(key) || log.bn < firstSeen.get(key).bn) {
      firstSeen.set(key, { bn: log.bn, ts: log.ts });
    }
    const prev = byPair.get(key);
    if (!prev || log.bn > prev.bn) byPair.set(key, { log, token, spender, key, ts: log.ts });
  }

  const approvals = [];
  for (const entry of byPair.values()) {
    const isForAll = entry.log.topics[0] === TOPIC_APPROVAL_FOR_ALL;
    let value = 0n;
    let active;
    if (isForAll) {
      // ApprovalForAll: the `approved` bool is indexed as topic3, data is empty.
      const approvedTopic = entry.log.topics[3];
      active = !!approvedTopic && approvedTopic.toLowerCase().endsWith('1');
      value = active ? 1n : 0n;
    } else {
      try {
        value = BigInt(entry.log.data && entry.log.data !== '0x' ? entry.log.data.slice(0, 66) : '0x0');
      } catch {
        value = 0n;
      }
      active = value > 0n;
    }
    if (!active) continue;

    approvals.push({
      key: entry.key,
      token: entry.token,
      spender: entry.spender,
      unlimited: isForAll || value >= UNLIMITED_THRESHOLD,
      flagged: blocklistAvailable && blocklist.has(entry.spender),
      lastActivityAt: entry.ts ?? null,
      grantedAt: firstSeen.get(entry.key)?.ts ?? null,
    });
  }

  const interactedContracts = [
    ...new Set(logs.flatMap((l) => [l.address.toLowerCase(), addrFromTopic(l.topics[2] || '')])),
  ].filter(Boolean);

  const flaggedContracts = blocklistAvailable
    ? interactedContracts.filter((c) => blocklist.has(c))
    : null;

  return { approvals, interactedContracts, flaggedContracts, blocklistAvailable, limitedDepth };
}

/**
 * PUBLIC EXPOSURE (OSINT) — OFAC SDN sanctions screening.
 *
 * Source: the auto-updated Ethereum address list extracted from the U.S.
 * Treasury OFAC SDN list, published by 0xB10C/ofac-sanctioned-digital-currency-
 * addresses (MIT), regenerated nightly from the official sdn_advanced.xml.
 *
 * Privacy: the entire list (~100 addresses) is downloaded and matched locally
 * in the browser. The wallet address never leaves the device for this check —
 * stronger than the k-anonymity range design, and fully stateless.
 *
 * Returns an array of { source } matches (empty = clean), or null when the list
 * is unreachable so the scorer marks the sub-score Unavailable rather than guess.
 */
const OFAC_ETH_URL =
  'https://raw.githubusercontent.com/0xB10C/ofac-sanctioned-digital-currency-addresses/lists/sanctioned_addresses_ETH.txt';

export async function lookupPublicExposure(address) {
  try {
    const res = await fetch(OFAC_ETH_URL);
    if (!res.ok) return null;
    const text = await res.text();
    const set = new Set(
      text
        .split('\n')
        .map((l) => l.trim().toLowerCase())
        .filter(Boolean)
    );
    const matches = [];
    if (set.has(address.toLowerCase())) {
      matches.push({ source: 'OFAC SDN sanctions list' });
    }
    return matches;
  } catch {
    return null; // unavailable — callers must not fabricate
  }
}
```

## src/lib/storage.js

```js
// All WalletXS persistence lives in localStorage. No backend, no accounts.
// The active address + API key are also mirrored to IndexedDB so the service
// worker can read them during a Periodic Background Sync (SWs can't access
// localStorage).

import { syncSetAddress, syncSetApiKey, syncClearAddress, syncClearApiKey } from '@/lib/syncdb';

const ADDRESS_KEY = 'walletxs.address'; // legacy single-address key (migrated away)
const TRACKED_KEY = 'walletxs.trackedAddresses'; // array of { address, label, addedAt }
const ACTIVE_KEY = 'walletxs.activeAddress'; // which tracked wallet is currently viewed
const HISTORY_KEY = 'walletxs.history';
const APIKEY_KEY = 'walletxs.etherscanKey';

const PER_ADDRESS_HISTORY = 50;

function readJSON(key, fallback) {
  try {
    const raw = localStorage.getItem(key);
    return raw ? JSON.parse(raw) : fallback;
  } catch {
    return fallback;
  }
}

function writeJSON(key, value) {
  try {
    localStorage.setItem(key, JSON.stringify(value));
  } catch {
    /* storage unavailable */
  }
}

// --- One-time migration from the legacy single-address key -----------------
// An existing user with the old 'walletxs.address' key and no tracked list
// gets folded into the new multi-wallet model transparently: their address
// becomes the first tracked wallet and the active one, then the old key is
// dropped. History is keyed per-address already, so it carries over untouched.
// Idempotent: once a tracked list exists (even an empty []), this never re-runs.
let migrated = false;
function migrateLegacyAddress() {
  if (migrated) return;
  migrated = true;
  let legacy = null;
  try {
    legacy = localStorage.getItem(ADDRESS_KEY);
  } catch {
    legacy = null;
  }
  if (!legacy) return;
  const tracked = readJSON(TRACKED_KEY, null);
  if (tracked === null) {
    // No tracked list yet — seed it with the legacy address.
    writeJSON(TRACKED_KEY, [{ address: legacy.toLowerCase(), label: null, addedAt: Date.now() }]);
    try {
      localStorage.setItem(ACTIVE_KEY, legacy.toLowerCase());
    } catch {
      /* noop */
    }
    syncSetAddress(legacy.toLowerCase());
  }
  // The legacy key is obsolete in either branch.
  try {
    localStorage.removeItem(ADDRESS_KEY);
  } catch {
    /* noop */
  }
}

// --- Tracked wallet list ---------------------------------------------------
export function loadTrackedAddresses() {
  migrateLegacyAddress();
  return readJSON(TRACKED_KEY, []);
}

export function addTrackedAddress(address, label = null) {
  const lower = address.toLowerCase();
  const list = loadTrackedAddresses();
  if (list.some((w) => w.address === lower)) return; // already tracked — no-op
  list.push({ address: lower, label, addedAt: Date.now() });
  writeJSON(TRACKED_KEY, list);
}

export function removeTrackedAddress(address) {
  const lower = address.toLowerCase();
  const list = loadTrackedAddresses().filter((w) => w.address !== lower);
  writeJSON(TRACKED_KEY, list);
  // NOTE: history is intentionally NOT deleted here — re-adding the same
  // address later restores its prior score history.
}

// --- Active (currently-viewed) wallet --------------------------------------
export function loadActiveAddress() {
  migrateLegacyAddress();
  try {
    return localStorage.getItem(ACTIVE_KEY);
  } catch {
    return null;
  }
}

export function saveActiveAddress(address) {
  const lower = address.toLowerCase();
  try {
    localStorage.setItem(ACTIVE_KEY, lower);
  } catch {
    /* noop */
  }
  // Mirror to IndexedDB so the service worker can read it for background checks.
  syncSetAddress(lower);
}

export function clearActiveAddress() {
  try {
    localStorage.removeItem(ACTIVE_KEY);
  } catch {
    /* noop */
  }
  syncClearAddress();
}

// --- API key ---------------------------------------------------------------
export function loadApiKey() {
  try {
    return localStorage.getItem(APIKEY_KEY);
  } catch {
    return null;
  }
}

export function saveApiKey(key) {
  try {
    localStorage.setItem(APIKEY_KEY, key);
  } catch {
    /* storage unavailable */
  }
  syncSetApiKey(key);
}

export function clearApiKey() {
  try {
    localStorage.removeItem(APIKEY_KEY);
  } catch {
    /* noop */
  }
  syncClearApiKey();
}

// --- Per-address score history --------------------------------------------
function readHistory() {
  return readJSON(HISTORY_KEY, []);
}

/** The most recent stored check for this address, before the current one. */
export function lastCheckFor(address) {
  const entries = readHistory().filter((e) => e.address === address.toLowerCase());
  return entries.length ? entries[entries.length - 1] : null;
}

/** The most recent `limit` checks for this address, oldest first (for sparklines). */
export function historyFor(address, limit = 10) {
  const entries = readHistory().filter((e) => e.address === address.toLowerCase());
  const window = entries.slice(-limit);
  return window.map((e) => ({ score: e.score, findings: e.findings, at: e.at }));
}

export function recordCheck(address, score, findings) {
  if (score === null || score === undefined) return;
  const lower = address.toLowerCase();
  const history = readHistory();
  history.push({ address: lower, score, findings: findings || [], at: Date.now() });
  // Cap PER ADDRESS so a frequently-checked wallet can't evict another
  // wallet's history from this shared array. Group by address, keep the
  // newest PER_ADDRESS_HISTORY entries for each, then flatten back to one array.
  const byAddr = new Map();
  for (const e of history) {
    if (!byAddr.has(e.address)) byAddr.set(e.address, []);
    byAddr.get(e.address).push(e);
  }
  const trimmed = [];
  for (const entries of byAddr.values()) {
    trimmed.push(...entries.slice(-PER_ADDRESS_HISTORY));
  }
  writeJSON(HISTORY_KEY, trimmed);
}
```

## src/lib/syncdb.js

```js
// IndexedDB mirror of the wallet address + Etherscan key, so the service
// worker can read them during a Periodic Background Sync (SWs cannot access
// localStorage). 100% client-side, no backend. The 'lastSig' entry is written
// by the SW itself (the approval-surface signature from the last background
// check); the page only mirrors address + apiKey.

const DB = 'walletxs-sync';
const STORE = 'kv';
const VERSION = 1;

function openDB() {
  return new Promise((resolve, reject) => {
    const req = indexedDB.open(DB, VERSION);
    req.onupgradeneeded = () => req.result.createObjectStore(STORE, { keyPath: 'k' });
    req.onsuccess = () => resolve(req.result);
    req.onerror = () => reject(req.error);
  });
}

async function setKV(k, v) {
  try {
    const db = await openDB();
    await new Promise((resolve) => {
      const tx = db.transaction(STORE, 'readwrite');
      tx.objectStore(STORE).put({ k, v });
      tx.oncomplete = () => { db.close(); resolve(); };
      tx.onerror = () => resolve();
    });
  } catch {
    /* IDB unavailable */
  }
}

async function delKV(k) {
  try {
    const db = await openDB();
    await new Promise((resolve) => {
      const tx = db.transaction(STORE, 'readwrite');
      tx.objectStore(STORE).delete(k);
      tx.oncomplete = () => { db.close(); resolve(); };
      tx.onerror = () => resolve();
    });
  } catch {
    /* IDB unavailable */
  }
}

export const syncSetAddress = (addr) => setKV('address', addr);
export const syncSetApiKey = (key) => setKV('apiKey', key);
export const syncClearAddress = () => Promise.all([delKV('address'), delKV('lastSig')]);
export const syncClearApiKey = () => delKV('apiKey');
```

## src/lib/backup.js

```js
// Client-side backup of the tracked-wallet list. No backend, no library —
// just the native Blob/File APIs. Export downloads the current list as JSON;
// import reads a JSON file back in, validates each entry is a 0x address,
// and adds the valid ones via addTrackedAddress (which no-ops on duplicates).

import { loadTrackedAddresses, addTrackedAddress } from '@/lib/storage';

const isAddress = (v) => typeof v === 'string' && /^0x[a-fA-F0-9]{40}$/.test(v);

export function exportBackup() {
  const list = loadTrackedAddresses();
  const blob = new Blob([JSON.stringify(list, null, 2)], { type: 'application/json' });
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = 'walletxs-backup.json';
  document.body.appendChild(a);
  a.click();
  document.body.removeChild(a);
  URL.revokeObjectURL(url);
}

// Reads a JSON file and adds every valid wallet entry. Returns
// { added, duplicate, invalid } on success, or throws with a plain-language
// message the caller can show directly (never silently fails).
export async function importBackup(file) {
  const text = await file.text();
  let data;
  try {
    data = JSON.parse(text);
  } catch {
    throw new Error("Couldn't parse the file as JSON.");
  }
  if (!Array.isArray(data)) {
    throw new Error('Backup file must be a list of wallets.');
  }

  const existing = new Set(loadTrackedAddresses().map((w) => w.address));
  let added = 0;
  let duplicate = 0;
  let invalid = 0;
  for (const item of data) {
    if (!item || typeof item !== 'object' || !isAddress(item.address)) {
      invalid += 1;
      continue;
    }
    const addr = item.address.toLowerCase();
    if (existing.has(addr)) {
      duplicate += 1;
      continue;
    }
    addTrackedAddress(addr);
    existing.add(addr);
    added += 1;
  }

  if (added === 0) {
    const reasons = [];
    if (duplicate) reasons.push(`${duplicate} already tracked`);
    if (invalid) reasons.push(`${invalid} invalid`);
    throw new Error(`No new wallets imported (${reasons.join(', ') || 'empty list'}).`);
  }
  return { added, duplicate, invalid };
}
```

## src/lib/percentile.js

```js
import sample from '@/lib/percentileSample.json';

/**
 * Percentile against a STATIC bundled sample shipped with the app.
 * Never live aggregation of other users' scores.
 */
export function percentileFor(score) {
  if (score === null || score === undefined) return null;
  let below = 0;
  for (const b of sample.buckets) {
    if (b.max < score) below += b.count;
  }
  return Math.round((below / sample.sampleSize) * 100);
}

export const SAMPLE_SIZE = sample.sampleSize;
export const SAMPLE_UPDATED = sample.updated;
```

## src/lib/percentileSample.json

```json
{
  "sampleSize": 4812,
  "updated": "2026-06-01",
  "source": "Static bundled sample of public Ethereum wallets scored offline. Not live user data.",
  "buckets": [
    { "min": 0, "max": 9, "count": 96 },
    { "min": 10, "max": 19, "count": 168 },
    { "min": 20, "max": 29, "count": 289 },
    { "min": 30, "max": 39, "count": 433 },
    { "min": 40, "max": 49, "count": 626 },
    { "min": 50, "max": 59, "count": 770 },
    { "min": 60, "max": 69, "count": 818 },
    { "min": 70, "max": 79, "count": 703 },
    { "min": 80, "max": 89, "count": 529 },
    { "min": 90, "max": 100, "count": 380 }
  ]
}
```

## src/lib/severity.js

```js
// Single source of truth for the four-tier score language and colors.

export function tier(score) {
  if (score === null || score === undefined) {
    return {
      label: 'Unavailable',
      text: 'text-neutral-400',
      border: 'border-neutral-200',
      dot: 'bg-neutral-300',
      ring: '#d4d4d4',
      badge: 'text-neutral-500 border-neutral-200 bg-neutral-50',
    };
  }
  if (score >= 80)
    return {
      label: 'Strong',
      text: 'text-emerald-600',
      border: 'border-emerald-200',
      dot: 'bg-emerald-500',
      ring: '#10b981',
      badge: 'text-emerald-700 border-emerald-200 bg-emerald-50',
    };
  if (score >= 60)
    return {
      label: 'Fair',
      text: 'text-amber-600',
      border: 'border-amber-200',
      dot: 'bg-amber-500',
      ring: '#f59e0b',
      badge: 'text-amber-700 border-amber-200 bg-amber-50',
    };
  if (score >= 40)
    return {
      label: 'At Risk',
      text: 'text-orange-600',
      border: 'border-orange-200',
      dot: 'bg-orange-500',
      ring: '#f97316',
      badge: 'text-orange-700 border-orange-200 bg-orange-50',
    };
  return {
    label: 'Critical',
    text: 'text-red-600',
    border: 'border-red-200',
    dot: 'bg-red-500',
    ring: '#ef4444',
    badge: 'text-red-700 border-red-200 bg-red-50',
  };
}

export const shortAddr = (a) => (a ? `${a.slice(0, 6)}…${a.slice(-4)}` : '');
```

## src/lib/notify.js

```js
// 100% stateless browser notifications. No backend, no push tokens, no server
// storage — the Notification API fires entirely client-side, in the user's
// browser, only while the WalletXS tab is open.
//
// Permission is requested ONLY on an explicit user gesture (the "Enable
// alerts" button), per browser policy — we never auto-prompt on page load.
// Notifications fire only when the tab is hidden: if the user is already
// looking at the score, a notification would be noise.

export function notifySupported() {
  return typeof window !== 'undefined' && 'Notification' in window;
}

export function notifyPermission() {
  return notifySupported() ? Notification.permission : 'denied';
}

export async function requestNotifyPermission() {
  if (!notifySupported()) return 'denied';
  try {
    return await Notification.requestPermission();
  } catch {
    return 'denied';
  }
}

// Fire a score-change notification. Stateless: nothing is sent anywhere.
// Returns true only if a notification was actually shown.
export function notifyScoreChange(addr, prevScore, score) {
  if (!notifySupported() || Notification.permission !== 'granted') return false;
  if (!document.hidden) return false; // user is already looking at the page

  const diff = score - prevScore;
  const dir = diff > 0 ? 'up' : 'down';
  const short = `${addr.slice(0, 6)}…${addr.slice(-4)}`;
  try {
    new Notification('WalletXS score changed', {
      body: `${short}: ${prevScore} → ${score} (${dir} ${Math.abs(diff)} pt${
        Math.abs(diff) === 1 ? '' : 's'
      })`,
      tag: `walletxs-${addr}`, // replaces a stale notification for the same wallet
    });
    return true;
  } catch {
    return false;
  }
}
```

---

_Continued in **WalletXS-Source-Part2.md** (components) and **WalletXS-Source-Part3.md** (pages + dependencies)._
