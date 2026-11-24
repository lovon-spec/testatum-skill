# Jurisdiction Guide: United States (Federal / IRC & IRS)

Apply this guide for U.S. persons (citizens, green-card holders, and resident aliens) subject to federal income tax under the Internal Revenue Code (IRC). Always verify whether Congress, Treasury/IRS, or FinCEN enacted new statutory or regulatory changes for the target tax year, and remind the taxpayer to verify **state income tax** conformity separately.

> **Scope note:** This guide documents U.S. federal tax rules, primary IRS citations, Form 1099-DA / Form 8949 mechanics, and recurring classification ambiguities so the accounting package is structured accurately for U.S. filing and CPA / tax-counsel review. It is not legal or tax advice.

## 1. Tax Year Cutoff, Property Regime & Core Scope Difference

- **Tax year cutoff:** Calendar-year individuals report events from `Jan 1 00:00:00` through `Dec 31 23:59:59` in the taxpayer's local timezone (or UTC consistently across the return — document the timezone convention explicitly and apply it uniformly to every transaction and lot).
- **Property regime (IRS Notice 2014-21; T.D. 10000):** Digital assets are treated as **property**, not currency. Every sale for fiat, crypto-for-crypto swap, payment for goods/services, or payment of blockchain gas in crypto is a **taxable disposition** under IRC § 1001.
- **No federal wealth tax — cost-basis lineage is critical:** Unlike wealth-tax jurisdictions, U.S. reporting depends on **complete lot-level acquisition history** (exact acquisition date, cost basis in USD, and wallet custody trail) for every unit disposed of during the tax year.
  - **Missing basis blocks:** If an asset disposed of during the tax year lacks verifiable acquisition cost basis (e.g., from unprovided prior-year exchange CSVs or missing earlier wallets), record a blocking item in `unknowns.json`. Never invent a basis, and never silently default to `$0` without explicit taxpayer sign-off (which overstates taxable gain).
- **Holding period (IRC § 1222):** Short-term = held **1 year or less** (taxed at ordinary rates); long-term = held **more than 1 year** (preferential capital-gains rates). The holding period begins the day **after** acquisition and includes the day of disposition.
- **Form 1040 Digital Asset Question:** Flag the mandatory Form 1040 yes/no question ("At any time during the tax year, did you receive or sell, exchange, or otherwise dispose of a digital asset?"). Buying crypto with USD and holding or transferring solely between own wallets answers "No"; receiving rewards/airdrops or executing any sale, swap, or crypto gas payment answers "Yes".

## 2. Mandatory Wallet-by-Wallet Cost Basis & Safe Harbors (2025+)

Under final Treasury Regulations (**T.D. 10000**, Treas. Reg. **§ 1.1012-1(j)**), effective **January 1, 2025**:

- **Universal pooling is ended:** Cost basis and lot selection MUST be tracked strictly **wallet-by-wallet / account-by-account**. Each exchange account and each self-custody wallet address maintains its own isolated lot ledger.
- **Self-transfers carry basis over:** Moving your own tokens between Wallet A and Wallet B (or between self-custody and your own exchange account) is **not** a taxable disposition. Transfer the exact lot(s) (original acquisition date and USD cost basis) from the sending wallet's ledger into the receiving wallet's ledger, matching by tx hash, timestamp, and amount.
- **Lot-selection method:**
  - **Default:** First-In, First-Out (**FIFO**) per wallet/account.
  - **Specific Identification (SpecID / HIFO / LIFO):** Permitted only if the taxpayer identifies the exact units disposed of **no later than the date and time of the disposition** via contemporaneous books and records (Treas. Reg. § 1.1012-1(j)(2)–(3)).
  - **Notice 2025-7 (2025 broker relief):** Provided temporary transitional relief for tax year **2025** allowing taxpayers to use their own contemporaneous books and records (rather than broker-side instructions) to specifically identify units held at custodial brokers while brokers built compliance tooling. Starting **January 1, 2026**, standing orders or contemporaneous instructions to the broker are required for broker-held assets.
- **Rev. Proc. 2024-28 (Jan 1, 2025 Safe Harbor):** For taxpayers transitioning from pre-2025 universal pooling to wallet-by-wallet accounting, Rev. Proc. 2024-28 provided a one-time irrevocable safe harbor (Specific Unit Allocation or Global Allocation) to assign unused pre-2025 basis to units held in each wallet as of January 1, 2025. Always ask during Phase 0 for the taxpayer's contemporaneous Rev. Proc. 2024-28 allocation schedule and carry those exact starting lots forward into 2025+.

