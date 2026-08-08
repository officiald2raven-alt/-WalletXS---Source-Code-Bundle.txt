# WalletXS — GitHub Source Bundle (Part 2 of 3)

Components. See **WalletXS-Source-Part1.md** for config / build files / public assets / hooks / lib, and **WalletXS-Source-Part3.md** for pages + dependencies.

---

## src/components/walletxs/Header.jsx

```jsx
import React from 'react';
import { Shield, KeyRound, Github, Wallet } from 'lucide-react';
import { Button } from '@/components/ui/button';
import InstallButton from '@/components/walletxs/InstallButton';
import ScoreAlertsToggle from '@/components/walletxs/ScoreAlertsToggle';
import { GITHUB_REPO_URL } from '@/lib/config';

export default function Header({ address, apiKey, onKeyChange, onShowWallets }) {
  return (
    <header className="flex flex-wrap items-center justify-between gap-3 py-6">
      <div className="flex items-center gap-2">
        <Shield className="w-5 h-5 text-neutral-900" strokeWidth={1.75} />
        <span className="text-lg font-semibold tracking-tight text-neutral-900">WalletXS</span>
      </div>
      <div className="flex items-center gap-2 sm:gap-3">
        {address && (
          <Button
            variant="outline"
            size="sm"
            onClick={onShowWallets}
            title="View and manage your tracked wallets"
            className="h-8 text-xs border-neutral-200 bg-white hover:bg-neutral-50"
          >
            <Wallet className="w-3.5 h-3.5" strokeWidth={1.75} />
            <span className="hidden sm:inline">My wallets</span>
          </Button>
        )}
        <a
          href={GITHUB_REPO_URL}
          target="_blank"
          rel="noopener noreferrer"
          title="View source code on GitHub"
          className="inline-flex items-center gap-2 rounded-lg border border-neutral-200 bg-white px-3 py-2 text-sm font-medium text-neutral-700 hover:bg-neutral-50 transition-colors"
        >
          <Github className="w-4 h-4" strokeWidth={1.75} />
          <span className="hidden sm:inline">Source</span>
        </a>
        <InstallButton />
        {apiKey && (
          <Button
            variant="outline"
            size="sm"
            onClick={onKeyChange}
            title="Change Etherscan API key"
            className="h-8 text-xs border-neutral-200 bg-white hover:bg-neutral-50"
          >
            <KeyRound className="w-3.5 h-3.5" strokeWidth={1.75} />
            API key
          </Button>
        )}
        {address && <ScoreAlertsToggle />}
      </div>
    </header>
  );
}
```

## src/components/walletxs/AddressEntry.jsx

```jsx
import React, { useState } from 'react';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Wallet } from 'lucide-react';
import { shortAddr } from '@/lib/severity';

const isAddress = (v) => /^0x[a-fA-F0-9]{40}$/.test(v.trim());

export default function AddressEntry({ onSubmit }) {
  const [value, setValue] = useState('');
  const [error, setError] = useState('');
  // Accounts returned by the wallet when more than one is available; when set,
  // we show a picker instead of the connect/paste form.
  const [accounts, setAccounts] = useState(null);

  const connect = async () => {
    setError('');
    if (!window.ethereum) {
      setError('No browser wallet detected. Paste an address instead.');
      return;
    }
    try {
      // Read-only: accounts request only. Never a signature or transaction.
      const accs = await window.ethereum.request({ method: 'eth_requestAccounts' });
      if (!accs || !accs.length) {
        setError('No accounts found in your wallet.');
        return;
      }
      const lower = accs.map((a) => a.toLowerCase());
      if (lower.length === 1) {
        onSubmit(lower[0]);
      } else {
        // Multiple wallets — let the user pick which one to check.
        setAccounts(lower);
      }
    } catch {
      setError('Wallet connection was cancelled.');
    }
  };

  const pick = (a) => {
    setAccounts(null);
    onSubmit(a);
  };

  const submit = (e) => {
    e.preventDefault();
    if (!isAddress(value)) {
      setError('Enter a valid 0x address.');
      return;
    }
    onSubmit(value.trim().toLowerCase());
  };

  return (
    <div className="rounded-xl border border-[#e5e5e7] bg-[#f5f5f6] p-8">
      <h1 className="text-xl font-semibold tracking-tight text-neutral-900">
        Check a wallet's exposure
      </h1>
      <p className="mt-1.5 text-sm text-neutral-500">
        Scored live from public on-chain data each time you check. Nothing is stored on a server.
      </p>

      {accounts && accounts.length > 1 ? (
        <div className="mt-6">
          <p className="text-sm font-medium text-neutral-700">
            Select a wallet to check ({accounts.length} found)
          </p>
          <div className="mt-3 space-y-2">
            {accounts.map((a, i) => (
              <button
                key={a}
                onClick={() => pick(a)}
                className="flex w-full items-center justify-between rounded-lg border border-neutral-200 bg-white px-4 py-3 text-left hover:bg-neutral-50 transition-colors"
              >
                <span className="font-mono text-sm text-neutral-800">{shortAddr(a)}</span>
                <span className="text-xs text-neutral-400">Account {i + 1}</span>
              </button>
            ))}
          </div>
          <button
            onClick={() => setAccounts(null)}
            className="mt-3 text-xs text-neutral-500 hover:text-neutral-800 transition-colors"
          >
            ← Back
          </button>
        </div>
      ) : (
        <>
          <button
            onClick={connect}
            className="mt-6 inline-flex w-full items-center justify-center gap-2 rounded-lg bg-neutral-900 px-4 py-3 text-sm font-medium text-white hover:bg-neutral-800 transition-colors"
          >
            <Wallet className="w-4 h-4" strokeWidth={1.75} />
            Connect wallet (read-only)
          </button>

          <div className="mt-5 flex items-center gap-3">
            <div className="flex-1 h-px bg-neutral-200" />
            <span className="text-xs text-neutral-400">or paste an address</span>
            <div className="flex-1 h-px bg-neutral-200" />
          </div>

          <form onSubmit={submit} className="mt-4 flex flex-col sm:flex-row gap-2">
            <Input
              value={value}
              onChange={(e) => setValue(e.target.value)}
              placeholder="0x…"
              className="font-mono text-sm bg-white border-neutral-200 h-11"
            />
            <Button type="submit" className="h-11 bg-neutral-900 hover:bg-neutral-800">
              Check
            </Button>
          </form>
        </>
      )}

      {error && <p className="mt-3 text-sm text-red-600">{error}</p>}
    </div>
  );
}
```

## src/components/walletxs/ApiKeyEntry.jsx

