# PR description style: examples

Curated from smartcontractkit/explorer PR history (Jun 2025 to Oct 2026): 164 PRs by
marek-sadura, 275 by a-colleague. Bodies are quoted or lightly trimmed (typos fixed, dashes
normalised). Each example says what to copy from it. Copy the shape, not the words.

**Who brings what:**

| | Marek | a-colleague | Target (the mix) |
|---|---|---|---|
| What | 1-4 short bullets, often one line | verb-first bullets, nested sub-bullets for detail | Marek's length, a-colleague's verb-first + nesting |
| Why | often missing or "we want X" | the actual cause or constraint, nested | always the cause, one level of nesting max |
| Testing | real tx hashes / message links when present, often missing | always present: Visit / Do / Confirm, with addresses and links | always present, Visit / Do / Confirm with concrete data |
| Notes | only when it matters | merge order, follow-ups, diff-size breakdown | only what a reviewer must know before approving |

## Marek: copy these

### Tiny fix: three lines that say everything (#9780, +7/-0)

```markdown
## What

- Handle ccip 2.0 executor failed status

## Why

- It shows 404 right now

## Testing

- Open https://<preview-url>/#/side-drawer/msg/0x129a…4b4b, confirm it shows the executor failed status instead of 404
```

Copy: the Why is the user-visible symptom, the Testing is one link plus what you should see.
A 7-line diff gets a 3-bullet description.

### Small feature: the reason is the interesting bit (#11433, +141/-1)

```markdown
## What

- Adds a data-driven testnet lane e2e spec `form-laneFeesAndRejectSend-testnet.test.ts`, covering Hedera Testnet ↔ Sepolia with CCIP-BnM
- Hedera Testnet → Sepolia asserts the fee estimate, then triggers a send and rejects it in the wallet; Sepolia → Hedera Testnet asserts the fee estimate only

## Why

- The Hedera testnet lane had no e2e coverage
- The fee assertion is bounded (USD > 0 and < 50) so an HBAR 8-vs-18 decimals regression (fee off by ~10^10) fails the test instead of slipping past a bare > 0 check

## Testing

- `BRIDGE_UI_URL=<preview-url> pnpm e2e-local form-laneFeesAndRejectSend` from `apps/bridge-e2e`
- Hedera → Sepolia send needs the e2e wallet funded with HBAR + CCIP-BnM on Hedera testnet (topped up)

## Notes

- Covering a new lane is one `FormConfig` constant plus one entry in `FEE_LANES` or `SEND_LANES`
```

Copy: the second Why bullet. It explains the one non-obvious decision (the upper bound) by
naming the exact regression it catches. Testing is a copy-paste command plus the precondition.

### Bug fix: cause in one bullet each, reviewer steps numbered (#10921, +42/-5)

```markdown
## What

- Fix Solflare connection on Transporter: connected, but send failed with "Invalid account"

## Why

- The Solana adapter passed a cached `WalletAccount` to sign/send. Solflare rejects a stale reference (Phantom is lenient). Now resolved from live `wallet.accounts` before signing
- That surfaced a race: `checkConnection()` fired a silent `connect()` during sign/send and Phantom dropped the in-flight request. Silent reconnect now only runs when there are no accounts yet (session restore)

## Testing

- No automated coverage for wallet extension code, verified manually on devnet
- Solana → Base, e.g. zBTC:
  1. Connect Solflare, bridge, approve. Confirm no "Invalid account"
  2. Connect Phantom, bridge. Confirm the approval popup appears and it completes
  3. Refresh while connected. Confirm the wallet auto-reconnects
```

Copy: each Why bullet is cause → fix in two sentences. Testing covers the fix, the regression
risk (Phantom) and the preserved path (reload), and says honestly why there is no unit test.

### Data change: the test data IS the testing section (#9193)

```markdown
## Testing

ccip 2.0 messages to test:
- 0xf0d66bd82e945247a22e08ba5fc9ad9eecf028b5abbea095eff62b8e9c8e9e24
- 0xc25181dd7b564ba1172f50d01bbe0ddbb62470edf27484c61a20e1c94919a261

## Notes

- API does not return executor details yet, so only the address is shown. Separate ticket for details.
```

Copy: when the UI renders data, give the reviewer the exact records (tx hash, message id,
address) that exercise it. A note that pre-empts "why is X missing?" saves a review round.

## Marek: avoid these

| Seen in | Problem | Instead |
|---|---|---|
| #10905, #10500, #9375, #8531 and ~20 more | Template placeholders left in ("Why is this change needed?") | Fill the section or delete it. Never ship placeholder text |
| #10961, #10960, #10957 | Testing: "Everything should work the same way" | Name the flow and the comparison: "Search a tx hash, a message id and an address on preview, confirm same results as prod" |
| #10927, #9689 | Testing: "Manual exec should just work" | Give one message that is ready for manual exec and the expected result |
| #10013, #9136, #8012 | Body is only the Jira link | At least one What bullet and one Why bullet |
| #11770, #11318, #11435 | 40+ lines, design-doc Notes, em dashes, every decision justified | Keep what a reviewer needs to approve. Design rationale goes in a doc or code comment, linked |

## a-colleague: copy these

### Small fix, full rigor (#11752, +67/-18)

