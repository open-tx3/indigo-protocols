# Indigo Protocol — TX3 Comparison Report (VX / Latest On-Chain)

## Protocol Version

On-chain deployment matches **VX** from `IndigoProtocol/indigo-upgrade-details-v2`.
- CDP validators: **VX** — AdjustCDP has 3 fields `(timestamp, iasset_change, collateral_change)`
- SP validator: **VX** — request/process pattern with AccountAction
- Staking validator: **V1/V2** — empty datums `Constr(0,[])`, VX upgrade pending

## Generated Transactions (10/10 generate CBOR)

| # | Transaction | Version | Status |
|---|---|---|---|
| 1 | `create_cdp` | VX | ✅ |
| 2 | `adjust_cdp_mint` | VX | ✅ |
| 3 | `adjust_cdp_burn` | VX | ✅ |
| 4 | `close_cdp` | VX | ✅ |
| 5 | `create_sp_account` | VX | ✅ |
| 6 | `adjust_sp_account` | VX | ✅ |
| 7 | `close_sp_account` | VX | ✅ |
| 8 | `create_staking` | V1/V2 | ✅ |
| 9 | `adjust_staking` | V1/V2 | ✅ |
| 10 | `unstake` | V1/V2 | ✅ |

---

## Detailed CBOR Comparison: create_cdp

### Reference TX: `8f586803c607a0b770b7f8547d92260105d5ce7b9784d1ff1255dcac7342a6a3`

### Inputs

| # | Generated | On-chain | Match |
|---|---|---|---|
| Count | 3 (CDPCreator + Collector + User) | 3 | ✅ |
| Script inputs | Collector `0752ab...` + CDPCreator `0910f7...` | Same hashes | ✅ |

### Outputs

**Output 0 — CDP UTxO:**

| Field | Generated | On-chain | Match |
|---|---|---|---|
| Address | `0x11` script+stake, `0805d854...` | Same | ✅ |
| Value | 55M lovelace + 1 CDP NFT | Same | ✅ |
| Datum | `Constr(0,[Constr(0,[JustPkh, "iUSD", amount, Constr(0,[ts, acc])])])` | Same structure | ✅ |

**Output 1 — CDPCreator:** Address `0910f7...`, datum `Constr(0,[])` ✅
**Output 2 — Collector:** Address `0752ab...`, datum `Constr(0,[])` ✅
**Output 3 — User change:** ADA + minted iUSD ✅

### Minting, Redeemers, Reference Inputs

| Component | Generated | On-chain | Match |
|---|---|---|---|
| Minting | +1 CDP NFT, +N iUSD | Same | ✅ |
| Redeemers | 4 (2 spend + 2 mint), correct Constr indices | Same | ✅ |
| Reference inputs | 7 (4 scripts + 3 data) | 7 | ✅ |
| Required signers | 1 PKH | 1 PKH | ✅ |
| Validity | since_slot + until_slot | Both present | ✅ |

### Verdict: **~100% structural match** ✅

---

## Detailed CBOR Comparison: adjust_cdp_mint

### Reference TX: `a98820c6900b59f2277483c88678cacc2728da16c70afb3fde04b93e708eab90`

| Field | Generated | On-chain | Match |
|---|---|---|---|
| Inputs | 4 (CDP + Staking + Collector + User) | 5 (same scripts + extra wallet UTxO) | ✅ Same script structure |
| Outputs | 4 (CDP + Staking + Collector + User) | 4 | ✅ |
| Redeemers | 4 (AdjustCDP[ts,+amt,0] + Staking[4] + Collector[0] + Mint[0]) | 4 | ✅ |
| Reference inputs | 7 | 8 | ⚠️ 1 less (ok) |
| CDP datum | Double-wrapped, correct fields | Same | ✅ |
| Staking companion | `Constr(4,[])` with empty datum output | Same | ✅ |

### Verdict: **~100% structural match** ✅

---

## Remaining Differences (inherent, not bugs)

| Item | Impact | Notes |
|---|---|---|
| Interest accumulator | Low | Caller-provided value, not read from oracle datum. Source TBD (possibly computed off-chain) |
| SP/Account snapshot values | Low | Values live in pool/account UTxO datums (spent inputs + variant type). Caller must read and pass as params |
| ExUnits placeholder | None | Expected for `--skip-submit` (2M mem / 2B steps) |
| CBOR encoding style | None | Definite vs indefinite length — semantically identical |
| Collateral return/total | None | Optional fields, not included by tx3 |
| Metadata | None | On-chain Indigo txs have no metadata (confirmed null) |

## TX3 Limitations

See `investigacion/tx3-limitations-indigo.md` for full details.
Updated 2026-04-15 after tx3c fixes [#316](https://github.com/tx3-lang/tx3/pull/316) (field name shadowing) and [#318](https://github.com/tx3-lang/tx3/pull/318) (typed reference datums).

**Bugs (workaround applied):**

| # | Bug | Status |
|---|---|---|
| 1 | Type names with primitive prefixes → parse error | Workaround in place (`CdpInterest`), not yet fixed in tx3c |
| 2 | ~~Param = field name → lowering panic~~ | **Fixed** (tx3c [#316](https://github.com/tx3-lang/tx3/pull/316)). Prefixes `cr_`/`ci_` removed, original field names restored |

**Active limitations:**

| # | Limitation | Impact |
|---|---|---|
| 3 | No multiplication/division | Caller pre-computes ratios, fees, prices |
| 4 | ~~Cannot read datum from reference inputs~~ | **Syntax fixed** (tx3c [#318](https://github.com/tx3-lang/tx3/pull/318)). `datum_is: OracleDatum` now works on reference blocks. However, `timestamp_ms` and `interest_accumulator`/`accumulator` are caller-provided (not from oracle datum), so params remain |
| 5 | Datum field access fails on variant types | SP account/pool fields passed as explicit params (~13 params across 4 txs) |

**Future limitation (not blocking today):**

| # | Limitation | When it blocks |
|---|---|---|
| 6 | No tuple types (`Map<K,(V1,V2)>`) | When Indigo activates VX staking upgrade (lockedAmount field) |

## How to Reproduce

```bash
cd protocols_tx3/indigo
trix check && trix build

trix invoke --skip-submit --profile mainnet --args-json-path invoke-args/create_cdp.json
trix invoke --skip-submit --profile mainnet --args-json-path invoke-args/adjust_cdp_mint.json
trix invoke --skip-submit --profile mainnet --args-json-path invoke-args/adjust_cdp_burn.json
trix invoke --skip-submit --profile mainnet --args-json-path invoke-args/close_cdp.json
trix invoke --skip-submit --profile mainnet --args-json-path invoke-args/create_sp_account.json
trix invoke --skip-submit --profile mainnet --args-json-path invoke-args/adjust_sp_account.json
trix invoke --skip-submit --profile mainnet --args-json-path invoke-args/close_sp_account.json
trix invoke --skip-submit --profile mainnet --args-json-path invoke-args/create_staking.json
trix invoke --skip-submit --profile mainnet --args-json-path invoke-args/adjust_staking.json
trix invoke --skip-submit --profile mainnet --args-json-path invoke-args/unstake.json
```
