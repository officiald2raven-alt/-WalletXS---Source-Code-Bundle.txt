# WalletXS — GitHub Source Bundle (Part 3 of 3)

Pages and dependencies. See **WalletXS-Source-Part1.md** for config / build files / public assets / hooks / lib, and **WalletXS-Source-Part2.md** for components.

---

## src/pages/Home.jsx

```jsx
import React, { useCallback, useEffect, useRef, useState } from 'react';
import Header from '@/components/walletxs/Header';
import AddressEntry from '@/components/walletxs/AddressEntry';
import ApiKeyEntry from '@/components/walletxs/ApiKeyEntry';
import WalletList from '@/components/walletxs/WalletList';
import ScoreCard from '@/components/walletxs/ScoreCard';
import SubScoreGrid from '@/components/walletxs/SubScoreGrid';
import WhatChanged from '@/components/walletxs/WhatChanged';
import PercentileLine from '@/components/walletxs/PercentileLine';
import ScoreHistory from '@/components/walletxs/ScoreHistory';
import { computeScore, counterfactualGain, sourceLabel } from '@/lib/scoring';
import { collectWalletData, lookupPublicExposure } from '@/lib/chain';
import {
  loadTrackedAddresses,
  addTrackedAddress,
  removeTrackedAddress,
  loadActiveAddress,
  saveActiveAddress,
  clearActiveAddress,
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
import Footer from '@/components/walletxs/Footer';
import CopyMarkdownButton from '@/components/walletxs/CopyMarkdownButton';
import ApprovalBreakdown from '@/components/walletxs/ApprovalBreakdown';

export default function Home() {
  const [tracked, setTracked] = useState(() => loadTrackedAddresses());
  const [activeAddress, setActiveAddress] = useState(() => loadActiveAddress());
  const [showWalletList, setShowWalletList] = useState(false);
  const [apiKey, setApiKey] = useState(() => loadApiKey());
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);
  const [lastCheck, setLastCheck] = useState(null);
  const [findings, setFindings] = useState([]);
  const [approvalBreakdown, setApprovalBreakdown] = useState([]);
  const [limitedDepth, setLimitedDepth] = useState(false);
  const [scoreHistory, setScoreHistory] = useState([]);
  const { totalAudits, available, increment: incrementMetrics } = useLiveMetrics();

  // Print/export: stamp the report with the generation time, then open the
  // browser print dialog (which can "Save as PDF"). The timestamp + disclaimer
  // live in a print-only block hidden on screen.
  const tsRef = useRef(null);
  const saveReport = () => {
    if (tsRef.current) {
      tsRef.current.textContent = `Report generated: ${new Date().toLocaleString()}`;
    }
    window.print();
  };

  const run = useCallback(
    async (addr, key) => {
      setLoading(true);
      setResult(null);
      const previous = lastCheckFor(addr);
      setLastCheck(previous);

      const data = await collectWalletData(addr, key);
      setLimitedDepth(!!data.limitedDepth);
      const osintMatches = await lookupPublicExposure(addr);
      // Scoring gates on `blocklistAvailable`; treat it as the COMBINED
      // malicious-address check (ScamSniffer OR GoPlus) so one source
      // failing doesn't blank the sub-score when the other is still up.
      const inputs = {
        ...data,
        blocklistAvailable: data.blocklistAvailable || data.goplusAvailable,
        osintMatches,
        now: Date.now(),
      };
      const scored = computeScore(inputs);

      // Sort findings by point-impact (biggest win first). Null or
      // non-positive gains sort to the end rather than break the sort.
      const gainRank = (g) => (g == null || g <= 0 ? -Infinity : g);
      const byGainDesc = (a, b) => gainRank(b.gain) - gainRank(a.gain);

      // Current risky-approval findings for this check, attributed to the
      // real source(s) that flagged each contract.
      const currentFindings = (data.approvals || [])
        .filter((a) => a.flagged || a.unlimited)
        .map((a) => ({
          id: a.key,
          owner: addr,
          spender: a.spender,
          gain: counterfactualGain(inputs, a.key),
          text: a.flagged
            ? `Contract ${a.spender} is flagged by ${sourceLabel(
                a.flaggedScamsniffer,
                a.flaggedGoplus
              )}. You have ${a.unlimited ? 'an unlimited' : 'an active'} approval to it.`
            : `You have an unlimited approval to ${a.spender}, which can move that token from your wallet at any time.`,
        }))
        .sort(byGainDesc);

      // Real diff: only findings whose approval key was NOT present last check.
      const prevKeys = new Set((previous?.findings || []).map((f) => f.id));
      const newFindings = currentFindings.filter((f) => !prevKeys.has(f.id));

      // Full ranked breakdown of EVERY active approval (not just the risky
      // ones), so smaller unflagged capped approvals that still drag the
      // score are visible. Reuses counterfactualGain on already-fetched
      // data — no new network calls.
      const fullBreakdown = (data.approvals || [])
        .map((a) => ({
          key: a.key,
          spender: a.spender,
          token: a.token,
          unlimited: a.unlimited,
          flagged: a.flagged,
          gain: counterfactualGain(inputs, a.key),
        }))
        .sort(byGainDesc);
      setApprovalBreakdown(fullBreakdown);

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

  useScheduledCheck(activeAddress, apiKey, run);

  // Single tracked wallet → go straight to its score (no list, no friction).
  // Runs once on mount.
  useEffect(() => {
    if (tracked.length === 1 && !activeAddress) {
      saveActiveAddress(tracked[0].address);
      setActiveAddress(tracked[0].address);
    }
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, []);

  // Scan the active wallet whenever it changes (or the API key does).
  useEffect(() => {
    if (activeAddress) run(activeAddress, apiKey);
  }, [activeAddress, apiKey, run]);

  // Which screen to show:
  //  - 0 tracked: address entry flow
  //  - 1 tracked: straight to that wallet's score (auto-skip on load)
  //  - >1 tracked: WalletList unless an active wallet is being viewed
  // The "My wallets" header button can force the list open at any tracked
  // count (so a single-wallet user can still add a second one) — that's
  // the showWalletList short-circuit below.
  const screen = (() => {
    if (tracked.length === 0) return 'entry';
    if (showWalletList) return 'list';
    if (tracked.length === 1) return 'score';
    return !activeAddress ? 'list' : 'score';
  })();

  const refreshTracked = () => setTracked(loadTrackedAddresses());

  // Add a wallet (from the entry flow or the WalletList "add" row).
  const onAddWallet = (addr) => {
    addTrackedAddress(addr);
    saveActiveAddress(addr);
    refreshTracked();
    setActiveAddress(addr);
    setShowWalletList(false);
  };

  // Select a wallet from the list → view its (freshly scanned) score.
  // Selecting the same wallet that's already active still triggers a fresh
  // scan, since the activeAddress effect won't fire for an unchanged value.
  const onSelectWallet = (addr) => {
    const same = activeAddress && activeAddress.toLowerCase() === addr.toLowerCase();
    saveActiveAddress(addr);
    setActiveAddress(addr);
    setShowWalletList(false);
    if (same) run(addr, apiKey);
  };

  // Remove a wallet from the list. History is kept (re-adding restores it).
  const onRemoveWallet = (addr) => {
    removeTrackedAddress(addr);
    refreshTracked();
    if (activeAddress && activeAddress.toLowerCase() === addr.toLowerCase()) {
      clearActiveAddress();
      setActiveAddress(null);
      setResult(null);
      setFindings([]);
      setApprovalBreakdown([]);
      setLimitedDepth(false);
      setScoreHistory([]);
    }
    // If multiple wallets remain, stay on the list; the screen derivation
    // falls back to 'entry' (0 left) or 'score' (1 left) automatically.
  };

  // "Change" button: switch to the list when tracking multiple wallets, or
  // clear-and-restart when tracking exactly one.
  const onChange = () => {
    if (tracked.length > 1) {
      setShowWalletList(true);
    } else {
      const gone = activeAddress || tracked[0]?.address;
      if (gone) removeTrackedAddress(gone);
      clearActiveAddress();
      refreshTracked();
      setActiveAddress(null);
      setShowWalletList(false);
      setResult(null);
      setFindings([]);
      setApprovalBreakdown([]);
      setLimitedDepth(false);
      setScoreHistory([]);
    }
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
    setApprovalBreakdown([]);
  };

  return (
    <div className="min-h-screen bg-[#fafafa] text-neutral-900">
      <div className="mx-auto max-w-2xl px-5 pb-24">
        <div className="no-print">
          <Header
            address={activeAddress}
            apiKey={apiKey}
            onKeyChange={onKeyChange}
            onShowWallets={() => setShowWalletList(true)}
          />
        </div>

        <div className="no-print mt-2 mb-6">
          <LiveMetricsBanner totalAudits={totalAudits} available={available} />
        </div>

        {screen === 'entry' && (
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
              <AddressEntry onSubmit={onAddWallet} />
            </div>
          </>
        )}

        {screen === 'list' && (
          <WalletList
            tracked={tracked}
            onSelect={onSelectWallet}
            onRemove={onRemoveWallet}
            onAdd={onAddWallet}
            onImported={refreshTracked}
          />
        )}

        {screen === 'score' && (
          <>
            <div className="no-print mb-4 flex items-center justify-between">
              <span className="font-mono text-xs text-neutral-500">
                {activeAddress ? shortAddr(activeAddress) : ''}
              </span>
              <button
                onClick={onChange}
                className="text-xs font-medium text-neutral-500 hover:text-neutral-900 transition-colors"
              >
                {tracked.length > 1 ? 'Switch wallet' : 'Change wallet'}
              </button>
            </div>

            {(!activeAddress || loading) && (
              <div className="rounded-xl border border-[#e5e5e7] bg-[#f5f5f6] px-6 py-20 text-center">
                <div className="mx-auto w-6 h-6 border-2 border-neutral-200 border-t-neutral-800 rounded-full animate-spin" />
                <p className="mt-4 text-sm text-neutral-500">
                  {apiKey
                    ? 'Reading lifetime approvals and blocklist data from Etherscan…'
                    : 'Reading approvals and blocklist data (limited ~55-day window)…'}
                </p>
              </div>
            )}

            {activeAddress && !loading && result && (
              <>
                <div className="no-print mb-3 flex justify-end gap-2">
                  <CopyMarkdownButton address={activeAddress} result={result} findings={findings} />
                  <button
                    onClick={saveReport}
                    className="inline-flex items-center gap-1.5 rounded-lg border border-neutral-200 bg-white px-3 py-1.5 text-xs font-medium text-neutral-700 hover:bg-neutral-50 transition-colors"
                  >
                    Save report
                  </button>
                </div>
                <ScoreCard score={result.overall} lastCheck={lastCheck} />
                <div className="no-print mt-3">
                  <ScoreHistory history={scoreHistory} />
                </div>
                {limitedDepth && (
                  <div className="mt-3 rounded-lg border border-amber-200 bg-amber-50 px-4 py-2.5 text-xs text-amber-800">
                    Limited scan depth (~55 days) — Approval staleness and Historical contract exposure
                    are based on the last ~55 days only. Add an Etherscan API key for full lifetime
                    history.
                  </div>
                )}
                <div className="no-print mt-4">
                  <TipJar />
                </div>
                <SubScoreGrid subs={result.subs} />
                <WhatChanged findings={findings} />
                <ApprovalBreakdown breakdown={approvalBreakdown} owner={activeAddress} />
                <PercentileLine score={result.overall} />
                <div className="print-only" aria-hidden="true">
                  <div ref={tsRef} className="mt-8 border-t border-neutral-300 pt-3 text-xs text-neutral-500">
                    Report generated: —
                  </div>
                  <p className="mt-2 text-xs text-neutral-500">
                    Generated by WalletXS — a high score is not a safety guarantee. See the
                    Methodology page for details.
                  </p>
                </div>
              </>
            )}
          </>
        )}

        <Footer />
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
import Footer from '@/components/walletxs/Footer';

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

        <p className="mt-2 text-sm text-neutral-500">
          For a plain-language privacy policy, see{' '}
          <Link to="/privacy" className="text-neutral-800 underline underline-offset-2 hover:text-neutral-900">
            here
          </Link>
          .
        </p>

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
            Contract addresses are also checked against the GoPlus Security Malicious Address API,
            a second, independent malicious-address source. For each contract address already being
            checked against ScamSniffer (both active approval spenders and every contract the
            wallet has interacted with), the app asks GoPlus whether it considers that address
            malicious. An address counts as flagged if EITHER source flags it, so the two catch
            more than either could alone. If one source is unreachable, the check still runs on
            whichever is available — it is only marked "Unavailable" when both fail. GoPlus's core
            endpoints work without a key; an optional developer key raises rate limits. The request
            sends only the contract address being checked, never your wallet address, and is
            subject to GoPlus's own privacy practices, not this app's.
          </p>
          <p>
            The wallet address is also screened against the OFAC SDN sanctions list (Ethereum
            addresses), extracted and auto-updated nightly from the official U.S. Treasury list by
            the open-source 0xB10C/ofac-sanctioned-digital-currency-addresses project. The entire
            list is downloaded and matched in your browser — your address never leaves your device
            for this check.
          </p>
          <p>
            You can track more than one wallet at a time. Each tracked wallet is only scanned when
            you actively select it to view. Tracking multiple wallets does not cause them to be
            checked automatically or in the background — nothing is queried for a wallet you
            haven't opened.
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
            {SAMPLE_SIZE.toLocaleString()} public wallets, refreshed on a monthly schedule by
            re-scoring a fresh batch of addresses. The current sample was last refreshed{' '}
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
            provider (in the limited-depth fallback), and requests are made to the ScamSniffer
            blocklist host and to the GoPlus Security API in order to run a check. For GoPlus, only
            the contract address being checked is sent — never your wallet address. Those requests
            are subject to those third parties' own privacy practices, not ours. We have no server
            and collect nothing.
          </p>
          <p>
            Each scan also sends one anonymous increment ping to the audit-counter endpoint
            (api.walletxs.com/count) so the header can show a live "audits run" total. The ping
            carries no wallet address, no user id, and no identifying data — only an empty counter
            increment. If it is blocked (ad-blocker or offline) the counter simply shows its
            fallback value.
          </p>
          <p>
            Exporting your tracked wallet list downloads a JSON file to your own device — nothing
            is transmitted anywhere. Importing reads a JSON file you provide back into local
            storage. Neither operation involves any server or third party.
          </p>
          <p>
            The in-app support link takes you to Ko-fi, an external third-party platform with its
            own account system and privacy practices, separate from WalletXS. WalletXS does not
            receive any information you provide to Ko-fi (payment details, email, name, etc.) —
            that relationship is entirely between you and Ko-fi.
          </p>
        </Section>

        <p className="mt-10 text-sm text-neutral-500">
          <Link to="/privacy" className="text-neutral-800 underline underline-offset-2 hover:text-neutral-900">
            Privacy Policy
          </Link>
        </p>

        <Footer />
      </div>
    </div>
  );
}
```

