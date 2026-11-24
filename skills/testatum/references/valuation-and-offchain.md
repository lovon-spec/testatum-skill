# Valuation, Custodial Statements & Off-Chain Forensics

Valuation must be deterministic, anchored to verified asset identity, and free of silent stand-ins or phantom liquidity. Every price, exchange statement, and off-chain fact requires archived evidence and an explicit provenance tier.

## 1. Identity-First Valuation Hierarchy

Never look up a price by bare ticker symbol. Always anchor every lookup to `(network, verified contract or mint/issuer address)` or an official tax-authority security ID mapped to that verified contract.

Apply this source order (top available tier wins; never mix conflicting unit prices for the same asset across sections of the same filing):

1. **Official tax-authority rate list** (where the taxpayer's jurisdiction publishes one — e.g., Switzerland's ESTV ICTax *Kursliste*). Always verify that the official security entry corresponds to the exact verified contract address and tax year, not merely a homonym ticker.
2. **Market-data services by contract address** (e.g., CoinGecko, DefiLlama, or Binance daily klines multiplied by official year-end/daily FX rates such as ECB/ESTV). Cross-check against a second independent market feed for material positions; investigate any divergence `> 2%`.
   - *Licensing note:* DefiLlama's free API terms are for personal/non-commercial use. Fine for personal self-audits; commercial advisors should use licensed market feeds, official lists, exchange klines, or primary on-chain DEX reads.
   - *Date-window note:* CoinGecko free/Demo keys reject historical dates older than 365 days; use official lists, Binance klines + FX, or primary on-chain DEX pools for older periods.
3. **Primary on-chain DEX pool reads at the exact cutoff block** (when no official rate or verified market feed exists). Subject to the strict liquidity and manipulation guards in Section 2 below.
4. **Verified protocol vault or issuer look-through** (for wrapped, liquid-staking, or ERC-4626 tokens with no deep direct market — e.g., `wstETH → stETH` via `getStETHByWstETH`, `ERC-4626` via `convertToAssets`, or issuer-published redemption rate), valuing the underlying via Tiers 1–3.
   - *Consistency warning:* If a jurisdiction treats a staking/vault token directly as a standalone capital asset rather than looking through to the underlying (see `references/jurisdictions/ch.md`), keep the quantity and unit price aligned (`token_qty × token_unit_price`, **never** `token_qty × underlying_price` or unwrapped quantity under a wrapped label).

