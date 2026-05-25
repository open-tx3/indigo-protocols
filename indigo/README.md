# Indigo Protocol

tx3 protocol for [Indigo Protocol](https://indigoprotocol.io/) — synthetic assets (iAssets) on Cardano.

> **Status: WIP** — This tx3 off-chain implementation is under testing and may not cover all edge cases yet.

## Overview

Indigo allows users to deposit ADA as collateral in Collateralized Debt Positions (CDPs) to mint synthetic assets like iUSD, iBTC, iETH, and iSOL. The protocol also supports Stability Pool deposits and INDY token staking. All on-chain contracts are PlutusV2.

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

## Important Considerations

- **On-chain version mismatch:** The deployed contracts differ from the public GitHub source in several ways (extra datum fields, different redeemer structures). This tx3 was built from on-chain analysis and verified against real transactions.
- **CDP validator version:** On-chain CDPs use VX (AdjustCDP with 3 fields). Staking validators still use V1/V2 with empty datums.
- **Double-wrapped datums:** CDP datums are double-wrapped (`Constr(0, [Constr(0, [fields...])])`), which the type definitions reflect.
- **Network profile required:** Policy IDs, reference script UTxOs, and script addresses must be configured per network.
- **Reference scripts:** 7 different reference script UTxOs are required (CDP spend, CDP creator, collector, iAsset mint, CDP NFT mint, stability pool, staking).
- **Stability Pool:** SP operations use a request/process two-step pattern — user submits a request, then a batcher processes it.

## Caller Preparation

Many values must be queried from on-chain UTxO datums before invoking transactions. tx3 cannot read datum fields from reference inputs, so the caller must query them via Koios/Ogmios and pass them as parameters.

### All CDP transactions

- `timestamp_ms: Int` — Current POSIX timestamp in milliseconds (from oracle datum `od_expiration` or current time).
- `interest_accumulator: Int` / `accumulator: Int` — The current interest accumulator value from the oracle datum.
- `oracle_utxo`, `iasset_config_utxo`, `cdp_manager_utxo` — Reference input UTxOs that must be queried and provided.

### `adjust_cdp_mint` / `adjust_cdp_burn`

- `new_minted_total: Int` — The new total minted amount after the operation (current + additional or current - burn). Must be computed off-chain.
- `new_collateral: Int` — The new collateral amount in the CDP. For mint: same as current. For burn: current - withdrawn.

### `close_cdp`

- `pool_iasset: Int` — The iAsset amount in the Stability Pool (from SP pool UTxO datum).
- `sp_snapshot_p`, `sp_snapshot_d`, `sp_snapshot_s`, `sp_snapshot_epoch`, `sp_snapshot_scale` — All 5 fields from the Stability Pool's `snapshot` datum. These would be eliminated if tx3 supported reading reference input datums (~20 params total across SP txs).

### Stability Pool transactions (`create_sp_account`, `adjust_sp_account`, `close_sp_account`)

- `sp_snapshot_*` or `acc_snapshot_*` (5 fields each) — Pool or account snapshot values from the corresponding on-chain datum.
- `output_addr: Address` — The user's output address for receiving funds.

### Staking transactions (`create_staking`, `adjust_staking`, `unstake`)

- `indy_policy_id`, `indy_asset_name` — INDY token identifiers.
- `staking_token_policy_id`, `staking_token_name` — Staking position NFT identifiers.
- `manager_utxo`, `collector_utxo` — Protocol UTxOs that must be queried on-chain.

## tx3 Limitations

Several tx3 language limitations affect this protocol. The most impactful: **cannot read datum fields from reference inputs**, requiring ~20 extra parameters that could otherwise be extracted automatically. For the full list, see [investigacion/tx3-limitations-indigo.md](investigacion/tx3-limitations-indigo.md).

## Smart Contracts

- PlutusV2
- Source: [IndigoProtocol/indigo-smart-contracts](https://github.com/IndigoProtocol/indigo-smart-contracts)
- VX upgrade: [IndigoProtocol/indigo-upgrade-details-v2](https://github.com/IndigoProtocol/indigo-upgrade-details-v2)