```jsx
import React, { useState } from 'react';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { KeyRound, ExternalLink } from 'lucide-react';

export default function ApiKeyEntry({ onSaved }) {
  const [value, setValue] = useState('');
  const [error, setError] = useState('');

  const submit = (e) => {
    e.preventDefault();
    const v = value.trim();
    if (!/^[A-Za-z0-9]{20,}$/.test(v)) {
      setError('Enter a valid Etherscan API key.');
      return;
    }
    onSaved(v);
  };

  return (
    <div className="rounded-xl border border-[#e5e5e7] bg-[#f5f5f6] p-8">
      <div className="flex items-center gap-2">
        <KeyRound className="w-5 h-5 text-neutral-900" strokeWidth={1.75} />
        <h1 className="text-xl font-semibold tracking-tight text-neutral-900">
          Connect a free Etherscan key
        </h1>
      </div>
      <p className="mt-1.5 text-sm text-neutral-500">
        WalletXS reads your wallet's full lifetime approval history from Etherscan.
        A free key is required — it's stored only in your browser and sent only to Etherscan.
      </p>
      <form onSubmit={submit} className="mt-6 flex flex-col sm:flex-row gap-2">
        <Input
          value={value}
          onChange={(e) => setValue(e.target.value)}
          placeholder="Your Etherscan API key"
          className="font-mono text-sm bg-white border-neutral-200 h-11"
        />
        <Button type="submit" className="h-11 bg-neutral-900 hover:bg-neutral-800">
          Save key
        </Button>
      </form>
      <a
        href="https://etherscan.io/myapikey"
        target="_blank"
        rel="noreferrer"
        className="mt-4 inline-flex items-center gap-1.5 text-sm text-neutral-600 hover:text-neutral-900 transition-colors"
      >
        Get a free key at etherscan.io/myapikey
        <ExternalLink className="w-3.5 h-3.5" strokeWidth={1.75} />
      </a>
      {error && <p className="mt-3 text-sm text-red-600">{error}</p>}
    </div>
  );
}
```

## src/components/walletxs/ScoreCard.jsx

```jsx
import React from 'react';
import { motion } from 'framer-motion';
import { tier } from '@/lib/severity';
import ScoreGauge from '@/components/walletxs/ScoreGauge';
import DeltaPill from '@/components/walletxs/DeltaPill';

export default function ScoreCard({ score, lastCheck }) {
  const t = tier(score);
  return (
    <motion.div
      initial={{ opacity: 0, y: 8 }}
      animate={{ opacity: 1, y: 0 }}
      transition={{ duration: 0.35, ease: 'easeOut' }}
      className="rounded-2xl border border-[#e5e5e7] bg-gradient-to-b from-white to-[#f5f5f6] px-6 py-12 flex flex-col items-center"
    >
      <ScoreGauge score={score} size={240} stroke={18} tierObj={t} fluid>
        {score === null ? (
          <>
            <div className="text-6xl font-light tracking-tight text-neutral-300">—</div>
            <div className="mt-2 text-[11px] uppercase tracking-[0.18em] text-neutral-400">
              Unavailable
            </div>
          </>
        ) : (
          <>
            <div className={`text-6xl font-light tracking-tighter tabular-nums ${t.text}`}>
              {score}
            </div>
            <div
              className={`mt-1 text-xs font-medium uppercase tracking-[0.18em] ${t.text}`}
            >
              {t.label}
            </div>
          </>
        )}
      </ScoreGauge>
      {score === null ? (
        <div className="mt-6 text-sm text-neutral-500 text-center max-w-xs">
          Score unavailable — no sub-score could be computed from real data.
        </div>
      ) : (
        <DeltaPill score={score} lastCheck={lastCheck} />
      )}
    </motion.div>
  );
}
```

## src/components/walletxs/ScoreGauge.jsx

```jsx
import React from 'react';
import { motion } from 'framer-motion';

// Animated circular score gauge.
// `score` is 0-100 or null (unavailable). `tierObj.ring` sets the arc color.
// Children render centered inside the ring.
export default function ScoreGauge({
  score,
  size = 220,
  stroke = 16,
  tierObj,
  children,
  fluid = false,
}) {
  const radius = (size - stroke) / 2;
  const circ = 2 * Math.PI * radius;
  const pct = score == null ? 0 : Math.max(0, Math.min(100, score)) / 100;
  const color = tierObj?.ring || '#d4d4d4';
  const track = '#e5e5e7';

  return (
    <div
      className={`relative inline-flex items-center justify-center${
        fluid ? ' aspect-square' : ''
      }`}
      style={fluid ? { maxWidth: size, width: '100%' } : { width: size, height: size }}
    >
      <svg
        width={size}
        height={size}
        viewBox={`0 0 ${size} ${size}`}
        className={`-rotate-90${fluid ? ' w-full h-auto' : ''}`}
        style={{ filter: `drop-shadow(0 0 10px ${color}33)` }}
      >
        <circle
          cx={size / 2}
          cy={size / 2}
          r={radius}
          fill="none"
          stroke={track}
          strokeWidth={stroke}
        />
        <motion.circle
          cx={size / 2}
          cy={size / 2}
          r={radius}
          fill="none"
          stroke={color}
          strokeWidth={stroke}
          strokeLinecap="round"
          strokeDasharray={circ}
          initial={{ strokeDashoffset: circ }}
          animate={{ strokeDashoffset: circ - circ * pct }}
          transition={{ duration: 1.1, ease: 'easeOut' }}
        />
      </svg>
      <div className="absolute inset-0 flex flex-col items-center justify-center">
        {children}
      </div>
    </div>
  );
}
```

## src/components/walletxs/DeltaPill.jsx

```jsx
import React from 'react';
import { ArrowDown, ArrowUp, Minus } from 'lucide-react';

export default function DeltaPill({ score, lastCheck }) {
  // First-ever check: render nothing rather than a fake "no change" state.
  if (!lastCheck) return null;

  const diff = score - lastCheck.score;
  const days = Math.max(0, Math.round((Date.now() - lastCheck.at) / 86400000));
  const when = days === 0 ? 'today' : days === 1 ? '1 day ago' : `${days} days ago`;

  const Icon = diff > 0 ? ArrowUp : diff < 0 ? ArrowDown : Minus;
  const cls =
    diff > 0
      ? 'text-emerald-700 border-emerald-200 bg-emerald-50'
      : diff < 0
      ? 'text-red-700 border-red-200 bg-red-50'
      : 'text-neutral-600 border-neutral-200 bg-white';
  const text =
    diff === 0
      ? `unchanged since your last check, ${when}`
      : `${diff > 0 ? 'up' : 'down'} ${Math.abs(diff)} point${
          Math.abs(diff) === 1 ? '' : 's'
        } since your last check, ${when}`;

  return (
    <div className="mt-6 flex justify-center">
      <span
        className={`inline-flex items-center gap-1.5 rounded-full border px-3 py-1 text-xs ${cls}`}
      >
        <Icon className="w-3 h-3" strokeWidth={2} />
        {text}
      </span>
    </div>
  );
}
```

