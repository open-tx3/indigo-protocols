# Indigo Protocol - On-Chain Research

## Overview

Indigo Protocol is a decentralized synthetic assets (iAssets) protocol on Cardano.
Users deposit ADA as collateral in CDPs (Collateralized Debt Positions) to mint synthetic assets
(iUSD, iBTC, iETH, iSOL) that track real-world prices. All contracts are PlutusV2.

**GitHub Sources:**
- V1: https://github.com/IndigoProtocol/indigo-smart-contracts (outdated — NOT what's on-chain)
- V2/VX: https://github.com/IndigoProtocol/indigo-upgrade-details-v2 (current on-chain source)

**On-chain version (2026-04-07):**
- CDP validators: **VX** — AdjustCDP has 3 fields `(ts, iasset_change, collateral_change)`
- SP validator: **VX** — request/process pattern with AccountAction
- Staking validator: **V1/V2** — empty datums `Constr(0, [])`, VX upgrade pending
- Collector: **V2** — unchanged

## INDY Governance Token

| Field | Value |
|-------|-------|
| Policy ID | `533bb94a8850ee3ccbe483106489399112b74c905342cb1792a797a0` |
| Asset Name (hex) | `494e4459` (INDY) |
| Fingerprint | `asset1u8caujpkc0km4vlwxnd8f954lxphrc8l55ef3j` |
| Total Supply | 35,000,000 INDY (35000000000000 with 6 decimals) |
| Minted in | 1 tx, never burned |
| Minting Tx | `b5c8b52f91d6e52709cf990d6439adedab2d125ce43348d3a6c7e791a60c02dd` |
| Decimals | 6 |

**NOTE:** The commonly cited policy ID `533bb94a8850ee3ccbe483106489399112b74c905342cb1f14571571` is WRONG.
The correct one ends in `...1792a797a0`.

## iAsset Minting Policy

| Field | Value |
|-------|-------|
| Policy ID / Script Hash | `f66d78b4a3cb3d37afa0ec36461e51ecbde00f26c8f0a68f94b69880` |
| Script Type | PlutusV2 |
| Script Size | 1,189 bytes |
| Reference Script UTxO | `99329591f444f68ed4a33ed664c146fbf278cf9202067974cfa1a26d09a34107#0` |
| Reference Script Address | `addr1w9zewuy339m9l8ax7gz8g08q2j9478lgntzv9qhmwvy3hwg5s6vjv` |

### iAssets Minted Under This Policy

| Asset | Hex Name | Fingerprint | Supply | Decimals |
|-------|----------|-------------|--------|----------|
| iUSD | `69555344` | `asset1rm38ahl5n88c3up6r67y7gn0ffxqwuw7thjxqr` | ~2,092,000 | 6 |
| iBTC | `69425443` | `asset1kfw6plmuzggq7uv90hhvky5p6xycax3l4mru58` | ~10.89 | 6 |
| iETH | `69455448` | `asset1nftftqmrxtgxakuhs8fmkcxl636xutgjm8qk3y` | ~86.56 | 6 |
| iSOL | `69534f4c` | - | ~223.63 | 6 |

## Protocol NFT / Auth Token Policies

| Purpose | Policy ID | Asset Name (hex) | Supply |
|---------|-----------|-------------------|--------|
| CDP NFT | `708f5e6d597fc038d09a738d7be32edd6ea779d6feb32a53668d9050` | `434450` (CDP) | ~521 active |
| CDP_CREATOR Auth | `735b37149eb0c2a5fb590bd60e39fe90ae3a96b6065b05d7aca99ebb` | `4344505f43524541544f52` (CDP_CREATOR) | 24 |
| Stability Pool NFT | `3f28fb7d6c40468262dffb1c3adb568b342499826b664d940085d022` | `53544142494c4954595f504f4f4c` (STABILITY_POOL) | 4 |
| Staking Manager NFT | (TBD - from on-chain) | `7374616b696e675f6d616e616765725f6e6674` (staking_manager_nft) | 1 |
| Staking Token | (TBD - from on-chain) | `7374616b696e675f746f6b656e` (staking_token) | N |
| Account Token (SP) | (TBD - from on-chain) | `6163636f756e745f746f6b656e` (account_token) | N |
| iAsset Config NFT | `97da12de04a6b527cc3b3469c5e5485cf258dfd1021f12e728f2e714` | `494153534554` (IASSET) | 4 |
| iUSD Interest Oracle | `eedb4a24cea6132d3dae1966217f86900e1d8c6a0d668408ecd7eb1b` | `695553445f494e544552455354` | 1 |

