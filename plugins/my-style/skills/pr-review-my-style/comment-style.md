# Marek's PR comment style

Curated from ~160 written GitHub comments (Sep 2025 – Sep 2026, smartcontractkit/explorer).
Typos lightly cleaned, wording otherwise verbatim. Use these as the target shape, not as
templates to fill in.

## The shape of a comment

A comment is one of five things. Pick one, write it, stop.

| Kind | Shape | Length |
|---|---|---|
| Bug | what breaks + when + the fix (or the one-liner) | 1–3 sentences, optional code block |
| Question | a real question the author can answer with one sentence | 1 sentence, plus an optional `if yes, X breaks` clause |
| Suggestion | the alternative, and where it already exists (path) if it does | 1–2 sentences |
| Nit | `nit:` prefix + the thing + "not a big deal" if it isn't | 1 sentence |
| Scope flag | `not in scope but` / `not a blocker but` / `Side question:` + the concern | 1–3 sentences |

Rules that hold across all five:

- Start with the point. No "Great work here, but", no restating what the code does.
- Lowercase start and casual tone are fine. `imho`, `tbh`, `I would`, `probably` are the hedges.
- Concrete over general: name the value, the input, the chain, the line that fails.
- Code goes in backticks; a fix that fits on 1–5 lines goes in a fenced block with no prose around it.
- Point to existing code by repo path when suggesting reuse.
- Bigger concerns get a numbered list `1) 2) 3)`, max 3 points, and end with whether it blocks.
- Assume the author knows the codebase. Never explain what a function does before critiquing it.
- One emoji or `:D` per review at most, never on a bug comment.
- Never use long dashes (— / –) in comments.

## Length contract

| Kind | Sentences | What each sentence carries |
|---|---|---|
| Bug | 1–3 | (1) what breaks and on which input/chain/state, (2) the fix or the one-liner, (3) optional: how bad / whether it blocks |
| Question | 1 | the question |
| Suggestion | 1–2 | (1) the alternative, (2) the path where it already exists |
| Nit | 1 | `nit:` + the thing |
| Scope flag | 1–3 | (1) the marker (`not a blocker but`), (2) the concern, (3) what to do about it |

Praise is its own separate two-word comment ("this looks 🔥", "oh nice idea with expected fee in config"), never an opener.
A test-coverage comment names the scenario (`e2e for approval + send in one flow`) or is not written.

## Examples by kind

### Bug — say what breaks, when, and the fix

> This wont work.
> 1) `[-1]` index will always return undefined - I believe you meant `.at(-1)`
> 2) Potential case when we have more than 1 tx - 1st one is approval, 2nd one is ccip send. Given that, ccip send will fail without approving first anyway. I know this case does not apply to native send, but in general it is safe to assume we should be using `transactions[0]` in this scenario.

> `formatUnits(1n, 18)` returns `"0.000000000000000001"` — a decimal string. BigInt cannot parse decimal strings and will throw. This should be just `1n` / `BigInt(1)`

> `new SafeApiKit` throws for chains missing from its built-in service list (on testnets only Sepolia and Base Sepolia are covered), and it sits outside the `try`. On those chains the deploy fails after the tx was already sent, even for EOA wallets.

> This catch can't tell a 404 from a 5xx or a not-yet-indexed proposal, so a real safeTxHash ends up in the receipt lookup and fails after ~3 min. Worth checking the status before falling back.

> No try/catch on this poll, a single transient error from the Safe service will fail a deploy that is still queued.

> Would also check `safeTx.isSuccessful === false` here, the outer Safe tx can succeed while the inner factory call reverted.

> there are 2 types of response, while this one handles only one? `type Response = { signature: string } | { transaction: string }`

> Lost null guard vs the old Prismic version (`if (!supportedNetworkInfo) return entries`). If Contentful has a token on a network not yet in supportedNetworks, this destructure throws and aborts the whole generate run. Cheap solution:
> ```
> const info = supportedNetworks[network]
> if (!info) return entries
> ```
> probably not a big deal since networks should be syncing automatically

> `useLaneFeatures()` returns a useQuery result where `data` is `{ minBlockConfirmations }`. The destructuring `const { data: minBlockConfirmations }` renames `data`, so the variable holds the whole object and `typeof ... === 'number'` is always false — fast transfers will always appear unsupported.

followed by a second, separate comment with only the fix:

> should be:
> ```
> const { data } = useLaneFeatures()
> return typeof data?.minBlockConfirmations === 'number' && data.minBlockConfirmations > 0
> ```

> Since this shares `tokenPoolConfig.ccipVersion` with the direct form, a version like 2.0.0 picked there survives the switch and passes validation, then `resolveFactory` throws. `z.enum(FACTORY_CCIP_VERSIONS)` would turn that into a field error.

### Question — one real question

> any reason to wrap string with resolve?