## src/components/walletxs/SubScoreCard.jsx

```jsx
import React from 'react';
import { tier } from '@/lib/severity';
import ScoreGauge from '@/components/walletxs/ScoreGauge';

export default function SubScoreCard({ label, result }) {
  const unavailable = !result || result.unavailable;
  const t = tier(unavailable ? null : result.score);
  const score = unavailable ? null : result.score;

  return (
    <div className={`rounded-xl border bg-[#f5f5f6] p-5 flex items-center gap-4 ${t.border}`}>
      <ScoreGauge score={score} size={88} stroke={9} tierObj={t}>
        {unavailable ? (
          <span className="text-lg font-light text-neutral-300">—</span>
        ) : (
          <span className={`text-xl font-light tabular-nums ${t.text}`}>{score}</span>
        )}
      </ScoreGauge>
      <div className="min-w-0">
        <div className="text-[11px] uppercase tracking-[0.14em] text-neutral-500">{label}</div>
        <div className={`mt-1 text-sm font-medium ${unavailable ? 'text-neutral-400' : t.text}`}>
          {unavailable ? 'Unavailable' : t.label}
        </div>
      </div>
    </div>
  );
}
```

## src/components/walletxs/SubScoreGrid.jsx

```jsx
import React from 'react';
import SubScoreCard from '@/components/walletxs/SubScoreCard';
import { SUB_SCORE_LABELS } from '@/lib/scoring';

export default function SubScoreGrid({ subs }) {
  return (
    <div className="mt-4 grid grid-cols-1 sm:grid-cols-2 gap-4">
      {Object.keys(SUB_SCORE_LABELS).map((key) => (
        <SubScoreCard key={key} label={SUB_SCORE_LABELS[key]} result={subs[key]} />
      ))}
    </div>
  );
}
```

## src/components/walletxs/WhatChanged.jsx

```jsx
import React from 'react';
import { ExternalLink } from 'lucide-react';
import { Button } from '@/components/ui/button';

export default function WhatChanged({ findings }) {
  if (!findings.length) return null;

  return (
    <section className="mt-10">
      <h2 className="text-sm font-semibold uppercase tracking-[0.14em] text-neutral-500">
        What changed
      </h2>
      <div className="mt-3 space-y-3">
        {findings.map((f) => (
          <div key={f.id} className="rounded-xl border border-[#e5e5e7] bg-[#f5f5f6] p-5">
            <p className="text-sm leading-relaxed text-neutral-800">{f.text}</p>
            {f.spender && (
              <p className="mt-2 font-mono text-xs text-neutral-500 break-all">{f.spender}</p>
            )}
            {f.gain !== null && f.gain !== undefined && f.gain > 0 && (
              <p className="mt-2 text-xs text-emerald-700">
                +{f.gain} points if you revoke this approval
              </p>
            )}
            {f.spender && (
              <Button
                asChild
                variant="outline"
                size="sm"
                className="mt-4 h-8 text-xs border-neutral-200 bg-white hover:bg-neutral-50"
              >
                <a
                  href={`https://revoke.cash/address/${f.owner}?chainId=1`}
                  target="_blank"
                  rel="noreferrer"
                >
                  Revoke this approval
                  <ExternalLink className="ml-1.5 w-3 h-3" strokeWidth={1.75} />
                </a>
              </Button>
            )}
          </div>
        ))}
      </div>
    </section>
  );
}
```

## src/components/walletxs/PercentileLine.jsx

```jsx
import React from 'react';
import { Link } from 'react-router-dom';
import { ChevronRight } from 'lucide-react';
import { percentileFor, SAMPLE_SIZE, SAMPLE_UPDATED } from '@/lib/percentile';

export default function PercentileLine({ score }) {
  const pct = percentileFor(score);
  if (pct === null) return null;

  return (
    <div className="mt-12 border-t border-[#e5e5e7] pt-6">
      <Link
        to="/methodology"
        className="inline-flex items-center text-sm text-neutral-500 hover:text-neutral-800 transition-colors"
      >
        Safer than {pct}% of scanned wallets
        <ChevronRight className="ml-0.5 w-4 h-4" strokeWidth={1.75} />
      </Link>
      <p className="mt-1 text-xs text-neutral-400">
        Based on a sample of {SAMPLE_SIZE.toLocaleString()} public wallets, refreshed monthly
        (last: {new Date(SAMPLE_UPDATED).toLocaleDateString(undefined, { month: 'short', year: 'numeric' })})
      </p>
    </div>
  );
}
```

## src/components/walletxs/InstallButton.jsx

```jsx
import React from 'react';
import { Download, Check } from 'lucide-react';
import { useInstallPrompt } from '@/hooks/useInstallPrompt';

export default function InstallButton() {
  const { canInstall, prompt, installed } = useInstallPrompt();

  if (installed) {
    return (
      <button
        disabled
        className="inline-flex items-center gap-2 rounded-lg border border-emerald-200 bg-emerald-50 px-3 py-2 text-sm font-medium text-emerald-700 cursor-default"
      >
        <Check className="w-4 h-4" strokeWidth={2} />
        <span className="hidden sm:inline">Installed</span>
      </button>
    );
  }

  if (!canInstall) {
    return (
      <button
        disabled
        title="Install isn't available in this browser. Use your browser's Add to Home Screen / Install option."
        className="inline-flex items-center gap-2 rounded-lg border border-neutral-200 bg-neutral-100 px-3 py-2 text-sm font-medium text-neutral-400 cursor-not-allowed"
      >
        <Download className="w-4 h-4" strokeWidth={2} />
        <span className="hidden sm:inline">Install</span>
      </button>
    );
  }

  return (
    <button
      onClick={prompt}
      className="inline-flex items-center gap-2 rounded-lg border border-neutral-900 bg-neutral-900 px-3 py-2 text-sm font-medium text-white hover:bg-neutral-800 transition-colors"
    >
      <Download className="w-4 h-4" strokeWidth={2} />
      <span className="hidden sm:inline">Install</span>
    </button>
  );
}
```

## src/components/walletxs/ScoreAlertsToggle.jsx

```jsx
import React, { useState, useEffect } from 'react';
import { Bell, BellRing } from 'lucide-react';
import { notifySupported, notifyPermission, requestNotifyPermission } from '@/lib/notify';