```markdown
## What

- Safely parses the stored active transfer using zod
- Falls back to no active transfer if the data is invalid

## Why

- The stored `activeTransfers` value was parsed with `JSON.parse`, which throws on corrupted data
- An uncaught error crashes the entire app (all the way to `FullScreenErrorBoundary`)
- Any breaking change to the stored transfer shape risked the same crash, nothing validated it

## Testing

- Visit the preview deployment
- Initiate a token transfer
  - Confirm you land on the active transfer page
- Dev tools → Application → Local storage → mangle `activeTransfers` (e.g. rename `sender` to `origin`)
- Reload the page
  - Confirm the app doesn't crash
  - Confirm it renders `No pending transfers`
```

Copy: the Testing section. It tells the reviewer how to reach the broken state by hand
(mangle local storage), then two `Confirm` lines that each check one thing.

### Dead code: two words of Why is enough (#11754, +0/-390)

```markdown
## What

- Cleans up (almost) all of the legacy code left on the Transporter SDK
  - Only `logResponseHMAC` stays, it is still in use
  - A follow-up PR moves it into `bridge-ui` and removes the rest

## Why

- Dead code

## Testing

- `typecheck` confirms none of these were in use
```

Copy: Why is allowed to be two words when it is true. Testing names the check that proves it.

### Refactor: real test data and a size breakdown (#11787, +602/-817)

```markdown
## What

- Transporter uses the `@chainlink/wallet-screening` package
- Drops the legacy remnants from `transporter-sdk`

## Why

- Simplify the screening route
- Clean up the Transporter SDK

## Testing

- Visit the preview deployment
- Connect a forbidden wallet (impersonator: `0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045`)
  - Confirm the banner renders: `Sender wallet address is forbidden`
- Connect an authorized wallet (e2e wallet: `0x7b13921946c29AAfF64b0f1B5932B4b6738848b2`)
  - Confirm the banner doesn't render

## Notes

- Diff size: lockfile +546 -559, legacy removal -124, rest +56 -134
```

Copy: both the positive and negative case, each with the exact address. The Notes line turns
a scary +602/-817 into "the real change is ~190 lines".

### Not testable: say so, say why, say how you checked (#11874, #11793)

```markdown
## Testing

- Checkout locally, run e2e
- Confirm `ManualExec.test.ts` > `Manual Exec > TON > EVM -> TON` succeeds

## Notes

- The extension is still named Tonkeeper, so nothing else was renamed
- The test is skipped in CI (known broken). Follow-up PR unskips it once this merges
```

When a path only runs on a thrown error (#11793), a-colleague writes the temporary local
hack to trigger it ("throw from a page component, confirm the fallback renders, undo") instead
of skipping the section.

## a-colleague: where to trim

| Seen in | Too much | Trim to |
|---|---|---|
| #11702 (+6/-2) | Why + Problem scenario + Root cause + Solution sections for a one-line swap | 2 Why bullets: what failed (`===` on differently-cased addresses → approval too small → revert) and that `isSameAddress` is already used elsewhere |
| #11884 | 12 preview links in Testing | 3 links: one that must work, one that must fail, one legacy format |
| #10515 | Cost model with per-month math in the PR body | One line with before/after, link the doc |

## The mix: rewrites in the target style

### Rewrite of a verbose PR (#11817 original: 30 lines, 4 em dashes)

```markdown
## What

- `isTokenSupportedByPool` supports Aptos (ccip-contracts feature + ccip-data dispatcher)
- `applyChainUpdates` remote address encoding is family-agnostic via `encodeAddressToAny`, in all four versions
  - Was "EVM address, else base58"
- Unimplemented chain families return `501`; the form skips the on-chain check instead of showing "Invalid pool address"
- De-dupes `validateRemotePoolAddresses` (two identical copies)

## Why

- Adding an Aptos remote failed with "Invalid pool address". Two causes:
  - `isTokenSupportedByPool` threw `Unsupported chain` for Aptos and the form reported it as bad input
  - Calldata generation ran `bs58.decode()` on a `0x` 32-byte address

## Testing

- Unit: encoder padding per family, all four `applyChainUpdates` versions, Aptos pool read, 501 skip
- Encoder output matches plasma-mainnet tx [`0x9a96…17d0`](https://plasmascan.to/tx/0x9a962b65404862317230fd0a88fa161f3de11acc8c7f681a6581fbbd920017d0) byte for byte; [Tenderly](https://tdly.co/shared/simulation/34e7dc97-4497-4b9b-a1b1-3e8bce6fccfe) at the block before it succeeds
- Manual: Pool Owner → Apply Chain Updates (TMEM link in ticket), add Aptos, paste the ticket's token + pool
  - Confirm the pool address field validates

## Notes

- No e2e: needs a deployed aptos-testnet token + pool, none exists
- TON 36-byte `workchain:hash` still throws in `encodeAddressToHex32`, unchanged from `develop`

[JIRA](https://<jira-host>/browse/DAPP-11388)
```

What changed: em dashes gone; the "field-level validation was never at fault" paragraph and
test counts dropped (the diff shows it); the byte-for-byte proof and the "no e2e, here's why"
kept, because a reviewer can't get those from the code.

### Rewrite of a thin PR (#10961 original: "Everything should work the same way")

```markdown
## What

- Homepage search uses ccip-api instead of the Atlas endpoint

## Why

- ccip-api is the source of truth; Atlas is being sunset

## Testing

- On preview, search a tx hash, a message id and an address
  - Confirm each lands on the same page as on prod

[JIRA](https://<jira-host>/browse/DAPP-7273)
```

What changed: the Why names the reason, Testing names three inputs and what "the same" means.
Still 9 lines.