## Validator Script Hashes (PlutusV2)

### CDP Spend Validator
- **Script Hash:** `0805d8541db33f4841585fed4c3a7e87e2ff7018243038f06ceb660c`
- **Script Size:** 15,396 bytes
- **Ref UTxO:** `00430c1c2d2c57974069db6597184c8129a934ef0de6c701178bda822fd25a8a#0`
- **Address:** `addr1wyyqtkz5rken7jzptp076np606r79lmsrqjrqw8sdn4kvrqewrkdg` (enterprise, no staking)
- **CDP UTxO Address:** `addr1z...` (type 1 = script payment + key stake) - stake part is user's

### Collector Validator (aka CDP Manager fees)
- **Script Hash:** `0752abd65a0c983bfb1c9c3880cc632c099ba3adb2fe307afb4bbd9c`
- **Script Size:** 3,873 bytes
- **Ref UTxO (addr2):** `f0b4faf71b4ea83fa1a41eadd97d060863576adfc026b11e1fff106ca79e9956#0`
- **Ref UTxO (addr1):** `1080679cc67ab9914282c693bc77816f502a863a96942a31bfcccf1a89e9e2ac#0`
- **Address:** `addr1wyr4927ktgxfswlmrjwr3qxvvvkqnxar4ke0uvr6ld9mm8qrzhplw`
- Holds ADA fees/change from CDP operations; datum is always `Constructor(0, [])`
- **Redeemers:** `Collect(0)`, `UpgradeVersion(1)`

### Stability Pool Validator
- **Script Hash:** `88e0299018563dd10c4860d9f34eda56fdb77f302da0e3980620535c`
- **Script Size:** 15,220 bytes
- **Ref UTxO:** `3356e6602d13e4fcc6563ca2c664b054d528cfe899f32258935d3e886f0d52a4#0`
- **Address:** `addr1wxywq2vsrptrm5gvfpsdnu6wmft0mdmlxqk6pcucqcs9xhqxh5ct9`
- Holds STABILITY_POOL NFTs + iAsset tokens + user deposit UTxOs (1000+ UTxOs)

### INDY Staking Validator
- **Script Hash:** `3bd5f8ba0100f39952472619abfddb52d941a5347b88635e874a7b37`
- **Script Size:** 8,176 bytes
- **Ref UTxO:** `c4e3831610269573d5e506dc072dc8f00b0659e12e9017b6e4d535c8041c0bdf#0`
- **Address:** `addr1x...` (type 3 = script payment + script stake)
- Holds staked INDY tokens

### CDP Creator Validator
- **Script Hash:** `0910f79461a71f74782dcd450f22b0c2cac31ab036b66c9219355d99`
- **Script Size:** 7,047 bytes
- **Ref UTxO:** `b30b10cee01675b02a269c66fa9a420f4766a71b0ebbdd87c6eefbe22b48c59b#0`
- **Address:** `addr1wyy3pau5vxn37arc9hx52rezkrpv4sc6kqmtvmyjry64mxgefqrn0`
- Holds CDP_CREATOR auth tokens

### iAsset Config Validator
- **Script Hash:** `0805d8541db33f4841585fed4c3a7e87e2ff7018243038f06ceb660c` (same as CDP Spend)
- **Address:** `addr1wyyqtkz5rken7jzptp076np606r79lmsrqjrqw8sdn4kvrqewrkdg`
- Holds IASSET config NFTs with oracle addresses embedded in datums

## Reference Script Addresses

