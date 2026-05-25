# TX3 Limitations Found During Indigo Protocol Implementation

Discovered with trix 0.21.1 (2026-04-07).
Updated 2026-04-15 after tx3c fixes [#316](https://github.com/tx3-lang/tx3/pull/316), [#318](https://github.com/tx3-lang/tx3/pull/318).

---

## Bugs (resolved)

### 1. Type Names Starting with Primitive Keywords (Parser)

Custom type names starting with `Int`, `Bool`, `Bytes`, `Address`, `UtxoRef`, or `AnyAsset` cause parse failures. PEG parser matches primitive keyword greedily.

**Example:** `InterestData` → parser matches `Int`, fails on `erestData`.
**Workaround applied:** Renamed to `CdpInterest`. (still in place — bug not yet fixed)
**Fix for tx3-lang:** `("Int" | ...) ~ !ASCII_ALPHANUMERIC`

### 2. Param/Field Name Collision (Lowering Panic) — FIXED ([#316](https://github.com/tx3-lang/tx3/pull/316))

When a tx param has the same name as a type field, lowering panics with `not yet implemented`.

**Example:** Param `owner_pkh` collides with `CDPCreatorRedeemer::CreateCDP { owner_pkh }`.
**~~Workaround applied:~~** ~~Prefixed type fields: `cr_owner`, `ci_timestamp`.~~
**Fixed in tx3c [#316](https://github.com/tx3-lang/tx3/pull/316):** Support shadowing of record field names. Prefixes removed, original field names restored.

---

## Active Limitations (impact on production use)

### 3. No Multiplication/Division

Only `+` and `-` supported. Cannot compute collateral ratios, interest rates, or prices.

**Impact:** All computed values must be pre-calculated by the API caller before invoking.
**Affected params:** Collateral ratios, protocol fees, price calculations.

### 4. Cannot Read Datum from Reference Inputs — SYNTAX FIXED ([#318](https://github.com/tx3-lang/tx3/pull/318))

`reference` blocks now support `datum_is` to parse and expose datum fields.

**Fixed in tx3c [#318](https://github.com/tx3-lang/tx3/pull/318):** `datum_is: OracleDatum` now works on `reference` blocks, enabling field access like `oracle_data.od_price`. Applied structurally to all 4 CDP oracle references.

**However, `timestamp_ms` and `interest_accumulator`/`accumulator` remain as caller-provided params.** On-chain analysis confirmed these values do NOT come from the oracle datum:
- `timestamp_ms` is the user's "current time" (differs from `od_expiration` by days)
- `interest_accumulator`/`accumulator` differs from `od_nonce`

The oracle reference input is used by the on-chain validator for price/interest rate checks, but the timestamp and accumulator are provided independently by the caller.

**Still blocked (spent inputs + variant types):**

SP pool/account UTxOs are SPENT inputs (not reference inputs), and `StabilityDatum` is a variant type (limitation #5). These params remain:

| Param | Blocked by |
|---|---|
| `sp_snapshot_p/d/s/epoch/scale` | spent input + variant field access (#5) |
| `pool_iasset` | Same |
| `owner_pkh` (SP txs) | Same |
| `iasset_name` (SP txs) | Same |
| `acc_snapshot_p/d/s/epoch/scale` | Same |

### 5. Datum Field Access on Variant Types (still active)

Field access (`input.field`) works on record types but fails on variant types with "not in scope".
Spread (`...input`) inside a variant constructor does work.

**Impact:** SP pool and account datums (`StabilityDatum::PoolState`, `StabilityDatum::Account`) require all fields as explicit params instead of reading from input datum. This is the primary remaining blocker for reducing SP tx params (~13 params across 3 SP txs + close_cdp).

**Works:** `...position_input` inside `StakingDatum::StakingPosition { ... }`
**Fails:** `account_input.acc_content.acc_owner` on `StabilityDatum` (variant)
**Not fixed by [#316](https://github.com/tx3-lang/tx3/pull/316) or [#318](https://github.com/tx3-lang/tx3/pull/318)** — those fix name shadowing and reference datum access respectively, not variant field access on spent inputs.

---

## Future Limitations (not blocking today, will block on upgrade)

### 6. No Tuple Types

Cannot represent `Map<Int, (Int, POSIXTime)>`.

**Impact today:** None — on-chain staking uses empty datums `Constr(0, [])`.
**Impact on VX staking upgrade:** Will block modeling `StakingPositionContent.lockedAmount` field when Indigo activates VX staking validators with full datum fields.

**Proposed fix:** Add tuple support or allow named records as Map values.