// Stateless browser-notification toggle for score-change alerts. Requests
// permission only on click (a user gesture, per browser policy) and shows the
// current permission state. Nothing is stored or sent anywhere.
export default function ScoreAlertsToggle() {
  const [permission, setPermission] = useState(() => notifyPermission());

  // Re-check on focus in case the user changed the setting in browser settings.
  useEffect(() => {
    const sync = () => setPermission(notifyPermission());
    window.addEventListener('focus', sync);
    return () => window.removeEventListener('focus', sync);
  }, []);

  if (!notifySupported()) return null;

  const enable = async () => {
    const result = await requestNotifyPermission();
    setPermission(result);
  };

  if (permission === 'granted') {
    return (
      <span
        className="inline-flex items-center gap-2 rounded-lg border border-emerald-200 bg-emerald-50 px-3 py-2 text-sm font-medium text-emerald-700"
        title="You'll get a browser notification when this wallet's score changes while the tab is in the background."
      >
        <BellRing className="w-4 h-4" strokeWidth={1.75} />
        <span className="hidden sm:inline">Alerts on</span>
      </span>
    );
  }

  if (permission === 'denied') {
    return (
      <span
        className="inline-flex items-center gap-2 rounded-lg border border-neutral-200 bg-neutral-100 px-3 py-2 text-sm font-medium text-neutral-400 cursor-not-allowed"
        title="Notifications are blocked for this site. Re-enable them in your browser's site settings."
      >
        <Bell className="w-4 h-4" strokeWidth={1.75} />
        <span className="hidden sm:inline">Blocked</span>
      </span>
    );
  }

  // 'default' — not yet asked.
  return (
    <button
      onClick={enable}
      className="inline-flex items-center gap-2 rounded-lg border border-neutral-200 bg-white px-3 py-2 text-sm font-medium text-neutral-700 hover:bg-neutral-50 transition-colors"
      title="Get a browser notification when this wallet's score changes."
    >
      <Bell className="w-4 h-4" strokeWidth={1.75} />
      <span className="hidden sm:inline">Enable</span>
    </button>
  );
}
```

## src/components/walletxs/ScoreHistory.jsx

```jsx
import React from 'react';
import { tier } from '@/lib/severity';

// A quiet inline sparkline of a wallet's recent score history (localStorage only).
// Renders nothing for fewer than 2 checks — same omit-on-first-check principle
// as DeltaPill. No axes, labels, hover, or tooltips in this iteration.
//
// Direction is encoded per segment: up=emerald, down=red, flat=grey, so a glance
// shows whether the wallet is trending up or down. A net-trend label summarizes
// the overall change from the first to the last check in the window.

const UP = '#10b981';
const DOWN = '#ef4444';
const FLAT = '#a3a3a3';

export default function ScoreHistory({ history }) {
  if (!history || history.length < 2) return null;

  const n = history.length;
  // Map each check to a 0..100 coordinate space; the SVG stretches to fit, so
  // x = index fraction, y = inverted score (100 at top). Dots are rendered as
  // HTML spans over the SVG so they stay perfectly round at any width.
  const pts = history.map((e, i) => ({
    x: (i / (n - 1)) * 100,
    y: (1 - e.score / 100) * 100,
    score: e.score,
  }));

  // One colored segment per consecutive pair: up=emerald, down=red, flat=grey.
  const segments = [];
  for (let i = 1; i < n; i++) {
    const dir =
      history[i].score > history[i - 1].score
        ? UP
        : history[i].score < history[i - 1].score
        ? DOWN
        : FLAT;
    segments.push({
      x1: pts[i - 1].x,
      y1: pts[i - 1].y,
      x2: pts[i].x,
      y2: pts[i].y,
      color: dir,
    });
  }

  // Dots only where the score actually changed from the previous check.
  const dots = [];
  for (let i = 1; i < n; i++) {
    if (history[i].score !== history[i - 1].score) {
      dots.push({ x: pts[i].x, y: pts[i].y, color: tier(history[i].score).ring });
    }
  }

  const net = history[n - 1].score - history[0].score;
  const earliest = history[0].at;
  const latest = history[n - 1].at;
  const days = Math.max(0, Math.round((latest - earliest) / 86400000));
  const span = days === 0 ? 'today' : `the last ${days} day${days === 1 ? '' : 's'}`;

  const trendColor = net > 0 ? 'text-emerald-600' : net < 0 ? 'text-red-600' : 'text-neutral-500';
  const trendArrow = net > 0 ? '↑' : net < 0 ? '↓' : '→';
  const trendLabel =
    net === 0 ? 'no net change' : `${trendArrow} ${Math.abs(net)} pt${Math.abs(net) === 1 ? '' : 's'}`;

  return (
    <div className="rounded-xl border border-[#e5e5e7] bg-[#f5f5f6] px-5 py-4">
      <div className="relative" style={{ height: 40 }}>
        <svg
          viewBox="0 0 100 100"
          preserveAspectRatio="none"
          className="absolute inset-0 h-full w-full"
          aria-hidden="true"
        >
          {segments.map((s, i) => (
            <line
              key={i}
              x1={s.x1}
              y1={s.y1}
              x2={s.x2}
              y2={s.y2}
              stroke={s.color}
              strokeWidth={1.5}
              strokeLinecap="round"
              vectorEffect="non-scaling-stroke"
            />
          ))}
        </svg>
        {dots.map((d, i) => (
          <span
            key={i}
            className="absolute rounded-full"
            style={{
              left: `${d.x}%`,
              top: `${d.y}%`,
              width: 5,
              height: 5,
              background: d.color,
              transform: 'translate(-50%, -50%)',
            }}
          />
        ))}
      </div>
      <div className="mt-2 flex items-center justify-between">
        <p className="text-xs text-neutral-500">
          {n} checks over {span}
        </p>
        <p className={`text-xs font-medium ${trendColor}`}>{trendLabel}</p>
      </div>
    </div>
  );
}
```

## src/components/walletxs/LiveMetricsBanner.jsx

```jsx
import React, { useState } from 'react';
import { ShieldCheck, Lock } from 'lucide-react';
import CountUp from './CountUp';
import StatelessModal from './StatelessModal';
import { MIN_VISIBLE_COUNT } from '@/lib/config';

const GLOW = 'text-cyan-400 [text-shadow:0_0_8px_rgba(56,189,248,0.6)]';

function Divider() {
  return <span className="hidden h-4 w-px bg-[#334155] sm:inline-block" />;
}