If an identified, non-spam asset has **no reliable market, DEX liquidity, or verified redemption value**, do **not** invent a price or silently zero it out — open a valuation item in `unknowns.json` (and check whether the jurisdiction's fallback rule, such as acquisition cost for unlisted holdings, applies).

## 2. On-Chain DEX Pricing & Liquidity Sanity Guards

On-chain pool quotes are easy to misread or manipulate. Enforce every guard below before relying on a DEX price:

1. **Active tradable liquidity required:** On Uniswap v3 / concentrated-liquidity pools, verify `liquidity() > 0` at the quoted tick (and inspect depth across adjacent ticks). A pool with `sqrtPriceX96` set at an arbitrary price but zero active liquidity is not a market.
2. **Prefer TWAP over spot:** Read a 30-minute (or 1-hour) `observe()` TWAP ending at the cutoff block rather than single-block `slot0`, which is vulnerable to single-block sandwich/flash-loan distortion. Cross-check against the last swap event; flag a reservation if the last swap diverges `> 2%` from TWAP or is `> 24 hours` stale.
3. **Depth floor & concentration checks** (evaluated in the reporting currency at cutoff):
   - **Below depth floor** (e.g., pool TVL / side depth `< 1,000` CHF/USD): do **not** treat the quote as a reliable market price; flag as illiquid / no reliable market.
   - **Thin pool reservation** (depth `< 10,000` CHF/USD): quote may be recorded only with an explicit thin-liquidity note.
   - **Single-pool reservation:** Flag when only one pool prices a material position.
   - **Cross-pool divergence:** When multiple deep pools exist, compute the liquidity-weighted price; flag any divergence `> 2%` between pools.
   - **Portfolio concentration check (`> 10%` of pool depth):** Sum the taxpayer's **total exposure across all wallets and all forms** (direct holdings + wrapped forms + vault/LP look-through shares). If combined holdings exceed `10%` of available market depth, flag explicitly — exit slippage means the spot quote may overstate realizable fair market value (especially critical for fork tokens such as PulseChain copies and micro-cap governance tokens).

## 3. Wrappers, Bridges & Asset Equivalents

Never assume a bridged or wrapped token is 1:1 equivalent to its namesake without verifying the backing **on-chain for the specific tax year**:

- **Verify backing on-chain for the tax year:** Backing mechanisms change over time. For example, Gnosis Chain's native `xDAI` transitioned its Ethereum bridge backing from Maker `DAI` to Sky `sUSDS` in November 2025. Check the live bridge contract state at the cutoff block — do not rely on stale blog posts or prior-year assumptions.
- **Name every non-identical stand-in explicitly:** Whenever a position is valued via a bridge, wrapper, or conversion rate (`wraps`, `bridged`, `converts` — e.g., `WETH ↔ ETH`, `WXDAI ↔ xDAI`, `DAI ↔ USD`, `stETH ↔ ETH`), record it as an explicit valuation mapping with the exact conversion ratio, on-chain check, and amount at stake.
- **Check depegs and bridge insolvencies:** Confirm the bridge is solvent and open at cutoff (remember Multichain/Anyswap, Harmony Horizon, Nomad, Ronin, Wormhole historical incidents — a bridged asset on a broken bridge is **not** worth 1:1 with the canonical asset).

## 4. Contingent & Disputed Positions

If an asset's ownership or recovery depends on an unresolved dispute, arbitration, or litigation at the cutoff date (for example a disputed deposit in a Kleros Curate registry or a contested escrow):

- **Do not** include the disputed stake in confirmed year-end wealth totals.
- List it separately as a **contingent / open-outcome position** with the exact contract, dispute/item ID, asset quantity, and value at stake, and raise an open item in `unknowns.json` for advisor review.

## 5. Custodial Exchanges & Third-Party Statements

Self-custody balances confirmed on-chain are **chain-proven**. Custodial exchange balances cannot be independently verified on-chain because exchanges commingle customer funds — always label custodial figures as **third-party attested** (never *chain-proven*).

1. **Discover missing exchanges from on-chain flows:** During Phase 2/3, every EOA that receives funds from an owned wallet and immediately sweeps ~100% to a tagged exchange hot wallet identifies an exchange account. Cross-check that discovered list against the statements provided by the user. If on-chain flows reveal an exchange (e.g., Kraken, Binance, Coinbase, Bitfinex, Bitstamp, SwissBorg, Bity) with no statement provided, open a blocking item in `unknowns.json`.
2. **Reconcile deposits & withdrawals:** Match every on-chain deposit to and withdrawal from an exchange against the exchange's transaction CSV by asset, amount (allowing for withdrawal fee), UTC timestamp, and deposit address.
   - Flag unmatched deposits/withdrawals (often caused by another owned wallet, a missing sub-account, or an incomplete CSV export window).
   - Never count an exchange-to-self-custody transfer as taxable income or a disposal (unless tracking in-flight funds across the midnight cutoff — check timestamps on both sides of midnight so funds in transit at 23:59 are counted **once**, not zero times and not twice!).
3. **Verify year-end statement arithmetic:** Archive the raw exchange PDF/CSV with its SHA-256 hash. Recompute totals in exact arithmetic from the exchange's own transaction export where possible, and confirm exchange staking rewards, lending interest, and commissions/fees are separated cleanly.

## 6. Off-Chain Facts, Stuck Contracts & Private Deals Forensics

Real portfolios rarely live 100% in standard liquid ERC-20s. Apply forensic discipline to every non-standard asset:

### Declared Off-Chain Facts (e.g., Off-Chain Reward Spreadsheets)
- When a taxpayer provides an off-chain record (such as monthly off-chain staking/validator reward spreadsheets or OTC receipts), archive the original file with its SHA-256 hash, verify internal arithmetic programmatically, cross-check payouts against any on-chain receipt transactions, and mark the figures clearly as **taxpayer-attested / third-party-attested**.

### Stuck, Paused, or Unresponsive Contracts
When a taxpayer holds tokens or deposits in a contract that appears frozen, unclaimable, or abandoned, **do not guess** whether it is worth zero or full value:
1. **Inspect contract-wide history across *all* users:** Query recent transactions and events on the contract (not just the taxpayer's address) to see when withdrawals or rewards actually stopped for everyone. Distinguish an explorer/API logging cutoff from a real protocol halt.
2. **Simulate withdrawal (`eth_call`):** Test whether `withdraw()` / `claim()` succeeds or reverts (and with what exact error) from the taxpayer's address.
3. **Check public company & insolvency registers:** Look up the operating entity in official commercial and bankruptcy registries (e.g., Swiss Zefix / SHAB, UK Companies House, US PACER/Chapter 11 dockets, Singapore ACRA, Cayman Gazette) and official administrator/liquidator notices.
   - If the entity is formally in bankruptcy/liquidation with zero expected recovery or open creditor claim, document the exact registry notice/date and check jurisdiction valuation rules (e.g., CHF 0 with commercial-register citation; any later liquidation dividend treated as capital recovery up to cost).
   - If the protocol is merely illiquid, paused, or under governance restructuring, record the exact facts and put the valuation judgment in `unknowns.json`.

### Private Token Deals, SAFTs, SPVs & Syndicates
When on-chain outflows show large stablecoin/ETH payments to multisigs, syndicate wallets, or escrow contracts — or when the user discloses private allocations:
1. **Cross-reference primary records:** Match outgoing payment tx hashes, dates, and exact USD/CHF amounts against investor confirmations, SAFT/SPA agreements, SPV subscription docs, and distributor CSVs / Merkle proofs.
2. **Trace distributor & vesting wallets on-chain:** Check whether tokens have already TGE'd, partially vested, or been distributed to any of the taxpayer's wallets (including newly derived or secondary wallets) so a position is never double-counted as both an "off-chain SAFT" and an "on-chain token".
3. **Check public filings & tokenomics:** Verify issuer status, TGE status, lock-up/cliff schedules, and fund structures against public filings (e.g., SEC EDGAR Form D notices, official tokenomics documentation, commercial registers).
4. **Confirm with the taxpayer before filing:** Present every private-deal finding — exact cost basis paid (in reporting currency at payment date), TGE/vesting status at cutoff, and any public-record facts — for explicit written confirmation before finalizing valuation.
