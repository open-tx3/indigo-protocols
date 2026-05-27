# Indigo Protocol

[Indigo](https://indigoprotocol.io/) is a decentralised synthetic-asset protocol on Cardano. Users lock ADA as collateral inside a Collateralised Debt Position (CDP) and mint *iAssets* — on-chain synthetic representations of real-world assets such as the US dollar (iUSD), Bitcoin (iBTC), Ether (iETH), and Solana (iSOL). The synthetic side is backed by over-collateralised ADA reserves and tracks underlying prices via on-chain oracles.

Indigo also runs two adjacent products in the same contract suite: a **Stability Pool**, where iAsset holders deposit synthetics to absorb under-collateralised CDPs in exchange for a share of liquidated collateral, and **INDY staking**, where holders of the governance token stake to vote on protocol parameters and earn protocol revenue. End users typically interact through the Indigo web app to open positions, borrow against ADA collateral, and earn yield on idle iAssets.

This tx3 covers the full user-facing surface: CDP open / adjust / close, Stability Pool create / adjust / close (request side), and INDY staking. Batcher-side processing of SP requests is out of scope.

## Overview

Indigo is a script-heavy protocol — every transaction consumes one or more reference inputs (oracles, iAsset config, CDP manager, collector) and at least one validator UTxO. Architecturally:

- **CDPs** are individual script UTxOs holding ADA collateral, with the synthetic debt encoded in the datum. Minting an iAsset requires the on-chain interest accumulator and oracle price, both supplied as caller parameters.
- **Stability Pool** uses a two-step pattern: the user submits a *request* UTxO to the pool, and a batcher later processes the request against the pool state. This tx3 implements the request side only.
- **INDY staking** is a simple lock-with-NFT pattern; current on-chain validators use V1/V2 with empty datums.

This implementation was built from on-chain analysis and verified against real mainnet transactions; the deployed contracts differ in several ways from the public GitHub source.

## Transactions

| Transaction | Description |
|---|---|
| `open_cdp` | Deposit ADA collateral and mint iAssets in a new CDP |
| `close_cdp` | Burn iAssets and reclaim collateral, closing the CDP |
| `deposit_collateral` | Add more ADA collateral to an existing CDP |
| `withdraw_collateral` | Remove excess collateral from a CDP |
| `mint_more` | Mint additional iAssets against existing collateral |
| `repay` | Burn iAssets to reduce CDP debt |
| `deposit_sp` | Request a deposit into a Stability Pool |
| `withdraw_sp` | Request a withdrawal from a Stability Pool |
| `stake_indy` | Stake INDY governance tokens |
| `unstake_indy` | Unstake INDY governance tokens |

## Important considerations

- **On-chain version mismatch.** The deployed contracts differ from the public GitHub source in several ways (extra datum fields, different redeemer structures). This tx3 was built from on-chain analysis and verified against real transactions.
- **CDP validator version.** On-chain CDPs use VX (`AdjustCDP` with 3 fields). Staking validators still use V1/V2 with empty datums.
- **Double-wrapped datums.** CDP datums are double-wrapped (`Constr(0, [Constr(0, [fields...])])`), which the type definitions reflect.
- **Network profile required.** Policy IDs, reference script UTxOs, and script addresses must be configured per network in the trix profile.
- **Reference scripts.** Seven different reference-script UTxOs are required: CDP spend, CDP creator, collector, iAsset mint, CDP NFT mint, stability pool, staking.
- **Stability Pool request/process pattern.** SP operations are two-step — user submits a request UTxO; a batcher processes it. This tx3 implements only the request side.
- **Caller-provided oracle values.** Although `datum_is` lets tx3 read fields from reference inputs, `timestamp_ms` and the interest accumulator are independent of the oracle datum and must be supplied by the caller.

## Caller preparation

Many values must be queried from on-chain UTxO datums before invoking a transaction. tx3 cannot read variant-type datum fields from spent inputs, so the Stability Pool transactions require the full snapshot tuple as explicit parameters.

### All CDP transactions

| Parameter | Source |
|---|---|
| `timestamp_ms: Int` | Current POSIX timestamp in milliseconds. Independent of the oracle's `od_expiration`. |
| `interest_accumulator: Int` / `accumulator: Int` | Current interest accumulator value. Independent of `od_nonce`; computed off-chain. |
| `oracle_utxo`, `iasset_config_utxo`, `cdp_manager_utxo` | Reference-input UTxOs queried on-chain. |

### `adjust_cdp_mint` / `adjust_cdp_burn`

| Parameter | Source |
|---|---|
| `new_minted_total: Int` | New total minted amount after the operation (current ± additional). Computed off-chain. |
| `new_collateral: Int` | New collateral amount in the CDP. For mint: unchanged. For burn: current − withdrawn. |

### `close_cdp`

| Parameter | Source |
|---|---|
| `pool_iasset: Int` | iAsset amount in the Stability Pool (from SP pool UTxO datum). |
| `sp_snapshot_p`, `sp_snapshot_d`, `sp_snapshot_s`, `sp_snapshot_epoch`, `sp_snapshot_scale` | All 5 fields from the Stability Pool's `snapshot` datum. Would be eliminated by tx3 support for variant-type datum field access on spent inputs (~20 params total across SP txs). |

### Stability Pool transactions (`create_sp_account`, `adjust_sp_account`, `close_sp_account`)

| Parameter | Source |
|---|---|
| `sp_snapshot_*` (5 fields) | Pool snapshot values from the corresponding on-chain datum. |
| `acc_snapshot_*` (5 fields) | Account snapshot values from the user's SP account datum. |
| `output_addr: Address` | User's output address for receiving funds. |

### Staking transactions (`create_staking`, `adjust_staking`, `unstake`)

| Parameter | Source |
|---|---|
| `indy_policy_id`, `indy_asset_name` | INDY token identifiers. |
| `staking_token_policy_id`, `staking_token_name` | Staking-position NFT identifiers. |
| `manager_utxo`, `collector_utxo` | Protocol UTxOs queried on-chain. |

## References

- **Smart contracts:** PlutusV2 — [IndigoProtocol/indigo-smart-contracts](https://github.com/IndigoProtocol/indigo-smart-contracts)
- **VX upgrade:** [IndigoProtocol/indigo-upgrade-details-v2](https://github.com/IndigoProtocol/indigo-upgrade-details-v2)
- **Homepage / app:** [indigoprotocol.io](https://indigoprotocol.io/)
- **Research notes:** [`investigacion/`](./investigacion/) — on-chain analysis, datum decoding, and version-mismatch findings.
