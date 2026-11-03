# Marek's voice: samples

Hand-written Slack messages, 2026, typos kept. Then generated text that went out under his
name, with what gives it away. Then two before/after pairs. Copy the shape, not the words.

## Hand-written

### Bug report to another team (#ccip-public-api-internal, Oct 27)

> hey! I would like to report a bug which is affecting canton integration in transporter.
> /messages endpoint does not work with canton tx hash search :
> ```For 'sourceTransactionHash': Value must follow pattern ^(0x[a-fA-F0-9]{1,64}|[1-9A-HJ-NP-Za-km-z]{32,88}|[a-fA-F0-9]{64})$.```
>
> canton tx hash is i.e. 12208bfdaae625f397683e8904fadfbc69b90a959ec0833440b12d5b49fd7165856a
>
> example canton->evm message: https://ccip.chain.link/msg/0xbb144ce9...

Point, error verbatim, the input, a live example. No explanation of what the regex does.

### Bigger problem for another team (#ccip-public-api-internal, June 3)

> hey! I started reworking explorer to use ccip-api /messages endpoint for list views - /tx page, /address transactions, "recent" transactions on home page. Unfortunately seems there is misalignment between designs and api. There is also a bug realted to using filters + cursor - if it is desired then I don't know how pagination is going to work.
>
> Issue 1) Design vs api:
> In designs for /tx page we have pagination showing x pages where user can select one of them. With curor based pagination, we don't have a count of pages at all. All we can do is kind of "infinite scroll" behavior to load more entries.
>
> Issue 2) Potential api bug. It is not possible to use filter + cursor. Api returns "Cannot combine filter parameters with pagination cursor. ..."
> If it is intended how are we supposed to build pagination based on filtered query? Example use case - we have /address page where we need to filter transactions by either source or dest address.
>
> cc @api-lead @api-pm @api-dev
>
> example curls:
> ```# 1) Filter alone - 200. Returns hasNextPage:true + a cursor.
> curl -s "https://api.<staging>/v2/messages?sourceChainSelector=5009297550715157269&limit=4"
> # 3) Filter + REAL cursor -> 400 INVALID_PARAMETER_COMBINATION (core blocker).
> curl -s "...?sourceChainSelector=5009297550715157269&cursor=GSmm5..."
> # 5) FILTERED query's cursor drops the filter on page 2.
> curl -s "...?cursor=83iAW...&limit=4"```

Two numbered issues, each: what was expected, what happened, the question. Curls at the end
so the other team can reproduce without asking. This is the long end of his range.

### Pushback on a PR, with an alternative (#team-dapp, Apr 30)

> I reviewed the pr and honestly it brings a lot of code debt spread across packages (lets call it temp state) and finally uses public rpc to send transactions on the client.
> If we are sure that:
> a) there will be immediate follow up
> b) this code will not become a blocker for potential sdk updates focused around ccip 2.0 release which soon will become top priority
> We can accept that, although public solana rpcs were giving us a bit of unpredictable results in the past.
>
> Given we need public rpc anyway I can also propose a different workaround - I think it will be more focused and easier to replace.
> [quotes the two sentences from the PR he is responding to]
> How about for solana implementation, we just use execute() - build everything on the client (we will need some kind of signer adapter as far as I remember but should be realatively low lift).
> Main reason we didnt want it, was the need to user public rpc to build tx, but with current approach it is needed anyway. IMHO it will be much cleaner
>
> cc @teammate @pm

Verdict first, the conditions as `a) b)`, then the alternative with its one reason.

### Clarifying scope with another team (#transporter-canton, Oct 28)

> From what I see in the ccip-sdk, a party's first send from Canton runs 2 setup steps before the transfer itself, each as its own tx:
> 1. Create the PerPartyRouter
> 2. Create the CCIPSender
>
> In my tests via cli I only faced PPR deployment, I am assuming our accounts already have CCIPSender?
>
> The question is - is it going to stay as 2 step flow? It would mean user 1st send from canton will require 3 txs (ppr, ccipsender, final transfer).