## 3. Gas Fees & Digital Asset Transaction Costs (T.D. 10000)

Under final **Treas. Reg. § 1.1001-7** and **§ 1.1012-1(h)**:

1. **Paying network gas in crypto is a taxable disposition:** Spending ETH, SOL, POL, etc. as a transaction fee disposes of that gas lot for services at its USD fair market value at the transaction timestamp, generating a separate short-term or long-term capital gain/loss (`FMV_at_tx − basis_of_gas_lot`).
2. **100% allocation to the disposed asset on swaps and sales:** Digital asset transaction costs (exchange commissions, DEX fees, and blockchain gas fees) incurred on a sale or crypto-for-crypto exchange are allocated **100% to the disposed asset** — reducing the **amount realized** (proceeds) on the outgoing asset rather than splitting 50/50 or adding to the acquired asset's basis.
   - *Cash-for-crypto purchases:* Fees paid to buy crypto with fiat (USD) are added to the acquired lot's cost basis.
3. **Gas fees on non-taxable self-transfers:** The gas spent still triggers gain/loss on the disposed gas lot itself. Whether the USD value of that self-transfer gas fee can be added to the moved asset's basis or is a non-deductible personal/investment expense is not settled in final regulations — flag explicitly for CPA confirmation rather than silently capitalizing.

## 4. Form 1099-DA & Form 8949 Reconciliation

### Broker Reporting Phasing & DeFi Status
- **Custodial / centralized brokers (Form 1099-DA):**
  - **2025 dispositions:** Brokers must report **gross proceeds (Box 1f)**; cost basis (Box 1g) reporting is optional for 2025 (assets generally marked noncovered in Box 9).
  - **2026+ dispositions:** Mandatory basis reporting phases in for **covered** digital assets acquired on or after January 1, 2026 and held continuously at the same broker. Transferred-in assets remain noncovered (basis blank or unverified).
- **DeFi Broker Rule Repealed:** Treasury's non-custodial DeFi broker rule (**T.D. 10021**) was repealed and nullified on **April 10, 2025** under the Congressional Review Act (H.J. Res. 25). Non-custodial DeFi front-ends and smart contracts do **not** issue Form 1099-DA, **but taxpayers remain fully obligated to report all DeFi dispositions on Form 8949**.
- **Notice 2024-57:** Indefinitely defers *broker 1099-DA reporting* (not substantive taxpayer taxability) for wrapping/unwrapping, liquidity provider transactions, staking, digital asset lending, and short sales pending further IRS guidance.

### Mandatory Form 8949 Digital-Asset Checkboxes (2025+)
Starting with tax year 2025, digital asset dispositions use dedicated **Form 8949 Boxes G–L** (do **not** use stock/securities Boxes A–F):

| Holding Period | Form 1099-DA: Basis Reported to IRS | Form 1099-DA: Basis NOT Reported to IRS | NOT Reported on Form 1099-DA (Self-Custody / DeFi / Foreign) |
|---|---|---|---|
| **Part I — Short-Term (`<= 1 year`)** | **Box G** | **Box H** | **Box I** |
| **Part II — Long-Term (`> 1 year`)** | **Box J** | **Box K** | **Box L** |

### Reconciliation Discipline (Preventing CP2000 Mismatches)
1. Archive every Form 1099-DA received from brokers and tie **Box 1f gross proceeds** to Form 8949 **Column (d)** per broker and per box category.
2. When Box 1g (basis) is blank/noncovered (common in 2025 and for transferred-in lots), supply the verified historical basis from the taxpayer's per-wallet ledger in **Column (e)** under Box H or Box K.
3. When a broker reports an incorrect basis on a Box G/J covered transaction, report the broker's Box 1g figure in Column (e) and correct it in **Columns (f) and (g)** using Form 8949 adjustment code **`B`** (with supporting lot schedule attached).
4. Reconcile **stablecoin / qualifying NFT optional aggregate reporting** on Form 1099-DA carefully so aggregated lines are neither dropped nor double-counted against transaction CSV exports.

## 5. Ordinary Income & Deductions

Every receipt of ordinary income in crypto establishes **two** facts simultaneously: (1) ordinary income in USD equal to fair market value at receipt/dominion, and (2) the **initial cost basis** (and acquisition timestamp) for that new lot on subsequent disposition.

