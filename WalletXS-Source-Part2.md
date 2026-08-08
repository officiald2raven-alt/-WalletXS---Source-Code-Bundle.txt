# WalletXS — GitHub Source Bundle (Part 2 of 2)

Components, pages, and dependencies. See **WalletXS-Source-Part1.md** for config / build files / public assets / hooks / lib.

---

## src/components/walletxs/Header.jsx

```jsx
import React from 'react';
import { Shield, KeyRound, Github } from 'lucide-react';
import { Button } from '@/components/ui/button';
import InstallButton from '@/components/walletxs/InstallButton';
import ScoreAlertsToggle from '@/components/walletxs/ScoreAlertsToggle';
import { GITHUB_REPO_URL } from '@/lib/config';

export default function Header({ address, apiKey, onKeyChange }) {
  return (
    <header className="flex flex-wrap items-center justify-between gap-3 py-6">
      <div className="flex items-center gap-2">
        <Shield className="w-5 h-5 text-neutral-900" strokeWidth={1.75} />
        <span className="text-lg font-semibold tracking-tight text-neutral-900">WalletXS</span>
      </div>
      <div className="flex items-center gap-2 sm:gap-3">
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
import { percentileFor, SAMPLE_SIZE } from '@/lib/percentile';

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
        Based on a sample of {SAMPLE_SIZE.toLocaleString()} public wallets, updated periodically
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
const toHexWei = (eth) => '0x' + BigInt(Math.round(eth * 1e18)).toString(16);

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
        const ethAmount = ethPrice ? value / ethPrice : value;
        txHash = await window.ethereum.request({
          method: 'eth_sendTransaction',
          params: [{ from, to: DEV_TIP_ADDRESS, value: toHexWei(ethAmount) }],
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

## src/pages/Home.jsx

```jsx
import React, { useCallback, useEffect, useState } from 'react';
import Header from '@/components/walletxs/Header';
import AddressEntry from '@/components/walletxs/AddressEntry';
import ApiKeyEntry from '@/components/walletxs/ApiKeyEntry';
import ScoreCard from '@/components/walletxs/ScoreCard';
import SubScoreGrid from '@/components/walletxs/SubScoreGrid';
import WhatChanged from '@/components/walletxs/WhatChanged';
import PercentileLine from '@/components/walletxs/PercentileLine';
import ScoreHistory from '@/components/walletxs/ScoreHistory';
import { computeScore, counterfactualGain } from '@/lib/scoring';
import { collectWalletData, lookupPublicExposure } from '@/lib/chain';
import {
  loadAddress,
  saveAddress,
  clearAddress,
  lastCheckFor,
  recordCheck,
  historyFor,
  loadApiKey,
  saveApiKey,
  clearApiKey,
} from '@/lib/storage';
import { notifyScoreChange } from '@/lib/notify';
import { shortAddr } from '@/lib/severity';
import { useLiveMetrics } from '@/hooks/useLiveMetrics';
import { useScheduledCheck } from '@/hooks/useScheduledCheck';
import LiveMetricsBanner from '@/components/walletxs/LiveMetricsBanner';
import TipJar from '@/components/walletxs/TipJar';

