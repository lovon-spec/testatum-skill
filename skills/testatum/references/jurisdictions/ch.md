# Jurisdiction Guide: Switzerland (Bund / DBG & StHG)

Apply this guide for taxpayers resident in Switzerland. Always verify whether the taxpayer's specific canton (e.g., ZH, ZG, GE, BS, BE, VD) has updated cantonal guidance for the target tax year, especially for unlisted stakes and professional-trading practice.

> **Scope note:** This guide documents Swiss statutory rules, official ESTV endpoints, and recurring classification questions so the accounting package is structured accurately for Swiss filing and advisor review. It is not legal or tax advice.

## 1. Cutoff, Wealth & Capital Gains Basics

- **Year-end cutoff:** `31.12.<year> 24:00 CET` (`Europe/Zurich`, UTC+1) = `<year>-12-31T23:00:00Z`. Take the last finalized block/ledger strictly **before** `23:00:00Z`.
- **Wealth tax (*Vermögenssteuer*):** Declare fair market value (*Verkehrswert*) of all cryptoassets at 31.12 in CHF, net of documented liabilities (*Schulden*, such as DeFi borrowing debt and margin loans).
- **Private capital gains (*Privatvermögen*):** Capital gains on private movable assets are tax-free (Art. 16 Abs. 3 DBG; Art. 7 Abs. 4 lit. b StHG), and capital losses are non-deductible — **provided** the taxpayer is not classified as a professional securities/crypto trader (*gewerbsmässiger Wertschriftenhandel*) under **ESTV Kreisschreiben Nr. 36 (KS 36)**.
  - **KS 36 portfolio-wide flag:** If the audit reveals heavy DeFi borrowing/leverage, high-frequency trading, significant short-term derivatives, or trading profits forming a large share of living income, add a concise separate advisor flag noting KS 36 safe-harbor criteria (6-month holding period, turnover `<= 5x` starting capital, capital gains `< 50%` of net income, no debt financing / leverage, derivatives only for hedging). Note that DeFi borrowing (e.g., Aave/Maker loans) touches the debt-financing criterion and should be evaluated portfolio-wide by tax counsel.

## 2. Official ESTV ICTax *Kursliste* Endpoints

For Swiss filings, the **ESTV ICTax *Kursliste*** is the Tier-1 official source for year-end CHF crypto rates and official FX rates. Always archive every raw response.

- **Identity mapping rule:** ESTV ICTax publishes `valorNumber` and `securityName` (for example `"USD Coin"`, `"Dai"`), **not** smart-contract addresses. Verify each on-chain holding's contract address against canonical issuer documentation or curated token lists *first*, then map only verified canonical tokens to their corresponding ESTV Valor entry — never map counterfeit contracts or homonym tickers.

### Session Handshake
ESTV ICTax API endpoints require session cookies (`JSESSIONID`, `cookiesession1`) and a CSRF token obtained via:
```text
GET https://www.ictax.admin.ch/extern/api/authentication/session.json
```
Extract the cookies and read the CSRF token from the JSON response body (`data.csrfToken`), then send `X-CSRF-TOKEN: <token>` alongside the session cookies on subsequent `POST` requests.

### Recommended Official Endpoints
1. **Yearly downloadable XML *Kursliste* archive (best primary source — complete & offline-reproducible):**
   - List available yearly XML exports: `POST https://www.ictax.admin.ch/extern/api/xml/xmls.json`
   - Select the target year's full XML v2.2.0 export (`exportType.shortName == "THIRD.INIT.220"`, plus any subsequent `DELTA` export for that year) and download using the returned `exportFile` metadata:
     ```text
     GET https://www.ictax.admin.ch/extern/api/download/<exportFile.id>/<exportFile.fileHash>/<exportFile.fileName>
     ```
   - Inside the XML:
     - **Official crypto rates** appear as `<currencyNote country="XV" securityType="CURRNOTE.TOKEN">` with `<yearend taxValueCHF="..."/>` and `valorNumber`. **Watch for `undefined="1"`** (used on delisted/migrated entries such as legacy Polygon Spot Valor `123828783`) — an undefined entry has no official ESTV rate and triggers the market-price fallback below.
     - **Official year-end FX rates** appear in the same XML under `<exchangeRateYearEnd currency="USD" value="..." valueMiddle="..."/>` (giving both exact 31.12 year-end FX and annual average FX).
   - Record the official **Valorennummer** in the filing table's Rate Source column whenever an ESTV rate is used.
2. **Official FX gadget (alternate JSON query):**
   - `POST https://www.ictax.admin.ch/extern/api/coreGadget/exchangeRates.json`
   - Use official ESTV year-end FX rates when converting verified USD/EUR market prices into CHF at 31.12, and ESTV monthly average / daily ECB rates for income receipt dates.