and, after their answer:

> So I will aim for:
> a) send: PPR + CCIPSender checks + respective deployments (current flow)
> b) receive: PPR+ CCIPREceiver check + respective deployments (current flow)
> c) prepare the code to plug in "global sender/receiver" check as 1st step so we can skip CCIPSender/CCIPREceiver checks if it is available
>
> do I get it right?

Observation, the assumption as a question, the real question with its consequence. The
follow-up is the plan as `a) b) c)` plus one line asking for confirmation.

### SDK issue, one paragraph (#transporter-canton, Oct 23)

> also faced issue with provider :smile: can give you more details on mon but in general fromProvider breaks on real wallets: they report operator-defined network ids like canton:testnet-cv1, the SDK only maps exact canton:testnet, and cantonConfig.chainId, which could override that, is never read. Please honor chainId, or match on the canton:testnet* prefix.

What breaks, on which input, and the two acceptable fixes. One paragraph.

### Finding shared with the team (#team-dapps-only, Aug 14)

> Interesting bug / finding / product issue.
>
> When you try to trigger transaction with MM for a custom chain not in you MM - you will see a popup in MM asking you to add the chain. BUT it works only if chan native token decimals are 18. Hedera is 8 decimals so MM wont offer it and bridge jsut shows error. Sounds like edge case probably not worth tackling (because in norma case, if you want to bridge from i..e hedera I would assume you have it in your wallet), but it became troublesome in e2e

### Standup (#team-dapps-only, Oct 30)

> I will wrap up before the standup today so leaving my updates:
> Doing progress on canton flow - I have prs for both send and receive multi-tx flows for canton (with no regressions on other chains):
> https://github.com/smartcontractkit/explorer/pull/12555
> https://github.com/smartcontractkit/explorer/pull/12574

### To a teammate who has the context

> pr rebased

> squeezed prs into one lf retick: https://github.com/smartcontractkit/explorer/pull/12211

> pr udpated, jsut added check, nothing removed. Anyway good observation for the future - avoid unnecessary refactors right away

> would it mean additional transaction for the user? We should probably handle it, but it does not feel like core requirement. Anyway more of a product decistion @canton-pm @pm

## Generated text that went out as his

### Repo decision post (#team-dapps-only, July 29, excerpt)

> It works end to end — but having gone through it, imho a separate repo is the better call.
>
> What in-explorer gives us: workspace consumption while iterating (no publish→bump loop), the existing changesets/publish pipeline, shared lint/CI standards.
>
> What the scaffolding taught me (case for separate):
> - the leak risk is only held back by tooling. The private/public boundary is scripts, not structure: eslint allowlist, tarball-install check in CI, build steps. Review caught sourcemaps shipping full private source into the public tarball — guardrails worked, but in a separate repo that mistake class can't exist.
> [...]
> My take: once you rule out direct reuse of shared packages as too fragile, explorer's remaining edge is basically shared lint/tooling — not much. The public audience is the point; a separate repo serves it better. Keen to hear where you land before we sink the real API into either.