## src/pages/PrivacyPolicy.jsx

```jsx
import React from 'react';
import { Link } from 'react-router-dom';
import { ArrowLeft } from 'lucide-react';
import Footer from '@/components/walletxs/Footer';

const Section = ({ title, children }) => (
  <section className="mt-8">
    <h2 className="text-sm font-semibold uppercase tracking-[0.14em] text-neutral-500">{title}</h2>
    <div className="mt-2 space-y-3 text-sm leading-relaxed text-neutral-800">{children}</div>
  </section>
);

export default function PrivacyPolicy() {
  return (
    <div className="min-h-screen bg-[#fafafa]">
      <div className="mx-auto max-w-2xl px-5 pb-24">
        <Link
          to="/"
          className="inline-flex items-center gap-1.5 py-6 text-sm text-neutral-500 hover:text-neutral-900 transition-colors"
        >
          <ArrowLeft className="w-4 h-4" strokeWidth={1.75} /> Back
        </Link>

        <p className="text-xs text-neutral-400">Last updated: August 8, 2026</p>
        <h1 className="mt-1 text-2xl font-semibold tracking-tight text-neutral-900">Privacy Policy</h1>

        <Section title="No accounts, no personal data">
          <p>
            WalletXS has no accounts and no registration. We never ask for your name, email, phone
            number, or any other personal information. We do not run a database, and we do not
            collect personal data — ever.
          </p>
        </Section>

        <Section title="Your wallet addresses stay in your browser">
          <p>
            The wallet addresses you check are processed entirely in your browser. They are never
            stored on any server controlled by WalletXS, and we have no server that records which
            addresses are checked.
          </p>
          <p>
            You can track more than one wallet at a time. Each tracked wallet is only scanned when
            you actively select it to view. Tracking multiple wallets does not cause them to be
            checked automatically or in the background — nothing is queried for a wallet you
            haven't opened.
          </p>
          <p>
            To run a scan, your browser makes requests to third-party services (described below).
            Those requests are subject to those services' own privacy practices, not ours. WalletXS
            itself does not log or store your addresses.
          </p>
        </Section>

        <Section title="Third-party requests made to run a scan">
          <p>
            Each time you run a check, your browser contacts some or all of the following public
            services. What they do with request data is governed by their own policies, not by
            WalletXS:
          </p>
          <ul className="list-disc pl-5 space-y-1.5">
            <li>
              A public Ethereum RPC provider — used to read on-chain approval logs when no Etherscan
              key is available (a limited-depth scan covering roughly the last 55 days).
            </li>
            <li>
              Etherscan — used to read your wallet's full lifetime approval history when an API key
              is present (either the optional developer key bundled with the app, or your own free
              key if you provide one).
            </li>
            <li>
              The ScamSniffer open scam-address blocklist — downloaded from its public GitHub
              repository so contract addresses can be checked against it.
            </li>
            <li>
              The GoPlus Security Malicious Address API — queried for each contract address already
              being checked against the ScamSniffer blocklist (active approval spenders and
              interacted contracts). Only the contract address being checked is sent, never your
              wallet address. Subject to GoPlus's own privacy practices, not WalletXS's.
            </li>
            <li>
              The OFAC SDN sanctions list (Ethereum addresses) — downloaded in full and matched
              against your address locally, in your browser. Your address is not sent to this
              service.
            </li>
          </ul>
        </Section>

        <Section title="What is stored on your own device">
          <p>
            WalletXS stores a small amount of data in your browser's local storage, on your own
            device only. This never leaves your device. It includes:
          </p>
          <ul className="list-disc pl-5 space-y-1.5">
            <li>The list of wallet addresses you are tracking, so the app can re-check them when you return.</li>
            <li>Your recent score history for each tracked wallet (scores and findings), used to show trends.</li>
            <li>
              Your optional Etherscan API key, if you choose to provide one, so full-depth scans
              persist across visits.
            </li>
          </ul>
          <p>
            You can export your tracked wallet list as a JSON file that downloads to your own
            device — nothing is transmitted anywhere. You can import a previously exported JSON
            file back into local storage. Neither operation involves any server or third party.
          </p>
          <p>
            You can clear all of this at any time by using your browser's site-data controls, or by
            removing wallets from the tracked list in the app.
          </p>
        </Section>

        <Section title="The anonymous usage counter">
          <p>
            WalletXS shows a live "audits run" total. Each completed scan sends a single anonymous
            increment to a counter endpoint. This ping contains no wallet address, no user
            identifier, no IP address, and no other identifying data — only an empty counter
            increment. It carries no query parameters and no cookies.
          </p>
          <p>
            If the ping is blocked (for example by an ad-blocker or because you are offline), the
            counter simply shows its fallback value. The counter does not track you across visits
            and cannot be used to identify you.
          </p>
        </Section>

        <Section title="The Ko-fi support link">
          <p>
            The in-app support link takes you to Ko-fi, an external third-party platform with its
            own account system and privacy practices, separate from WalletXS. WalletXS does not
            receive any information you provide to Ko-fi (payment details, email, name, etc.) —
            that relationship is entirely between you and Ko-fi.
          </p>
        </Section>

        <Section title="No tracking, no analytics, no cookies">
          <p>
            WalletXS does not use advertising trackers, does not set tracking cookies, and does not
            run third-party analytics. We do not run databases, store cookies, or log IP addresses
            or wallet queries. There is no backend that ties your activity to your identity.
          </p>
        </Section>

        <Section title="A high score is not a safety guarantee">
          <p>
            Privacy practices aside, please remember that a high WalletXS score only means nothing
            known-bad was found in the data checked. It is not a safety guarantee. See the{' '}
            <Link
              to="/methodology"
              className="text-neutral-800 underline underline-offset-2 hover:text-neutral-900"
            >
              Methodology
            </Link>{' '}
            page for details on what is and is not checked.
          </p>
        </Section>

        <p className="mt-10 text-sm text-neutral-500">
          <Link
            to="/methodology"
            className="text-neutral-800 underline underline-offset-2 hover:text-neutral-900"
          >
            Methodology
          </Link>
        </p>

        <Footer />
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