### Address 1 (46 scripts)
`addr1w9zewuy339m9l8ax7gz8g08q2j9478lgntzv9qhmwvy3hwg5s6vjv`
- Cred: `4597709189765f9fa6f204743ce0548b5f1fe89ac4c282fb73091bb9`
- Holds: iAsset minting, CDP NFT minting, CDP Manager, and many other validators

### Address 2 (18 scripts)
`addr1wx9gcdlfwa0rx6g33m6fgm8p2dx2v35s8q5z2graqhrjk9cd8u6d4`
- Holds: CDP Spend, CDP Manager, Stability Pool, INDY Staking, CDP Creator, and others

## Oracle

- **Oracle Address (iUSD Interest):** `addr1w9y77mqejj78e993xwr2a9fxygw4nqu7s6t2xdtewl5wttcm2tdyv`
  - Cred: `49ef6c1994bc7c94b13386ae9526221d59839e8696a3357977e8e5af`
  - Holds NFT `eedb4a24...:iUSD_INTEREST`
  - Datum format: `Constructor(0, [int(nonce), Constructor(0, [int(price)]), int(timestamp_ms)])`
  - Example datum: `{ nonce: 411016099050006287, price: 75526 (= ~0.075526 or scaled), timestamp: 1774914908000 }`

- Each iAsset has its own oracle NFT policy (stored in the iAsset config datum):
  - iETH oracle: `6c9497ffd7e8baf86c3c0d6fcd43c524daa49ad5fceba26d715468e9`
  - iBTC oracle: `408f13b240f57c3473b5727a68a88c50c1b6bb15c0c10912008dc59e`
  - iUSD oracle: `e3455f2715338b454fb853442f72dc03b98396854f97510027fe22ff`
  - iSOL oracle: `107992fe118ef6ae341ae0996f355124e03a8fccdcade60692175df2`

- Interest oracle NFTs (from `assetInterestOracleMap` in source):
  - iBTC: `61dd539719ac5aa2ccc493789796f17039d632e0d0a0d11a43dd5cfe:iBTC_INTEREST`
  - iETH: `7b75e317505dddce858ae7bf200656a967c7544e55efa5d18ef30249:iETH_INTEREST`
  - iUSD: `eedb4a24cea6132d3dae1966217f86900e1d8c6a0d668408ecd7eb1b:iUSD_INTEREST`

### Interest Accumulator Computation (2026-04-15 analysis)

The `interest_accumulator` (aka `itUnitaryInterestSnapshot`) in CDP datums is **computed off-chain** by the caller, NOT read directly from any UTxO. The on-chain validator verifies the computation.

**Source data (from interest oracle datum):**
- `od_nonce`: base accumulator value at oracle update time
- `od_price`: annual interest rate as `OnChainDecimal` (decimalUnit = 1,000,000)
- `od_expiration`: timestamp when oracle was last updated

**Constants (from source):**
- `decimalUnit = 1,000,000` (OnChainDecimal scale, in `Indigo.Common.Data.Decimal`)
- `oneYear = 31,536,000,000 ms` (in `Indigo.Common.Utils.Helpers`)

**Approximate formula (simple interest, ~99.93% accuracy):**
```
annual_rate = od_price / decimalUnit          // e.g., 83819/1e6 = 0.083819 = 8.38%
delta_t = user_timestamp - od_expiration      // ms since oracle update
new_accumulator = od_nonce + od_nonce * annual_rate * delta_t / oneYear
```

**Verified against 4 on-chain txs** using oracle `930f2525` (od_nonce=413102016746163059, od_price=83819):

| Tx | Δtime (h) | Δnonce | Rate/h | Match |
|---|---|---|---|---|
| `1291438f` | 48.2 | +460.8B | 9,560B/h | ~99.93% |
| `e2a4461e` | 91.8 | +878.6B | 9,571B/h | ~99.93% |
| `fca78b79` | 101.3 | +969.5B | 9,570B/h | ~99.93% |
| `4016aa6a` | 116.5 | +1,114.7B | 9,568B/h | ~99.93% |