3. **Token gadget query (quick single-date lookup for major coins only):**
   - `POST https://www.ictax.admin.ch/extern/api/coreGadget/tokenGadget.json`
   - Required JSON payload: `{"page": 0, "size": 200, "referenceDate": <utc_midnight_epoch_ms>, "tokens": []}` (where `referenceDate` is 00:00:00 UTC in milliseconds, e.g., `2025-12-31T00:00:00Z`). Covers only a small subset of top coins — use the XML *Kursliste* above for full coverage.
4. **Security / *Valorennummer* search (secondary lookup only):**
   - `POST https://www.ictax.admin.ch/lsi/api/security` (`categoryId: 648`, `typeId: 5164`). **Caveat:** silently caps `total` at 50 results and returns neither year-end CHF prices nor contract addresses; prefer the XML *Kursliste*.

### ESTV Fallback Rule
When a legitimate cryptoasset is **not** listed in the ESTV *Kursliste* for that tax year, ESTV's *Arbeitspapier Kryptowährungen* (14.12.2021) prescribes:
1. Year-end market price on a common trading platform (cross-checked across two sources, converted to CHF via ESTV year-end FX);
2. If no reliable market price exists (e.g., unlisted pre-TGE allocation or illiquid private token), **original purchase price in CHF** (*ursprünglicher Kaufpreis*) at payment date.

## 3. Settled Taxable Income & Deductions

Based on **Art. 16 Abs. 1 / Art. 20 Abs. 1 DBG** and the **ESTV *Arbeitspapier Kryptowährungen* (14.12.2021)**:

- **Taxable income (*Vermögensertrag* / *übriges Einkommen*):**
  - Claimed staking rewards, liquidity-mining incentives, and DeFi protocol rewards (valued in CHF at receipt/credit date);
  - Rebasing staking token inflows (`stETH`, `eETH`, etc.) — value each rebase event/period at receipt-date CHF rate, reconciled so total rebased quantity equals the exact year-end minus start-of-year balance excluding transfers;
  - Ethereum Beacon Chain validator **consensus-layer withdrawals** AND **execution-layer fee / MEV payouts**;
  - Lending interest earned (Aave, Compound, Morpho, centralized lending);
  - Tradable airdrops received for services, staking, or protocol participation (exclude worthless spam airdrops).
- **Custodial exchange staking commissions:** Declare either net rewards (if the platform deducts commission before crediting and documents net payout) or gross rewards with the staking commission deducted as asset-management / collection costs (*Vermögensverwaltungskosten*), consistently across exchanges.
- **Deductible borrowing interest (*Schuldzinsen*, Art. 33 Abs. 1 lit. a DBG):** Interest accrued/paid during the tax year on DeFi borrowing positions (Aave variable/stable debt, Maker stability fees, Morpho, exchange margin debt) is deductible up to gross investment income plus CHF 50,000. Quantify exact tax-year interest paid/accrued per debt position and verify against debt-token balance growth and repayments.

## 4. Two Recurring Open Legal Questions (Neutral Decision Framework)

Never silently decide these two questions for the taxpayer, and **never** label one option as *"gängige Praxis"* without a binding primary citation. Quantify **both** options to the exact centime, apply the chosen rule consistently across **every wallet and every economically equivalent position** (and across tax years), and present them neutrally to the taxpayer / tax advisor:

### Open Question 1 — Unclaimed but Freely Claimable Rewards
Examples: unwithdrawn DeFi farming rewards, EigenLayer rewards accrued by 31.12 but not yet claimed, Cardano epoch rewards sitting in the stake reward account, Merkl/Votium claimable distributions earned by 31.12.

- **Wealth at 31.12:** Freely claimable rewards with an enforceable entitlement at 31.12 belong in **year-end wealth** (unless economically unrecoverable because gas exceeds reward value).
- **Income timing — two neutral options:**
  1. **Option A — *Ertrag bei Gutschrift / fester Rechtsanspruch* (accrual / fixed legal claim):** Include rewards earned and freely claimable by 31.12 in the current year's taxable income (at 31.12 or accrual-date CHF rate); later on-chain claiming is then a tax-neutral transfer.
  2. **Option B — *Ertrag bei Bezug* (realization / on-chain claim):** Report only rewards actually claimed/credited to the wallet during the tax year as income; unclaimed rewards are taxed in the year they are claimed (requiring careful tracking so they are neither missed nor double-taxed).

### Open Question 2 — Yield Accruing Inside Non-Rebasing Staking & Vault Tokens
Examples: `wstETH`, `rETH`, `weETH`, `cbETH`, `st-yCRV`, `sUSDS`, `ether.fi Liquid-v1`, restaked `stETH` / `eETH` where the token quantity stays constant while the underlying exchange rate appreciates.