> what is ccipAddress / ccipObjectRef ?

> is it needed or leaving an object for future "optimization" XD

> it is only type casting change right? Looks like a lot of changes but it is just formatting?

> are these comments just for future awareness?

> it uses public rpc, is it some piece of important data?

> hmm shouldn't we have some value for native send which is amount + fee?

> any reason for this change? file says intentLanesEnabled but in fact it is lanes config - they don't need to be necessarily enabled. if it is needed as separate file - maybe intentLanesConfig ?

> Is the add-only behavior intentional? Edits to existing network metadata in libs/networks/src/** will push to code but not propagate to Contentful — is there a follow-up PR / manual process for updates?

> might be I'm missing some context, but this condition is unclear. why can I open it only when there are steps or can't open it when tx is complete / there is an error?

### Suggestion — the alternative, and where it already lives

> there is a hook with this logic:
> apps/bridge-ui/src/app/hooks/useFallbackImage.ts
>
> or probably the best would be to use one from blocks:
> libs/blocks/src/components/AssetIcons/useImageErrorFallback.ts

> this code is literally repeated on every page. I would recommend moving it to shared component which will take visibleTabs as param.

> we render this just for active tab, so I would rather pass TabComponent in props rather than passing whole formComponents

> this could probably use `const { network: chainSelectorName } = useNetwork()` directly in here?

> There is no need to disable this rule, you can compose component like:
> ```
> <form.Field ...props>
>   {(field) => (...children)}
> </form.Field>
> ```

> I think it is safe to skip this check - if chain is not supported then getLaneFeaturesBase would throw anyway

> there is no need to have ton and tonMainnet. we don't have such pattern, just ton is enough

> worth to update to 1.1.1 to avoid conflicts

> Pls avoid using ccip-tools-ts - import from @chainlink/ccip-sdk-external instead, or have centralized reexport of CCIPVersion in ccip-contracts and use this "local" one directly.

> can this use time formatter from other pr?

> cant this be just set via overrides? Feels weird to add dependency that is not a direct dependency - this is causing noise especially that I have no idea what this package does

> playwright guidelines discourage raw locators, would be better to create helper on the model

### Nit — prefixed, one line, says if it doesn't matter

> NIT: why name casting? not a big deal but this always gets me confused, why not update exported name or why not just use selectActiveContractsByVersionDesc ?

> NIT: We didn't really use _blaBla pattern anywhere and personally I don't like having it. Unfortunately I see it was already introduced in this file :<

> Nit: all other registered keys are chain-agnostic, and `getSchema` here isn't reachable from the app (the factory form comes via `DeployTokenPool.resolve`). Dropping `getSchema` would also cover the missing confirm Copilot mentioned.

### Scope flag / bigger concern — numbered, says whether it blocks

> I see this pattern was already introduced, but it feels wrong to use hardcoded error abis. This will be hard to maintain, supports only subset of errors. All supported contracts and abis are living in ccip-contracts and these abis should be the source of truth.
>
> Not a blocker but we should not go further with current approach.

> TBH I don't see a reason to have -contracts layer here. All it does is it sends built transaction. It could be simplified to just:
> 1) get quote - as far as I understand it has transaction in response ready to be signed?
> 2) in ccip-sdk verify if approval is needed
> 3) in ccip-sdk submit intent using sendTransactionEVM

> merging this pr will "expose" ton connect on prod. we should verify it is allowed, or for simplicity you can just comment this code in pr and use it only locally

> Side question: do we have e2e coverage for this feature? it feels like we don't since according to my knowledge this code won't work

> not in scope but our network conventions fail a bit. we have mainnets (evm), testnets (evm), non-evm/solana... not a big deal but I am not sure why non-evms deserve their own directory :D

### Review summary (top-level body) — one or two lines, states the condition

> Left a few comments, mostly around the Safe handling. Nothing on the schema/router changes, those look good.

> Left some notes. Overall idea, wrapper classes and generators are very cool! My main points:
> - we could utilise existing tools - wagmi/cli to simplify codegen
> - generated contract interfaces could be reduced by ~90% by not rewriting methods

> code looks fine, please resolve conflicts and I will reapprove.

> code looks fine. Tested manually with old draft and new token - both worked

> in general looks good, lets make sure e2e covers issues caused warrooms last time

> approving but can we please have follow up ticket with cleanup on the way?

> Pr looks good. One thing that might be worth discussing - maybe this logic should be a part of ccip-sdk (external)?

> it is more of a product decision, code looks good.

## Self-check before posting

For each drafted comment answer yes to all:

1. Could the author act on it without asking "what do you mean"?
2. Is the first sentence the point?
3. Is there a concrete input, value, chain, or path in it (or is it a one-line question)?
4. Would deleting any sentence lose information? If not, delete it.
5. Does it avoid explaining the author's own code back to them?