- **Staking rewards (Rev. Rul. 2023-14):** Under IRS published guidance, a cash-method taxpayer includes the USD FMV of staking rewards (validator consensus/execution payouts, delegated staking, exchange staking, rebasing `stETH`/`eETH` inflows) in gross income in the year the taxpayer gains **dominion and control** (the ability to sell, exchange, or otherwise dispose of the rewards).
  - *Note on contrary filing positions:* Taxpayers wishing to challenge immediate inclusion (e.g., created-property arguments raised in the *Jarrett* litigation) must consult tax counsel and consider protective disclosure on **Form 8275** to avoid accuracy-related penalties under IRC § 6662.
- **Airdrops & Hard Forks (Rev. Rul. 2019-24):**
  - A hard fork with **no** new cryptocurrency received is not taxable.
  - An airdrop (whether following a hard fork or as a protocol/retroactive distribution) is ordinary income at USD FMV when the taxpayer obtains **dominion and control**. Do **not** include worthless unsolicited spam/phishing tokens where no real market exists or no dominion is exercised.
- **Mining, Validation, DeFi Yield & Lending Interest:** Reportable as ordinary income at receipt FMV — **Schedule 1 (Line 8z, Other Income)** for passive/investment activity, or **Schedule C** (subject to self-employment tax, allowing ordinary and necessary business expense deductions under IRC § 162) if conducted as a regular, continuous trade or business.
- **DeFi Borrowing Interest (IRC § 163(d)):** Borrowing against crypto collateral (Aave, Maker, Morpho) is generally **not** a taxable sale (unless liquidated — liquidations are forced taxable dispositions!). Track accrued/paid borrowing interest separately: investment interest expense may be deductible up to net investment income on **Form 4952 / Schedule A** (only if itemizing), subject to tracing rules (Treas. Reg. § 1.163-8T). Never silently net borrowing interest against ordinary crypto income.
- **Capital Loss Limitation (IRC § 1211 / § 1212):** Net capital losses offset capital gains dollar-for-dollar, plus up to **$3,000/year** ($1,500 MFS) of ordinary income, with unused net capital losses carried forward indefinitely.

## 6. Recurring Open Legal Questions (Neutral Decision Framework)

U.S. federal law has significant unaddressed DeFi edge cases (several explicitly acknowledged as pending guidance in Notice 2024-57). Never silently guess — quantify **both** treatments in exact USD, apply the taxpayer/CPA's chosen rule consistently across all wallets and years, and present options neutrally in a Decision Memo:

1. **Wrapping / Unwrapping (`ETH ↔ WETH`, `stETH ↔ wstETH`) & Canonical Bridging:**
   - **Option A (Non-taxable change of form):** Same underlying economic asset, no material difference in legal entitlements under IRC § 1001 (*Cottage Savings*); cost basis and holding period carry over untouched.
   - **Option B (Taxable property-for-property exchange):** Distinct smart-contract token with different technical rights triggers gain/loss realization at swap FMV and resets holding period.
2. **Liquid Staking Deposits & Non-Rebasing LSTs (`ETH → rETH / wstETH / weETH`):**
   - **Option A (Taxable exchange on deposit; capital gain on redemption/sale):** Swapping `ETH` for `rETH`/`wstETH` realizes gain/loss on `ETH`; subsequent exchange-rate appreciation is realized as capital gain (long-term if held `> 1 year`) upon sale/redemption.
   - **Option B (Non-taxable deposit; yield component as ordinary income):** Treating staking receipt tokens as a custody receipt for staked ETH, with accrued staking yield recognized as ordinary income upon dominion/redemption.
3. **Liquidity Pools (Uniswap v2/v3, Curve, Balancer):**
   - **Option A (Taxable swap on entry and exit):** Depositing two tokens for an LP token/NFT realizes gain/loss on entry and exit (with collected fees taxed as ordinary income or realized on withdrawal depending on auto-compounding vs separate fee claims).
   - **Option B (Non-taxable deposit / partnership-style contribution):** Basis carries over into the LP position (rarely clean for active concentrated-liquidity rebalancing — flag for CPA).
4. **Worthless, Frozen, or Bankrupt Assets (`CCA 202302011`; IRC § 165):**
   - Under **IRS Chief Counsel Advice 202302011**, a token that has collapsed 99.99% but still trades anywhere on any exchange/DEX is **NOT** deductible as worthless or abandoned under IRC § 165 — realizing the loss requires an actual closed and completed disposition (sale/swap or documented irrevocable abandonment).
   - For **bankrupt exchanges or frozen platforms** (e.g., FTX, Celsius, BlockFi, Voyager): losses generally wait until the bankruptcy estate fixes recovery/distribution (closed transaction) unless a specific Ponzi/theft-loss safe harbor (**Rev. Proc. 2009-20** / IRC § 165(c)(2) profit-seeking theft analysis) applies — flag with exact claim documentation for CPA review rather than claiming an unsupported loss.