// Hero metrics banner: a live read-only badge, the all-time "Audits Run"
// total, and a clickable stateless-privacy link.
//
// Visibility: renders nothing — no placeholder, no loading state, no "coming
// soon" — until the cumulative total crosses MIN_VISIBLE_COUNT, and nothing
// while the stats endpoint is blocked/offline. Once the total crosses the
// threshold it only ever goes up, so the banner stays visible from then on.
export default function LiveMetricsBanner({ totalAudits, available }) {
  const [modalOpen, setModalOpen] = useState(false);

  if (!available) return null;
  if (totalAudits === null) return null;
  if (totalAudits < MIN_VISIBLE_COUNT) return null;

  return (
    <>
      <div className="flex flex-wrap items-center justify-center gap-x-4 gap-y-2 rounded-xl border border-[#334155] bg-[#1e293b]/90 px-4 py-3 text-sm backdrop-blur">
        {/* Live badge — pulsating green LED */}
        <span className="inline-flex items-center gap-2 text-slate-300">
          <span className="relative flex h-2.5 w-2.5">
            <span className="absolute inline-flex h-full w-full animate-ping rounded-full bg-green-400 opacity-75" />
            <span className="relative inline-flex h-2.5 w-2.5 rounded-full bg-green-500" />
          </span>
          Live Read-Only Queries
        </span>

        <Divider />

        <span className="inline-flex items-center gap-1.5 text-slate-300">
          <ShieldCheck className="w-4 h-4 text-cyan-400" strokeWidth={1.75} />
          <CountUp value={totalAudits} className={`font-bold tabular-nums ${GLOW}`} />
          <span>Audits Run</span>
        </span>

        <Divider />

        <button
          type="button"
          onClick={() => setModalOpen(true)}
          className="inline-flex items-center gap-1.5 text-slate-300 transition-colors hover:text-cyan-400"
        >
          <Lock className="w-4 h-4" strokeWidth={1.75} />
          100% Stateless &amp; Private
        </button>
      </div>

      <StatelessModal open={modalOpen} onOpenChange={setModalOpen} />
    </>
  );
}
```

## src/components/walletxs/CountUp.jsx

```jsx
import React, { useEffect, useRef, useState } from 'react';

// Smoothly animates a number from its previous value to the next over ~1s
// (easeOutCubic). Renders null until a real value arrives so we never flash 0
// for data that is still loading.
export default function CountUp({ value, duration = 1000, className }) {
  const [display, setDisplay] = useState(null);
  const fromRef = useRef(0);
  const rafRef = useRef(null);

  useEffect(() => {
    if (value === null || value === undefined) return;
    const from = fromRef.current;
    const start = performance.now();
    const tick = (now) => {
      const t = Math.min(1, (now - start) / duration);
      const eased = 1 - Math.pow(1 - t, 3);
      setDisplay(Math.round(from + (value - from) * eased));
      if (t < 1) {
        rafRef.current = requestAnimationFrame(tick);
      } else {
        fromRef.current = value;
      }
    };
    rafRef.current = requestAnimationFrame(tick);
    return () => {
      if (rafRef.current) cancelAnimationFrame(rafRef.current);
    };
  }, [value, duration]);

  if (display === null) return <span className={className}>—</span>;
  return <span className={className}>{display.toLocaleString()}</span>;
}
```

## src/components/walletxs/StatelessModal.jsx

```jsx
import React from 'react';
import {
  Dialog,
  DialogContent,
  DialogHeader,
  DialogTitle,
  DialogDescription,
} from '@/components/ui/dialog';
import { ShieldCheck, Github } from 'lucide-react';
import { GITHUB_REPO_URL } from '@/lib/config';

// Explains the stateless architecture to grant evaluators and Web3 users.
// Triggered from the "🔒 100% Stateless & Private" link in the live banner.
export default function StatelessModal({ open, onOpenChange }) {
  return (
    <Dialog open={open} onOpenChange={onOpenChange}>
      <DialogContent className="max-w-md border-[#334155] bg-[#1e293b] text-slate-200">
        <DialogHeader>
          <DialogTitle className="flex items-center gap-2 text-cyan-400">
            <ShieldCheck className="w-5 h-5" strokeWidth={1.75} />
            100% Stateless & Private
          </DialogTitle>
          <DialogDescription className="text-slate-400">
            How WalletXS protects your privacy by design.
          </DialogDescription>
        </DialogHeader>
        <p className="mt-2 text-sm leading-relaxed text-slate-300">
          WalletXS is 100% client-side and stateless. We do not run databases,
          store cookies, or log IP addresses or wallet queries. The audit
          counter works via an anonymous, payload-free signal that increments
          a single global integer. Inspect our source code on GitHub to verify.
        </p>
        <a
          href={GITHUB_REPO_URL}
          target="_blank"
          rel="noopener noreferrer"
          className="mt-4 inline-flex items-center gap-2 text-sm font-medium text-cyan-400 hover:text-cyan-300 transition-colors"
        >
          <Github className="w-4 h-4" strokeWidth={1.75} />
          View source on GitHub
        </a>
      </DialogContent>
    </Dialog>
  );
}
```

## src/components/walletxs/TipJar.jsx

```jsx
import React, { useState, useEffect } from 'react';
import { Heart, Copy, Check, QrCode, Coffee } from 'lucide-react';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import {
  Dialog,
  DialogContent,
  DialogHeader,
  DialogTitle,
  DialogTrigger,
} from '@/components/ui/dialog';
import { DEV_TIP_ADDRESS, USDC_CONTRACT, KOFI_URL } from '@/lib/config';

const PRESETS = [5, 10, 25];

// ABI encoding helpers for a bare eth_sendTransaction (no web3 library needed).
const encAddr = (addr) => addr.replace(/^0x/, '').toLowerCase().padStart(64, '0');
const encUint = (bn) => bn.toString(16).padStart(64, '0');
const TRANSFER_SELECTOR = '0xa9059cbb';

// Convert a decimal ETH amount (number or string) to hex wei using BigInt
// string math. The integer and fractional parts are parsed separately and
// assembled as wei with BigInt — never multiplying a float by 1e18, which
// overflows Number.MAX_SAFE_INTEGER above ~9 ETH and silently loses precision
// (real wei, sent from the user's own wallet).
const ethToWei = (eth) => {
  // Normalize scientific-notation numbers (e.g. 1e-9) to a plain decimal string
  // before splitting, so very small amounts parse correctly.
  const str = typeof eth === 'number' ? eth.toFixed(18).replace(/\.?0+$/, '') : String(eth);
  const [intPart = '', fracPart = ''] = str.split('.');
  const intWei = BigInt(intPart || '0') * 10n ** 18n;
  const f = fracPart.slice(0, 18); // truncate to wei precision
  const fracWei = f ? BigInt(f) * 10n ** BigInt(18 - f.length) : 0n;
  return '0x' + (intWei + fracWei).toString(16);
};