export default function Home() {
  const [address, setAddress] = useState(() => loadAddress());
  const [apiKey, setApiKey] = useState(() => loadApiKey());
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);
  const [lastCheck, setLastCheck] = useState(null);
  const [findings, setFindings] = useState([]);
  const [limitedDepth, setLimitedDepth] = useState(false);
  const [scoreHistory, setScoreHistory] = useState([]);
  const { totalAudits, available, increment: incrementMetrics } = useLiveMetrics();

  const run = useCallback(
    async (addr, key) => {
      setLoading(true);
      setResult(null);
      const previous = lastCheckFor(addr);
      setLastCheck(previous);

      const data = await collectWalletData(addr, key);
      setLimitedDepth(!!data.limitedDepth);
      const osintMatches = await lookupPublicExposure(addr);
      const inputs = { ...data, osintMatches, now: Date.now() };
      const scored = computeScore(inputs);

      // Current risky-approval findings for this check.
      const currentFindings = (data.approvals || [])
        .filter((a) => a.flagged || a.unlimited)
        .map((a) => ({
          id: a.key,
          owner: addr,
          spender: a.spender,
          gain: counterfactualGain(inputs, a.key),
          text: a.flagged
            ? `Contract ${a.spender} is on a phishing blocklist. You have ${
                a.unlimited ? 'an unlimited' : 'an active'
              } approval to it.`
            : `You have an unlimited approval to ${a.spender}, which can move that token from your wallet at any time.`,
        }));

      // Real diff: only findings whose approval key was NOT present last check.
      const prevKeys = new Set((previous?.findings || []).map((f) => f.id));
      const newFindings = currentFindings.filter((f) => !prevKeys.has(f.id));

      const dropped = previous && scored.overall !== null && scored.overall < previous.score;
      if (dropped) {
        if (newFindings.length > 0) {
          setFindings(newFindings);
        } else {
          // Score dropped with no new approvals → it's approval aging, not a new risk.
          setFindings([
            {
              id: '__aging__',
              text: 'Your score dropped due to approval aging — no new risky approvals since your last check.',
            },
          ]);
        }
      } else {
        setFindings([]);
      }

      setResult(scored);
      recordCheck(addr, scored.overall, currentFindings);
      setScoreHistory(historyFor(addr, 10));
      // Anonymous, stateless audit ping — fires only on a completed scan (a
      // real score was produced), never on a failed/empty scan or on page load.
      if (scored.overall !== null) incrementMetrics();
      // Stateless browser alert: notify only on a real score change vs the
      // prior check, and only while the tab is in the background (no noise
      // when the user is already looking at the score).
      if (previous && scored.overall !== null && scored.overall !== previous.score) {
        notifyScoreChange(addr, previous.score, scored.overall);
      }
      setLoading(false);
    },
    [incrementMetrics]
  );

  useScheduledCheck(address, apiKey, run);

  useEffect(() => {
    if (address) run(address, apiKey);
  }, [address, apiKey, run]);

  const onSubmit = (addr) => {
    saveAddress(addr);
    setAddress(addr);
  };

  const onChange = () => {
    clearAddress();
    setAddress(null);
    setResult(null);
    setFindings([]);
    setLimitedDepth(false);
    setScoreHistory([]);
  };

  const onKeySaved = (key) => {
    saveApiKey(key);
    setApiKey(key);
  };

  const onKeyChange = () => {
    clearApiKey();
    setApiKey(null);
    setResult(null);
    setFindings([]);
  };

  return (
    <div className="min-h-screen bg-[#fafafa] text-neutral-900">
      <div className="mx-auto max-w-2xl px-5 pb-24">
        <Header address={address} apiKey={apiKey} onKeyChange={onKeyChange} />

        <div className="mt-2 mb-6">
          <LiveMetricsBanner
            totalAudits={totalAudits}
            available={available}
          />
        </div>

        {address && (
          <div className="mb-4 flex items-center justify-between">
            <span className="font-mono text-xs text-neutral-500">{shortAddr(address)}</span>
            <button
              onClick={onChange}
              className="text-xs font-medium text-neutral-500 hover:text-neutral-900 transition-colors"
            >
              Change wallet
            </button>
          </div>
        )}

        {!address && (
          <>
            {!apiKey && (
              <>
                <ApiKeyEntry onSaved={onKeySaved} />
                <p className="mt-2 text-xs text-neutral-500">
                  A key is optional. Without one, WalletXS still scans but only sees the last ~55
                  days of approvals — add a key for full lifetime history.
                </p>
              </>
            )}
            <div className={apiKey ? '' : 'mt-6'}>
              <AddressEntry onSubmit={onSubmit} />
            </div>
          </>
        )}

        {address && loading && (
          <div className="rounded-xl border border-[#e5e5e7] bg-[#f5f5f6] px-6 py-20 text-center">
            <div className="mx-auto w-6 h-6 border-2 border-neutral-200 border-t-neutral-800 rounded-full animate-spin" />
            <p className="mt-4 text-sm text-neutral-500">
              {apiKey
                ? 'Reading lifetime approvals and blocklist data from Etherscan…'
                : 'Reading approvals and blocklist data (limited ~55-day window)…'}
            </p>
          </div>
        )}

        {address && !loading && result && (
          <>
            <ScoreCard score={result.overall} lastCheck={lastCheck} />
            <div className="mt-3">
              <ScoreHistory history={scoreHistory} />
            </div>
            {limitedDepth && (
              <div className="mt-3 rounded-lg border border-amber-200 bg-amber-50 px-4 py-2.5 text-xs text-amber-800">
                Limited scan depth (~55 days) — Approval staleness and Historical contract exposure
                are based on the last ~55 days only. Add an Etherscan API key for full lifetime
                history.
              </div>
            )}
            <div className="mt-4">
              <TipJar />
            </div>
            <SubScoreGrid subs={result.subs} />
            <WhatChanged findings={findings} />
            <PercentileLine score={result.overall} />
          </>
        )}
      </div>
    </div>
  );
}
```

## src/pages/Methodology.jsx

```jsx
import React from 'react';
import { Link } from 'react-router-dom';
import { ArrowLeft } from 'lucide-react';
import { SAMPLE_SIZE, SAMPLE_UPDATED } from '@/lib/percentile';

const Section = ({ title, children }) => (
  <section className="mt-8">
    <h2 className="text-sm font-semibold uppercase tracking-[0.14em] text-neutral-500">{title}</h2>
    <div className="mt-2 space-y-3 text-sm leading-relaxed text-neutral-800">{children}</div>
  </section>
);