The ~0.07% discrepancy is likely due to compound interest (exact formula is in compiled UPLC, not in public source).

**Impact on tx3:** Cannot compute in tx3 (requires multiplication/division, limitation #3). Must remain a caller-provided param.

---

## Smart Contract Types (from GitHub source code)

### CDPDatum (Constructor indices: CDP=0, IAssetDatum=1)

```haskell
data CDPDatum
  = CDP
      { cdpOwner :: Maybe PaymentPubKeyHash,  -- Nothing = frozen, can be liquidated
        cdpIAsset :: TokenName,               -- "iUSD", "iBTC", etc.
        cdpMintedAmount :: Integer             -- amount of iAsset minted
      }
  | IAssetDatum IAsset
```

**On-chain CBOR structure (Create CDP example):**
```
Constr(0, [           -- CDPDatum::CDP
  Constr(0, [         -- CDP record (inner wrapper)
    Constr(0, [       -- Maybe::Just(owner_pkh)
      owner_pkh       -- Bytes: user's payment pub key hash
    ]),
    iAsset_name,      -- Bytes: e.g., "69555344" = "iUSD"
    minted_amount,    -- Int: amount of iAsset minted (e.g., 7340000)
    Constr(1, [       -- Interest tracking (rational-like)
      int(timestamp), -- POSIX time ms when created
      int(nonce)      -- large integer seed/accumulator
    ])
  ])
])
```

**NOTE:** The on-chain datum has 4 fields inside the inner Constr(0), but the GitHub source shows only 3 fields (cdpOwner, cdpIAsset, cdpMintedAmount). The 4th field is likely interest-tracking data added in a later protocol upgrade.

### CDPRedeemer (VX on-chain constructor indices)

| Index | Variant | Fields | Description |
|-------|---------|--------|-------------|
| 0 | `AdjustCDP` | `currentTime: Int, mintedChange: Int, collateralChange: Int` | Modify existing CDP |
| 1 | `CloseCDP` | `currentTime: Int` | Close CDP, burn debt, recover collateral |
| 2 | `RedeemCDP` | `currentTime: Int` | Redeem CDP |
| 3 | `FreezeCDP` | `currentTime: Int` | Freeze under-collateralized CDP |
| 4 | `MergeCDPs` | none | Merge multiple frozen CDPs |
| 5 | `MergeAuxiliary` | `mainMergeUtxo: TxOutRef` | Auxiliary UTxO in merge tx |
| 6 | `Liquidate` | none | Liquidate frozen CDP against stability pool |
| 7 | `UpdateOrInsertAsset` | none | Update iAssetDatum |
| 8 | `UpgradeVersion` | none | Protocol version upgrade |

**NOTE:** V1 had different indices (Liquidate=2, FreezeCDP=5, etc.). The VX indices above are what's deployed on-chain.

### CDPCreatorRedeemer (Constructor indices)

| Index | Variant | Fields | Description |
|-------|---------|--------|-------------|
| 0 | `CreateCDP` | `owner_pkh: PaymentPubKeyHash, mint_amount: Int, collateral: Int, timestamp: Int` | Create new CDP |
| 1 | `UpgradeCreatorVersion` | none | Protocol version upgrade |

**On-chain redeemer CBOR (Create CDP):**
```
Constr(0, [owner_pkh, mint_amount, collateral_lovelace, timestamp_ms])
```

### StabilityDatum (Constructor indices: StabilityPoolDatum=0, AccountDatum=1, SnapshotEpochToScaleToSumDatum=2)

```haskell
data StabilityDatum
  = StabilityPoolDatum
      { spIAsset :: TokenName,
        spSnapshot :: StabilityPoolSnapshot,
        epochToScaleToSum :: EpochToScaleToSum   -- Map (Int, Int) SPInteger
      }
  | AccountDatum
      { accOwner :: PaymentPubKeyHash,
        accIAsset :: TokenName,
        accSnapshot :: StabilityPoolSnapshot
      }
  | SnapshotEpochToScaleToSumDatum
      { sessSnapshot :: EpochToScaleToSum,
        sessAsset :: TokenName
      }
```

### StabilityPoolSnapshot

```haskell
data StabilityPoolSnapshot = StabilityPoolSnapshot
  { snapshotP :: SPInteger,      -- Product snapshot (18-decimal precision)
    snapshotD :: SPInteger,      -- Deposit snapshot
    snapshotS :: SPInteger,      -- Sum snapshot
    snapshotEpoch :: Integer,    -- Pool emptying counter
    snapshotScale :: Integer     -- Prevents P truncation to 0
  }
```

### StabilityPoolRedeemer (VX on-chain constructor indices)

| Index | Variant | Fields | Description |
|-------|---------|--------|-------------|
| 0 | `RequestAction` | `action: AccountAction` | User requests create/adjust/close (V2 pattern) |
| 1 | `ProcessRequest` | `request: TxOutRef` | Batcher processes a request |
| 2 | `AnnulRequest` | none | Cancel a pending request |
| 3 | `LiquidateCDP` | none | Liquidate CDP against pool (bot) |
| 4 | `RecordEpochToScaleToSum` | none | Record snapshot (admin) |
| 5 | `UpgradeVersion` | none | Protocol upgrade |

### AccountAction (nested in RequestAction)

| Index | Variant | Fields |
|-------|---------|--------|
| 0 | `Create` | none |
| 1 | `Adjust` | `amount: Int, outputAddress: Address` |
| 2 | `Close` | `outputAddress: Address` |

**NOTE:** V1 used direct redeemers (CreateAccount, AdjustAccount, Close, SpendAccount). VX uses two-step request/process pattern.

### StakingDatum (Constructor indices: StakingManager=0, StakingPosition=1)

```haskell
data StakingDatum
  = StakingManager
      { totalStake :: Integer,
        mSnapshot :: RewardSnapshot     -- { snapshotAda :: OnChainDecimal }
      }
  | StakingPosition
      { owner :: PaymentPubKeyHash,
        lockedAmount :: Map Integer (Integer, POSIXTime),  -- poll_id -> (vote_amt, end_time)
        pSnapshot :: RewardSnapshot
      }
```

### StakingRedeemer (V2 on-chain constructor indices — VX removes Unlock)

| Index | Variant | Fields | Description |
|-------|---------|--------|-------------|
| 0 | `CreateStakingPosition` | `pkh: PaymentPubKeyHash` | Create new staking position |
| 1 | `UpdateTotalStake` | none | Update manager's total stake |
| 2 | `Distribute` | none | Distribute ADA rewards (bot) |
| 3 | `AdjustStakedAmount` | `amount: Int` | Add/remove INDY from staking |
| 4 | `Unstake` | none | Remove staking position entirely |
| 5 | `Lock` | none | Lock INDY for governance voting |
| 6 | `UpgradeVersion` | none | Protocol upgrade (was Unlock=6, UpgradeVersion=7 in V1) |

**NOTE:** On-chain staking datums are ALL `Constr(0, [])` (empty). The VX source defines
`StakingPositionContent` with fields (owner, lockedAmount, snapshot) but the upgrade hasn't
been activated yet. All 664+ staking UTxOs use empty datums.

### CollectorRedeemer (Constructor indices)

| Index | Variant | Fields | Description |
|-------|---------|--------|-------------|
| 0 | `Collect` | none | Distribute collected fees |
| 1 | `UpgradeVersion` | none | Protocol upgrade |

---

## Transaction Classification

### User-Facing Transactions (implemented in TX3)

| # | TX Name | Version | Redeemer | Description |
|---|---------|---------|----------|-------------|
| 1 | `create_cdp` | VX | CDPCreator `CreateCDP(0)` | Deposit ADA collateral, mint iAssets |
| 2 | `adjust_cdp_mint` | VX | CDP `AdjustCDP(0, [ts, +amount, 0])` | Mint more iAssets |
| 3 | `adjust_cdp_burn` | VX | CDP `AdjustCDP(0, [ts, -amount, collat])` | Burn iAssets / withdraw collateral |
| 4 | `close_cdp` | VX | CDP `CloseCDP(1, [ts])` | Close CDP, burn debt + NFT |
| 5 | `create_sp_account` | VX | SP `RequestAction(0, Create)` | Request SP account creation |
| 6 | `adjust_sp_account` | VX | SP `RequestAction(0, Adjust)` | Request SP deposit adjustment |
| 7 | `close_sp_account` | VX | SP `RequestAction(0, Close)` | Request SP account closure |
| 8 | `create_staking` | V1/V2 | Staking `CreateStakingPosition(0)` | Stake INDY tokens |
| 9 | `adjust_staking` | V1/V2 | Staking `AdjustStakedAmount(3)` | Add/remove INDY |
| 10 | `unstake` | V1/V2 | Staking `Unstake(4)` | Close staking position |

### Admin/Bot Transactions (NOT implemented)

| TX Name | Contract | VX Redeemer | Description |
|---------|----------|-------------|-------------|
| `freeze_cdp` | CDP | `FreezeCDP(3)` | Bot freezes under-collateralized CDPs |
| `liquidate_cdp` | CDP + SP | `Liquidate(6)` + `LiquidateCDP(3)` | Bot liquidates frozen CDPs |
| `merge_cdps` | CDP | `MergeCDPs(4)` / `MergeAuxiliary(5)` | Merge frozen CDPs |
| `process_sp_request` | SP | `ProcessRequest(1)` | Batcher processes SP create/adjust/close |
| `distribute_rewards` | Staking | `Distribute(2)` | Distribute ADA rewards to stakers |

---

## Detailed Transaction Analysis

### Create CDP (tx: `8f586803c607a0b770b7f8547d92260105d5ce7b9784d1ff1255dcac7342a6a3`)

**Block:** 13,183,571 | **Epoch:** 619 | **Fee:** 947,940 lovelace
**Validity interval:** slot 182,448,064 to 182,449,264 (~20 min window)
**Required signer:** user's PKH

**Inputs (3):**

| Index | UTxO | Address | Role | Value |
|-------|------|---------|------|-------|
| 0 | `54b1b172...#1` | User wallet (vkey) | Funds source | 68,693,573 lovelace |
| 1 | `92deaea4...#2` | Collector script (`0752abd6`) | Continuity | 32,353,920 lovelace |
| 2 | `dfd76bb5...#1` | CDPCreator script (`0910f794`) | Auth token | 1,094,740 lovelace |

**Reference Inputs (7):**
- Oracle/config UTxOs (oracle data, iAsset config)
- Reference scripts (iAsset minting, CDP NFT minting, CDP Manager, Collector)

**Minting:**
- +1 CDP NFT (`708f5e6d...` : `434450`)
- +7,340,000 iUSD (`f66d78b4...` : `69555344`)

**Outputs (4):**

| Index | Address | Role | Value | Datum |
|-------|---------|------|-------|-------|
| 0 | `addr1z...` (CDP script + user stake) | CDP UTxO | 55,000,000 lovelace + 1 CDP NFT | CDPDatum (see above) |
| 1 | `addr1wyy3pau...` (CDPCreator) | Return auth token | 1,094,740 lovelace + CDP_CREATOR token | `Constr(0,[])` |
| 2 | `addr1wyr4927...` (Collector) | Fees continuity | 32,381,408 lovelace | `Constr(0,[])` |
| 3 | User wallet | Change + minted iUSD | 12,718,145 lovelace + 8,613,078 iUSD | none |

**Redeemers:**
- SPEND[1] (Collector): `Constr(0,[])` (Collect)
- SPEND[2] (CDPCreator): `Constr(0, [owner_pkh, 7340000, 55000000, 1774014476000])` (CreateCDP)
- MINT[0] (CDP NFT): `Constr(0,[])` (unit)
- MINT[1] (iUSD): `Constr(0,[])` (unit)

**Metadata (tag 674):** `"[Cardano MCP] - Submit Transaction"`

### Close CDP (tx: `0c385168e1d4a75b0f3bdfd4665149ead5b8b3a53e0e163da0ef8bf4ee358831`)

**Fee:** 1,631,625 lovelace | **No validity interval**
**Required signer:** user's PKH

**Inputs (5):**

| Index | UTxO | Script Hash | Role | Value |
|-------|------|-------------|------|-------|
| 0 | `33831e50...#0` | (vkey) | User wallet | 172,808,255 lovelace |
| 1 | `8a0b5b2d...#1` | `0752abd6` | Collector | 37,269,723 lovelace |
| 2 | `ad6d5edb...#1` | `3bd5f8ba` | Staking (INDY) | 23,884,175 lovelace |
| 3 | `ae23f01b...#0` | `0805d854` | CDP UTxO | 9,985,966 lovelace |
| 4 | `ca53eca7...#3` | `88e02990` | Stability Pool | 343,703,740,699 lovelace |

**Reference Inputs (6):** All reference scripts for the validators involved.

**Minting (burning):**
- -1 CDP NFT (`708f5e6d...` : `434450`)
- -1,611,728 iUSD (`f66d78b4...` : `69555344`)

**Outputs (4):**

| Index | Script Hash | Role | Value | Datum |
|-------|-------------|------|-------|-------|
| 0 | `3bd5f8ba` (Staking) | INDY staking updated | 26,615,926 + 32,333,290 INDY | `Constr(0,[])` |
| 1 | `0752abd6` (Collector) | Fees continuity | 37,402,732 lovelace | `Constr(0,[])` |
| 2 | `88e02990` (StabilityPool) | Pool updated | 343,710,861,905 + SP NFT + 522B iUSD | SP state datum |
| 3 | (vkey) | User change | 171,176,630 lovelace | none |

**Redeemers:**
- SPEND[1] (Collector): `Constr(0,[])` (Collect)
- SPEND[2] (Staking): `Constr(4,[])` (Unstake)
- SPEND[3] (CDP Spend): `Constr(6,[])` (MergeCDPs — likely CloseCDP on-chain)
- SPEND[4] (StabilityPool): `Constr(3,[])` (Close)
- MINT[0] (CDP NFT): `Constr(0,[])` (unit)
- MINT[1] (iUSD): `Constr(0,[])` (unit)

**NOTE:** This tx uses Constr(6) for CDP spend which is `Liquidate` in VX indices (not `CloseCDP`=1).
This was a liquidation transaction, not a voluntary close. The VX constructor indices are confirmed:
AdjustCDP=0, CloseCDP=1, RedeemCDP=2, FreezeCDP=3, MergeCDPs=4, MergeAuxiliary=5, Liquidate=6, UpdateAsset=7, UpgradeVersion=8.

---

## Key Observations

1. **All scripts are PlutusV2** — no V1 or V3 contracts
2. **On-chain version is VX** (from indigo-upgrade-details-v2 repo), not the V1 in indigo-smart-contracts
3. **Two reference script addresses** store all protocol scripts
4. **CDP addresses use type-1 format** (addr1z) with script payment + user's stake key
5. **4 Stability Pool NFTs** = one per iAsset (iUSD, iBTC, iETH, iSOL)
6. **24 CDP_CREATOR auth tokens** = controls who can create CDPs (distributed across UTxOs)
7. **Interest tracked per-CDP** via `ActiveCDPInterestTracking { lastSettled, unitaryInterestSnapshot }` (VX)
8. **Oracle data** is read via reference inputs (not consumed)
9. **Collector UTxO** is consumed and recreated in CDP operations (continuity pattern)
10. **Companion Staking input** required in every CDP adjust/close operation with `Constr(4)` redeemer
11. **SP uses request/process pattern** (VX): user submits RequestAction, batcher does ProcessRequest
12. **Staking datums are empty** `Constr(0,[])` — all 664+ UTxOs. VX upgrade not yet activated
13. **Minting redeemers are always `Constr(0,[])`** (unit) for both CDP NFT and iAsset policies