5. **Wash-Sale Rule (IRC § 1091):**
   - IRC § 1091 statutorily applies to "stock or securities". Because the IRS classifies spot digital assets as property, the 30-day wash-sale disallowance has historically **not** applied to spot crypto — **however**, verify whether Congress enacted digital-asset wash-sale legislation for the target tax year, check whether a specific token is a tokenized security, and note economic-substance risk (IRC § 7701(o)) on zero-risk instantaneous round-trips.

## 7. Foreign Account Reporting, Gifts & Charitable Donations (Mandatory Flags)

Whenever the audit uncovers any of the following, add a dedicated **Advisor Compliance Flag** to the Executive Summary:

- **FBAR (FinCEN Form 114):** Required (filed separately via BSA E-Filing, not with Form 1040) if aggregate maximum value of **foreign financial accounts** exceeded **$10,000** at any moment during the calendar year.
  - Under **FinCEN Notice 2020-2**, pure crypto-only accounts at foreign exchanges were historically deferred from FBAR reporting pending rulemaking (verify FinCEN's latest effective rule for the target year!), **but** foreign exchange accounts that also hold fiat currency (hybrid accounts) are generally treated as reportable financial accounts. Self-custody wallets are **not** foreign financial accounts.
- **FATCA (IRS Form 8938):** Filed with Form 1040 if specified foreign financial assets exceed statutory thresholds (starting at `$50,000` year-end / `$75,000` peak for single U.S.-resident filers; higher for MFJ and expats). Flag every **non-U.S. custodial exchange** discovered on-chain or via statements so counsel evaluates both FBAR and Form 8938.
- **Charitable Donations (Form 8283 & Qualified Appraisal):** Donating appreciated long-term crypto (`> 1 year`) directly to a 501(c)(3) avoids capital gains tax and may qualify for an FMV charitable deduction — **critical trap (IRS CCA 202304008):** any charitable crypto deduction **exceeding $5,000** requires a **qualified appraisal** by a qualified appraiser (exchange quotes alone do **not** satisfy IRC § 170(f)(11)(C)!).
- **Gifts (IRC § 1015 & Form 709):** Outgoing bona-fide gifts are non-taxable dispositions for the donor (flag Form 709 if total gifts to any single recipient exceed the year's annual gift-tax exclusion). For incoming gifts, record **both** the donor's original cost basis + holding period AND the FMV at gift date (required for the IRC § 1015 dual-basis loss rule if later sold below donor's basis).

## 8. U.S. Filing Package Table Layout

Produce clean, CPA-ready schedules generated programmatically in exact USD `Decimal` arithmetic from the per-wallet lot engine:

1. **Form 8949 Disposal Schedules** — Separate tables strictly grouped by **Box G, H, I** (Part I Short-Term) and **Box J, K, L** (Part II Long-Term), with official IRS columns:
   `(a) Description of property` | `(b) Date acquired` | `(c) Date sold or disposed of` | `(d) Proceeds (USD)` | `(e) Cost or other basis (USD)` | `(f) Code(s)` | `(g) Amount of adjustment (USD)` | `(h) Gain or (loss) (USD)`
   *(Provide a companion CSV/JSON supporting ledger carrying exact wallet/account ID, tx hash, token contract, quantity, fee allocation, and lot ID for audit defense.)*
2. **Schedule D Summary** — Clean roll-up of Short-Term totals (Boxes G, H, I), Long-Term totals (Boxes J, K, L), and combined net capital gain/loss.
3. **Form 1099-DA Reconciliation Table** — Broker by broker comparison of 1099-DA Box 1f proceeds and Box 1g basis against Form 8949 totals, documenting every adjustment code and noncovered basis supplement.
4. **Ordinary Income Schedule (Schedule 1 / Schedule C)** — Chronological table (`Date UTC`, `Category`, `Asset & Contract`, `Quantity`, `Unit FMV (USD)`, `Ordinary Income (USD)`, `Wallet / Account`, `Tx Hash`) with category subtotals (Staking, Airdrops, Mining/Validation, Lending/DeFi Interest).
5. **Deductible Borrowing Interest & Liquidation Schedule** — Separate table for Form 4952 / Schedule A review.
6. **Year-End Per-Wallet Inventory & Open Lots Roll-Forward (as of Dec 31)** — Exact remaining quantities and per-lot cost basis isolated by wallet/account so next year's wallet-by-wallet accounting starts from an audited baseline.