export default function Methodology() {
  return (
    <div className="min-h-screen bg-[#fafafa]">
      <div className="mx-auto max-w-2xl px-5 pb-24">
        <Link
          to="/"
          className="inline-flex items-center gap-1.5 py-6 text-sm text-neutral-500 hover:text-neutral-900 transition-colors"
        >
          <ArrowLeft className="w-4 h-4" strokeWidth={1.75} /> Back
        </Link>

        <h1 className="text-2xl font-semibold tracking-tight text-neutral-900">Methodology</h1>

        <Section title="What is queried">
          <p>
            Approval history is fetched from the Etherscan API, in your browser, each time you run
            a check. Etherscan indexes full lifetime event logs, so staleness and historical
            exposure can see approvals years old — not just the last few weeks. Nothing is computed
            on a server and no score is stored anywhere but your own browser.
          </p>
          <p>
            The Etherscan key used by the app ships in the client bundle and is shared across all
            users; it is a developer key, not a user credential, and is subject to Etherscan's rate
            limits. You can also supply your own free key, which takes precedence. If no key is
            configured or the Etherscan request fails, the app falls back to a keyless public-RPC
            scan covering roughly the last 55 days, and the affected sub-scores are visibly marked
            "Limited scan depth (~55 days)" so a partial scan is never shown as a complete one.
          </p>
          <p>
            Contract addresses are cross-referenced against the ScamSniffer open scam-address
            database, fetched from its public GitHub repository. That list is maintained by a third
            party and we assume it refreshes on the order of days. We do not control when an address
            is added, and a malicious contract may not be on it yet.
          </p>
          <p>
            The wallet address is also screened against the OFAC SDN sanctions list (Ethereum
            addresses), extracted and auto-updated nightly from the official U.S. Treasury list by
            the open-source 0xB10C/ofac-sanctioned-digital-currency-addresses project. The entire
            list is downloaded and matched in your browser — your address never leaves your device
            for this check.
          </p>
        </Section>

        <Section title="What a high score does not mean">
          <p>
            A high score means nothing known-bad was found in the data we checked. It is not a
            safety guarantee. A wallet can score 100 and still be exposed to a contract nobody has
            flagged yet, to a risk outside approvals entirely, or to a compromise of your keys.
          </p>
        </Section>

        <Section title="The percentile figure">
          <p>
            "Safer than X% of scanned wallets" is calculated against a static sample of{' '}
            {SAMPLE_SIZE.toLocaleString()} public wallets bundled with the app, last updated{' '}
            {SAMPLE_UPDATED}. It is not a live comparison against other users, and no other user's
            score is ever sent anywhere.
          </p>
        </Section>

        <Section title="Unavailable sub-scores">
          <p>
            When a data source fails to load, that sub-score reads "Unavailable" and is excluded
            from the overall score rather than filled with a placeholder number. If the OFAC list
            can't be fetched (for example, blocked or offline), public exposure falls back to
            Unavailable and the remaining sub-scores are re-weighted to compute the overall.
          </p>
        </Section>

        <Section title="Third-party privacy">
          <p>
            Your address is sent to Etherscan (when a key is configured) or to the public RPC
            provider (in the limited-depth fallback), and requests are made to the blocklist host
            in order to run a check. Those requests are subject to those third parties' own privacy
            practices, not ours. We have no server and collect nothing.
          </p>
          <p>
            Each scan also sends one anonymous increment ping to the audit-counter endpoint
            (api.walletxs.com/count) so the header can show a live "audits run" total. The ping
            carries no wallet address, no user id, and no identifying data — only an empty counter
            increment. If it is blocked (ad-blocker or offline) the counter simply shows its
            fallback value.
          </p>
        </Section>
      </div>
    </div>
  );
}
```

---

## Dependencies

WalletXS is a Vite + React + Tailwind app. Required npm packages:

```
react, react-dom, react-router-dom
@tanstack/react-query
framer-motion
lucide-react
tailwindcss, tailwindcss-animate
class-variance-authority, clsx, tailwind-merge
@radix-ui/react-dialog, @radix-ui/react-slot
```

### shadcn/ui primitives used

These are standard [shadcn/ui](https://ui.shadcn.com) components and are not included in this bundle. Add them via the shadcn CLI so the imports below resolve:

- `@/components/ui/button` → `Button`
- `@/components/ui/input` → `Input`
- `@/components/ui/dialog` → `Dialog`, `DialogContent`, `DialogHeader`, `DialogTitle`, `DialogDescription`, `DialogTrigger`
- `@/components/ui/toaster` → `Toaster`

### Platform glue (replace with your own or stub out)

`src/App.jsx` imports a few platform wrappers that are outside WalletXS's logic. When porting to a plain Vite project, replace/stub these:

- `@/lib/query-client` → export a `queryClientInstance` from `new QueryClient()`
- `@/lib/AuthContext` → a no-op `AuthProvider` + `useAuth` returning `{ authError: null }`
- `@/components/UserNotRegisteredError` → any fallback component
- `./components/ScrollToTop` → a small component that scrolls to top on route change
- `./lib/PageNotFound` → a 404 page

### Path alias

The `@/` alias maps to `src/`. In `vite.config.js`:

```js
resolve: { alias: { '@': path.resolve(__dirname, 'src') } }
```

---

_Edit `src/lib/config.js` (`DEV_TIP_ADDRESS`, `GITHUB_REPO_URL`, `KOFI_URL`, `ETHERSCAN_API_KEY`) before deploying._