// USD → wei using integer math end to end. usdCents and priceCents are small
// (cents, well within safe-integer range for any realistic tip), so the
// BigInt product stays exact; integer division floors so the user never
// overpays. With no price available, the USD value is sent as raw ETH.
// Both branches return a hex string — eth_sendTransaction's `value` field
// must be a hex-encoded string per EIP-1193, not a raw BigInt.
const usdToWei = (usd, priceUsd) => {
  const usdCents = BigInt(Math.round(usd * 100));
  if (!priceUsd) return ethToWei(usd);
  const priceCents = BigInt(Math.round(priceUsd * 100));
  return '0x' + ((usdCents * 10n ** 18n) / priceCents).toString(16);
};

export default function TipJar() {
  const [open, setOpen] = useState(false);
  const [asset, setAsset] = useState('USDC');
  const [usd, setUsd] = useState(5);
  const [custom, setCustom] = useState('');
  const [status, setStatus] = useState('idle'); // idle | pending | success | error
  const [error, setError] = useState('');
  const [ethPrice, setEthPrice] = useState(null);
  const [copied, setCopied] = useState(false);
  const [showQr, setShowQr] = useState(false);

  const hasWallet = typeof window !== 'undefined' && !!window.ethereum;

  // Fetch ETH/USD price only when needed for ETH tips.
  useEffect(() => {
    if (!open || asset !== 'ETH' || ethPrice !== null) return;
    fetch('https://api.coingecko.com/api/v3/simple/price?ids=ethereum&vs_currencies=usd')
      .then((r) => r.json())
      .then((d) => setEthPrice(d?.ethereum?.usd || null))
      .catch(() => setEthPrice(null));
  }, [open, asset, ethPrice]);

  const amountUsd = custom ? parseFloat(custom) : usd;

  const reset = () => {
    setStatus('idle');
    setError('');
    setCustom('');
    setShowQr(false);
  };

  const sendTip = async () => {
    if (!hasWallet) return;
    const value = Number(amountUsd);
    if (!value || value <= 0) {
      setError('Enter an amount');
      return;
    }
    setStatus('pending');
    setError('');
    try {
      const accounts = await window.ethereum.request({ method: 'eth_requestAccounts' });
      const from = accounts[0];
      let txHash;
      if (asset === 'ETH') {
        const wei = usdToWei(value, ethPrice);
        txHash = await window.ethereum.request({
          method: 'eth_sendTransaction',
          params: [{ from, to: DEV_TIP_ADDRESS, value: wei }],
        });
      } else {
        const units = BigInt(Math.round(value * 1e6));
        const data = TRANSFER_SELECTOR + encAddr(DEV_TIP_ADDRESS) + encUint(units);
        txHash = await window.ethereum.request({
          method: 'eth_sendTransaction',
          params: [{ from, to: USDC_CONTRACT, data }],
        });
      }
      setStatus('success');
    } catch (e) {
      setStatus('error');
      const msg = e?.message || '';
      setError(/reject|denied|cancel/i.test(msg) ? 'Transaction rejected' : msg || 'Transaction failed');
    }
  };

  const copyAddress = async () => {
    try {
      await navigator.clipboard.writeText(DEV_TIP_ADDRESS);
      setCopied(true);
      setTimeout(() => setCopied(false), 2000);
    } catch {
      /* clipboard unavailable */
    }
  };

  const qrSrc = `https://api.qrserver.com/v1/create-qr-code/?size=240x240&margin=0&data=ethereum:${DEV_TIP_ADDRESS}`;

  return (
    <Dialog open={open} onOpenChange={(o) => { setOpen(o); if (!o) reset(); }}>
      <DialogTrigger asChild>
        <Button
          className="w-full h-11 border border-blue-200 bg-blue-50 text-blue-700 hover:bg-blue-100 hover:border-blue-300"
        >
          <Heart className="w-4 h-4 text-pink-500" strokeWidth={2} />
          Tip the developer
        </Button>
      </DialogTrigger>
      <DialogContent className="max-w-sm">
        <DialogHeader>
          <DialogTitle className="flex items-center gap-2">
            <Coffee className="w-4 h-4 text-rose-500" strokeWidth={2} />
            Support WalletXS
          </DialogTitle>
        </DialogHeader>

        {status !== 'success' && (
          <>
            <Button asChild variant="outline" className="w-full h-11 border-neutral-200 bg-white hover:bg-neutral-50">
              <a href={KOFI_URL} target="_blank" rel="noopener noreferrer">
                <Coffee className="w-4 h-4 text-rose-500" strokeWidth={2} />
                Tip with Ko-fi
              </a>
            </Button>
            <div className="relative my-1">
              <div className="absolute inset-0 flex items-center">
                <span className="w-full border-t border-neutral-200" />
              </div>
              <div className="relative flex justify-center text-xs">
                <span className="bg-background px-2 text-neutral-400">or send crypto</span>
              </div>
            </div>
          </>
        )}

        {status === 'success' ? (
          <div className="py-6 text-center">
            <div className="mx-auto flex h-12 w-12 items-center justify-center rounded-full bg-emerald-50">
              <Check className="h-6 w-6 text-emerald-600" strokeWidth={2} />
            </div>
            <p className="mt-4 text-sm font-medium text-neutral-900">
              Thank you for supporting open-source security!
            </p>
            <Button
              onClick={() => { reset(); setOpen(false); }}
              className="mt-5 w-full bg-neutral-900 hover:bg-neutral-800"
            >
              Done
            </Button>
          </div>
        ) : !hasWallet ? (
          <div className="space-y-4 py-2">
            <p className="text-sm text-neutral-600">
              No Web3 wallet detected. You can still tip by sending to the developer
              address below.
            </p>
            <div className="rounded-lg border border-neutral-200 bg-neutral-50 p-3">
              <div className="text-[11px] uppercase tracking-wide text-neutral-400">
                Developer address
              </div>
              <div className="mt-1 font-mono text-xs break-all text-neutral-800">
                {DEV_TIP_ADDRESS}
              </div>
            </div>
            <div className="flex gap-2">
              <Button
                onClick={copyAddress}
                variant="outline"
                className="flex-1 border-neutral-200 bg-white"
              >
                {copied ? <Check className="w-4 h-4 text-emerald-600" /> : <Copy className="w-4 h-4" />}
                {copied ? 'Copied' : 'Copy address'}
              </Button>
              <Button
                onClick={() => setShowQr((v) => !v)}
                variant="outline"
                className="flex-1 border-neutral-200 bg-white"
              >
                <QrCode className="w-4 h-4" />
                {showQr ? 'Hide QR' : 'Show QR'}
              </Button>
            </div>
            {showQr && (
              <div className="flex justify-center rounded-lg border border-neutral-200 bg-white p-3">
                <img src={qrSrc} alt="Tip address QR code" width={200} height={200} />
              </div>
            )}
          </div>
        ) : (
          <div className="space-y-4 py-2">
            {/* Asset selector */}
            <div className="grid grid-cols-2 gap-2">
              {['USDC', 'ETH'].map((a) => (
                <button
                  key={a}
                  onClick={() => setAsset(a)}
                  className={`rounded-lg border px-3 py-2 text-sm font-medium transition-colors ${
                    asset === a
                      ? 'border-neutral-900 bg-neutral-900 text-white'
                      : 'border-neutral-200 bg-white text-neutral-700 hover:bg-neutral-50'
                  }`}
                >
                  {a}
                </button>
              ))}
            </div>

            {/* Preset amounts */}
            <div className="grid grid-cols-3 gap-2">
              {PRESETS.map((p) => (
                <button
                  key={p}
                  onClick={() => { setUsd(p); setCustom(''); }}
                  className={`rounded-lg border px-3 py-2 text-sm font-medium transition-colors ${
                    !custom && usd === p
                      ? 'border-neutral-900 bg-neutral-900 text-white'
                      : 'border-neutral-200 bg-white text-neutral-700 hover:bg-neutral-50'
                  }`}
                >
                  ${p}
                </button>
              ))}
            </div>

            {/* Custom amount */}
            <div>
              <Input
                type="number"
                min="0"
                step="0.01"
                value={custom}
                onChange={(e) => setCustom(e.target.value)}
                placeholder="Custom amount (USD)"
                className="font-mono text-sm bg-white border-neutral-200"
              />
              {asset === 'ETH' && ethPrice === null && (
                <p className="mt-1.5 text-xs text-neutral-400">
                  Fetching ETH price… custom amount will be sent as ETH if unavailable.
                </p>
              )}
            </div>

            {error && <p className="text-sm text-red-600">{error}</p>}

            <Button
              onClick={sendTip}
              disabled={status === 'pending'}
              className="w-full h-11 bg-neutral-900 hover:bg-neutral-800"
            >
              {status === 'pending' ? (
                <>
                  <span className="mr-2 h-4 w-4 animate-spin rounded-full border-2 border-white/40 border-t-white" />
                  Waiting for confirmation…
                </>
              ) : (
                <>Tip ${amountUsd || 0} {asset}</>
              )}
            </Button>

            <p className="text-center text-xs text-neutral-400">
              Tip sent directly on-chain to the developer. No fees taken by WalletXS.
            </p>
          </div>
        )}
      </DialogContent>
    </Dialog>
  );
}
```

## src/components/walletxs/WalletList.jsx

```jsx
import React, { useRef, useState } from 'react';
import { Trash2, Eye, Download, Upload } from 'lucide-react';
import AddressEntry from '@/components/walletxs/AddressEntry';
import { lastCheckFor } from '@/lib/storage';
import { exportBackup, importBackup } from '@/lib/backup';
import { tier, shortAddr } from '@/lib/severity';