- **Consistency rule (critical):** Treat **all** economically equivalent price-accruing staking/vault positions under the same rule — never tax `wstETH` as income solely because a rebasing twin (`stETH`) exists while treating `rETH` or `Liquid-v1` as capital gain.
- **Two neutral options:**
  1. **Option A — Declare the staking/vault token directly as itself (capital gain on appreciation):** Report the exact token quantity × per-token CHF price (e.g., `wstETH` quantity × `wstETH` CHF price — **never** unwrap into `stETH` quantity!) in wealth, with zero annual imputed income (treating exchange-rate appreciation as tax-free private capital gain under Art. 16 Abs. 3 DBG).
     - *Transparency safeguard:* Can be paired with a concise factual disclosure annex (*Beilage*) identifying the staking/vault tokens and their annual exchange-rate growth so the tax office has all facts on the table.
  2. **Option B — Impute annual exchange-rate yield growth as taxable income:** Compute the exact underlying yield accrued during the holding period (`shares × (rate_end − rate_start)` valued in CHF) as taxable investment income (by analogy to Art. 20 Abs. 1 lit. e DBG / accumulation-fund or availability principles).

### Procedural Stakes to Highlight in Any Decision Memo
Always include these exact statutory guardrails when presenting open options to the taxpayer or counsel:
- **Final assessment lock-in (Art. 147 DBG / Art. 51 StHG):** Once an assessment becomes final (*rechtskräftig*), tax voluntarily over-declared on an unsettled legal question generally **cannot** be reclaimed later even if case law or ESTV guidance subsequently clarifies the issue in the taxpayer's favor.
- **Back-tax reassessment (Art. 151–152 DBG / Art. 53 StHG):** A *Nachsteuer* (up to 10 years) requires **new facts or evidence** unknown to the tax authority at assessment. Full, transparent factual disclosure in the return or *Beilage* prevents a back-tax claim on an already-disclosed position.
- **Tax evasion & one-time voluntary disclosure (Art. 175 DBG):** Full factual disclosure removes any risk of a tax-evasion fine (*Steuerhinterziehung*). If review reveals unreported prior-year positions or income, remind the taxpayer/counsel of Switzerland's one-time penalty-free voluntary disclosure (*straflose Selbstanzeige*, Art. 175 Abs. 3 DBG) — which is only available **before** the tax authority learns of the facts independently.
- **Advance ruling (*Vorabauskunft*):** For material amounts under open legal questions, counsel can request a binding cantonal ruling before filing.

## 5. Unlisted Stakes, Private Deals & Bankrupt Claims

- **Pre-TGE SAFTs / private token deals:** Under the ESTV working paper, when no market price or deliverable token quantity exists at 31.12, value at **historical purchase price in CHF** at payment date (archive payment tx + agreement + FX rate).
- **Private SPVs / start-up shares:** Check cantonal wealth-tax directives (for example Zurich **ZStB 39.1** and the SSK *Kreisschreiben Nr. 28* for unlisted securities: start-ups during build-up are typically valued at net asset value / *Substanzwert*, with investor financing-round prices excluded; where a foreign SPV provides no NAV, acquisition cost in CHF is commonly used as a cautious documented stand-in). If a contractual lock-up applies and cantonal rules allow a lock-up discount, document the legal basis and discount explicitly rather than applying it silently.
- **Bankrupt / liquidated platforms or token issuers:** Value at **CHF 0** at 31.12 **only** with primary documentary proof (commercial register / SHAB entry, official bankruptcy decree, or administrator notice) cited directly in the table/footnote. Note that any later liquidation dividend up to original cost basis represents a tax-free capital recovery, not taxable income.

## 6. Prior-Year Exposure Check

Whenever the current-year on-chain audit reveals wallets, staking income, validators, DeFi holdings, or private deals that clearly existed in prior tax years, add a **separate, concise advisor flag** in the Executive Summary (never inside the tax-office filing copy!) recommending that the taxpayer and their advisor verify prior-year returns before filing the current return — because newly appearing multi-year assets can trigger tax-office inquiries (*Rückfragen*) about earlier periods.

## 7. Swiss Filing Table Format

Structure the tax-office filing copy (*Aufstellung Kryptowerte per 31.12.<Jahr>*) cleanly:
- **Main table (Kryptowerte / Vermögen & Ertrag):** Sorted **alphabetically by asset name**, with columns:
  `Position / Asset` | `Bestand per 31.12.` | `Steuerkurs (CHF)` | `Kursquelle / Valor` | `Steuerwert (CHF)` | `Ertrag <Jahr> (CHF)`
- Omit internal wallet addresses from the main consolidated tax-office table (keep wallet-level breakdowns in the supporting schedules / audit trail if requested by the tax office).
- Follow with dedicated sub-tables for **Schulden & Schuldzinsen (DeFi / Börsen)**, **Nicht kotierte Beteiligungen / Private Vereinbarungen**, and **Insolvente / wertlose Forderungen (CHF 0)**, ending with a clean reconciliation box showing Gross Wealth, Liabilities, Net Wealth, Total Taxable Income, and Deductible Borrowing Interest in exact CHF to the centime.