Tells: long dashes, `What X gives us:` labels, every point stated then restated as a
conclusion, `My take:`, `Keen to hear`. His version of the same post is the Point ("imho a
separate repo", with the sourcemap leak as the one reason), `a) b)` for the two costs, and
"wdyt?".

### API feature request (group DM, Mar 11, excerpt)

> Needed fields
>
> 1. sourceFinalizedTimestamp — when source chain reached finality
>    → ETA for "Commitment" step
> [...]
> Impact
> Not a blocker — we can ship without these. The stepper works correctly but 2 of 3 "time remaining" indicators won't show, which is a UX degradation vs current version.
>
> Proposed addition
> ```{ "sourceFinalizedTimestamp": "...", "blessedTimestamp": "..." }```

Tells: section labels in a chat message, long dashes, "which is a UX degradation vs current
version" restating "won't show". The JSON block is the one part worth keeping.

## Before / after

### Slack bug report

Before (generated from the facts):

> Hey team :wave: We hit a blocker on the Canton integration in Transporter.
>
> We use `GET /v2/messages` to look up a transfer by its source tx hash. Canton tx hashes are 68 hex chars with no `0x` prefix, e.g.: `12208bfd...`
>
> The endpoint rejects it with: [error]
> None of the three branches matches: it isn't `0x`-prefixed, it isn't base58, and it's 68 chars, not 64.
>
> The messages themselves are already in the API, e.g. this Canton -> EVM one: [link]. The hash validation is the only thing blocking the lookup.
>
> Impact: after a user sends from Canton, Transporter can't show the pending transfer.
>
> Could you update the `sourceTransactionHash` validation to accept Canton hashes? Staging first, please, so we can verify on our side before it goes to prod.
>
> Also, who on the API team owns Canton issues? I'd like to know who to tag next time. Thanks! :pray:

After:

> hey! /messages rejects canton tx hashes, which blocks transporter from showing the pending transfer after a send from canton:
> ```For 'sourceTransactionHash': Value must follow pattern ^(0x[a-fA-F0-9]{1,64}|[1-9A-HJ-NP-Za-km-z]{32,88}|[a-fA-F0-9]{64})$.```
> canton tx hash is i.e. 12208bfdaae625f397683e8904fadfbc69b90a959ec0833440b12d5b49fd7165856a
> example canton->evm message: https://ccip.chain.link/msg/0xbb144ce9739b7c9137e3d1edd04e6ec8c68ff6fbee418be0c947e0f06840c745
>
> can we get it on staging first? also who should I ping with canton issues on api side?

Removed: the greeting, the regex explained to its owners, "the messages are already in the
API" (the link shows it), the `Impact:` label, the thanks.

### PRD section

Before (generated from the facts, excerpt):

> ### Problem
>
> Today the upstream ccip-sdk creates the CCIPReceiver inside execute without telling the user, and it never creates the PPR. So a first-time receiver:
> - can be asked for up to 3 signatures with no explanation of what they are for, and
> - has their Execute fail when the PPR is missing.
>
> ### Proposed behaviour
>
> 1. The user clicks **Execute**.
> 2. Transporter checks which setup steps are missing for the connected party.
> 3. **If nothing is missing:** the execute transaction runs in a one-step modal.
> 4. **If anything is missing:** a setup modal takes the user through each missing step, one transaction at a time. It ends on an **"Account ready"** state. No execute transaction runs in this flow.
> 5. The user clicks **Execute** again. Setup is now complete, so the execute transaction runs in the one-step modal.
>
> The send side already works this way (DAPP-12011), so users get the same setup experience in both directions.
> [...]
> **Recommendation: keep the second click.** If Execute ran automatically, a first-time user would sign three transactions in one modal. Ending setup on "Account ready" gives a clear stop, and the user then starts the transfer themselves.

After:

> ## Canton receive-side account setup before Execute
>
> Before a Canton party's first Execute it needs a PerPartyRouter and a CCIPReceiver, one tx each. Upstream ccip-sdk creates the CCIPReceiver silently inside execute and never creates the PPR, so a new user signs up to 3 txs with no explanation and the PPR case fails.
>
> Proposed: Execute checks which steps are missing for the connected party.
> a) something missing: modal runs them one tx at a time, ends with "Account ready", nothing executed. User clicks Execute again
> b) nothing missing: execute runs in a one-step modal
>
> Same pattern as send side (DAPP-12011).
>
> Global shared receiver (Canton team, config flag): CCIPReceiver step disappears from the check, no UI change.
>
> Out of scope: real Canton calls (stubbed until DAPP-11855), party vs receiver match, participant DAR check.
>
> Open question: should Execute run automatically after setup? I would keep the second click, setup modal already has up to 3 signatures.

Removed: bold labels, the "same experience in both directions" sentence, the second reason
for the recommendation. Problem and proposal are one paragraph each.