function timeAgo(at) {
  const days = Math.max(0, Math.round((Date.now() - at) / 86400000));
  if (days === 0) return 'checked today';
  if (days === 1) return 'checked 1 day ago';
  return `checked ${days} days ago`;
}

// The address-book screen. Shows each tracked wallet with its cached
// last-known score (NO auto-scan of every wallet — only the one the user
// selects to view gets a fresh scan, handled by the parent). Adding a new
// wallet reuses AddressEntry's validation + connect flow. Export/import
// backs the tracked list up to / restores from a JSON file.
const VISIBLE_CAP = 6;

export default function WalletList({ tracked, onSelect, onRemove, onAdd, onImported }) {
  const fileRef = useRef(null);
  const [importMsg, setImportMsg] = useState(null); // { type, text }
  const [showAll, setShowAll] = useState(false);

  // Sort worst-score-first (ascending). Wallets with no previous check
  // (lastCheckFor returns null) sort to the end — an unknown status isn't
  // more urgent than a confirmed low score.
  const sorted = [...tracked].sort((a, b) => {
    const sa = lastCheckFor(a.address);
    const sb = lastCheckFor(b.address);
    if (!sa && !sb) return 0;
    if (!sa) return 1;
    if (!sb) return -1;
    return sa.score - sb.score;
  });

  const visible = showAll ? sorted : sorted.slice(0, VISIBLE_CAP);
  const hiddenCount = sorted.length - VISIBLE_CAP;

  const handleImport = async (e) => {
    const file = e.target.files?.[0];
    e.target.value = ''; // allow re-importing the same file later
    if (!file) return;
    try {
      const { added, duplicate, invalid } = await importBackup(file);
      const parts = [`Imported ${added} wallet${added === 1 ? '' : 's'}`];
      if (duplicate) parts.push(`${duplicate} already tracked`);
      if (invalid) parts.push(`${invalid} skipped`);
      setImportMsg({ type: 'ok', text: parts.join(' · ') });
      onImported && onImported();
    } catch (err) {
      setImportMsg({ type: 'error', text: err.message || 'Import failed.' });
    }
    setTimeout(() => setImportMsg(null), 5000);
  };

  return (
    <div className="space-y-6">
      <div>
        <h1 className="text-xl font-semibold tracking-tight text-neutral-900">Your wallets</h1>
        <p className="mt-1.5 text-sm text-neutral-500">
          Select a wallet to run a fresh check, or add another below.
        </p>
      </div>

      {tracked.length > 0 && (
        <>
          <ul className="space-y-2">
            {visible.map((w) => {
              const last = lastCheckFor(w.address);
              const t = tier(last ? last.score : null);
              return (
                <li
                  key={w.address}
                  className="flex items-center justify-between rounded-xl border border-[#e5e5e7] bg-[#f5f5f6] px-4 py-3"
                >
                  <div className="flex min-w-0 items-center gap-3">
                    <span className={`h-2.5 w-2.5 shrink-0 rounded-full ${last ? t.dot : 'bg-neutral-300'}`} />
                    <div className="min-w-0">
                      <div className="flex items-center gap-2">
                        <span className="font-mono text-sm text-neutral-900">{shortAddr(w.address)}</span>
                        {w.label && (
                          <span className="truncate text-xs text-neutral-500">{w.label}</span>
                        )}
                      </div>
                      <div className="text-xs text-neutral-500">
                        {last
                          ? `${last.score} · ${t.label} · ${timeAgo(last.at)}`
                          : 'not yet checked'}
                      </div>
                    </div>
                  </div>
                  <div className="flex shrink-0 items-center gap-1">
                    <button
                      onClick={() => onSelect(w.address)}
                      className="inline-flex items-center gap-1.5 rounded-lg border border-neutral-200 bg-white px-2.5 py-1.5 text-xs font-medium text-neutral-700 hover:bg-neutral-50 transition-colors"
                    >
                      <Eye className="w-3.5 h-3.5" strokeWidth={1.75} />
                      View
                    </button>
                    <button
                      onClick={() => onRemove(w.address)}
                      title="Stop tracking this wallet (history is kept)"
                      className="inline-flex items-center justify-center rounded-lg border border-neutral-200 bg-white p-1.5 text-neutral-400 transition-colors hover:border-red-200 hover:text-red-600"
                    >
                      <Trash2 className="w-3.5 h-3.5" strokeWidth={1.75} />
                    </button>
                  </div>
                </li>
              );
            })}
          </ul>
          {hiddenCount > 0 && !showAll && (
            <button
              onClick={() => setShowAll(true)}
              className="text-xs font-medium text-neutral-500 hover:text-neutral-900 transition-colors"
            >
              Show all {sorted.length} wallets
            </button>
          )}
        </>
      )}

      <div className="flex items-center gap-2">
        <button
          onClick={exportBackup}
          className="inline-flex items-center gap-1.5 rounded-lg border border-neutral-200 bg-white px-3 py-1.5 text-xs font-medium text-neutral-700 hover:bg-neutral-50 transition-colors"
        >
          <Download className="w-3.5 h-3.5" strokeWidth={1.75} />
          Export
        </button>
        <button
          onClick={() => fileRef.current?.click()}
          className="inline-flex items-center gap-1.5 rounded-lg border border-neutral-200 bg-white px-3 py-1.5 text-xs font-medium text-neutral-700 hover:bg-neutral-50 transition-colors"
        >
          <Upload className="w-3.5 h-3.5" strokeWidth={1.75} />
          Import
        </button>
        <input
          ref={fileRef}
          type="file"
          accept="application/json,.json"
          onChange={handleImport}
          className="hidden"
        />
      </div>
      {importMsg && (
        <p
          className={`text-xs ${importMsg.type === 'ok' ? 'text-emerald-600' : 'text-red-600'}`}
        >
          {importMsg.text}
        </p>
      )}

      <AddressEntry onSubmit={onAdd} />
    </div>
  );
}
```

## src/components/walletxs/Footer.jsx

```jsx
import React from 'react';
import { Link } from 'react-router-dom';
import { Github } from 'lucide-react';
import { GITHUB_REPO_URL } from '@/lib/config';

// Quiet, always-present footer. The only always-visible path to Methodology
// and Privacy — PercentileLine only renders after a completed scan, so this
// guarantees both pages (and the source link) are reachable in every state.
export default function Footer() {
  return (
    <footer className="mt-16 flex flex-wrap items-center justify-center gap-x-4 gap-y-2 text-xs text-neutral-500">
      <Link to="/methodology" className="hover:text-neutral-900 transition-colors">
        Methodology
      </Link>
      <span className="text-neutral-300" aria-hidden="true">·</span>
      <Link to="/privacy" className="hover:text-neutral-900 transition-colors">
        Privacy
      </Link>
      <span className="text-neutral-300" aria-hidden="true">·</span>
      <a
        href={GITHUB_REPO_URL}
        target="_blank"
        rel="noopener noreferrer"
        className="inline-flex items-center gap-1 hover:text-neutral-900 transition-colors"
      >
        <Github className="w-3 h-3" strokeWidth={1.75} />
        Source
      </a>
    </footer>
  );
}
```

## src/components/walletxs/CopyMarkdownButton.jsx

```jsx
import React, { useState } from 'react';
import { Copy, Check } from 'lucide-react';
import { tier } from '@/lib/severity';
import { SUB_SCORE_LABELS } from '@/lib/scoring';
import { percentileFor, SAMPLE_SIZE } from '@/lib/percentile';

// Builds a Markdown string from the current score report, so it can be pasted
// into any tool that accepts Markdown (Notion, Obsidian, a GitHub issue, …)
// without this app integrating with any of them.
function buildMarkdown({ address, result, findings }) {
  const lines = [];
  const overall = result.overall;
  const overallTier = tier(overall);

  lines.push('# WalletXS Exposure Report');
  lines.push('');
  lines.push(`**Wallet:** \`${address}\``);
  lines.push(
    `**Overall score:** ${overall !== null ? `${overall} (${overallTier.label})` : 'Unavailable'}`
  );
  const pct = percentileFor(overall);
  if (pct !== null) {
    lines.push(`**Percentile:** safer than ${pct}% of ${SAMPLE_SIZE.toLocaleString()} sampled wallets`);
  }
  lines.push('');

  lines.push('## Sub-scores');
  lines.push('');
  for (const key of Object.keys(SUB_SCORE_LABELS)) {
    const s = result.subs?.[key];
    const label = SUB_SCORE_LABELS[key];
    if (!s || s.unavailable) {
      lines.push(`- **${label}:** Unavailable`);
    } else {
      lines.push(`- **${label}:** ${s.score} (${tier(s.score).label})`);
    }
  }
  lines.push('');

  if (findings && findings.length > 0) {
    lines.push('## Current findings');
    lines.push('');
    for (const f of findings) lines.push(`- ${f.text}`);
    lines.push('');
  }

  lines.push('---');
  lines.push('');
  const methodologyUrl = `${window.location.origin}/methodology`;
  lines.push(
    `_Generated by WalletXS on ${new Date().toLocaleString()} — not a safety guarantee. See ${methodologyUrl} for details._`
  );
  return lines.join('\n');
}

export default function CopyMarkdownButton({ address, result, findings }) {
  const [copied, setCopied] = useState(false);

  const copy = async () => {
    const md = buildMarkdown({ address, result, findings });
    try {
      await navigator.clipboard.writeText(md);
      setCopied(true);
      setTimeout(() => setCopied(false), 2000);
    } catch {
      /* clipboard unavailable */
    }
  };

  return (
    <button
      onClick={copy}
      className="inline-flex items-center gap-1.5 rounded-lg border border-neutral-200 bg-white px-3 py-1.5 text-xs font-medium text-neutral-700 hover:bg-neutral-50 transition-colors"
    >
      {copied ? (
        <Check className="w-3.5 h-3.5 text-emerald-600" strokeWidth={1.75} />
      ) : (
        <Copy className="w-3.5 h-3.5" strokeWidth={1.75} />
      )}
      {copied ? 'Copied' : 'Copy as Markdown'}
    </button>
  );
}
```

---

_Continued in **WalletXS-Source-Part3.md** (pages + dependencies)._
