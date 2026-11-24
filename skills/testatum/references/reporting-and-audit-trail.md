# Client Intake, Audience-Separated Reporting & Audit Trail

A technically perfect calculation fails in practice if the deliverables mix internal working notes with tax-office filings, leak client secrets, contain rounding mismatches, or swamp non-technical advisors in RPC logs.

## 1. Phase 0 Completeness Intake Checklist

Collect and record explicit answers (including explicit "none") for all five areas before closing scope:

1. **Self-custody wallets:** Every EVM and non-EVM address controlled by the taxpayer (hardware wallets, hot wallets, multisig/Safe signers, smart-contract wallets, old/archived wallets active in prior years).
2. **Bitcoin / UTXO extended public keys:** `xpub`/`ypub`/`zpub` or wallet descriptors for every HD Bitcoin/Litecoin wallet — explain clearly that a single BTC address rarely captures change and receive addresses.
3. **Custodial exchanges & platforms:** Every centralized exchange, broker, staking platform, or crypto bank account active during or before the tax year, with official year-end balance PDFs and full tax-year transaction/reward CSVs.
4. **Off-chain claims & private deals:** Any SAFTs, SPV/syndicate investments, OTC purchases, unpaid or off-chain reward spreadsheets, or creditor claims against bankrupt/frozen platforms.
5. **Prior-year tax treatment:** How previous tax returns handled recurring judgment calls (such as unclaimed rewards, staking-token yield, or unlisted stakes) so the current year stays consistent — or explicitly flags any deliberate change.

## 2. Audience Separation (Four Clean Deliverables)

Never dump every finding into a single cluttered document. Separate outputs cleanly by audience and purpose:

### Deliverable A — Tax-Office Filing Copy (`filing-statement`)
The clean statement intended to be attached directly to the tax return.
- **Final tone only:** Zero draft markers ("preliminary", "draft", "to be confirmed", "TODO", "pending user input"). Resolve or obtain written decisions on all blocking open items *before* generating the final filing copy.
- **Zero third-party noise:** No comparisons or diffs against CoinTracking, Koinly, Zerion, or prior drafts; no advisory commentary, no operational wallet tips, and no internal filenames or script paths.
- **No internal wallet columns (unless the jurisdiction requires per-wallet reporting, e.g. US Rev. Proc. 2024-28 per-wallet cost basis):** In wealth-focused filing summaries (such as Switzerland), present a clean **consolidated, alphabetical asset table** with columns:
  1. Asset / Position (clear name + verified identifier/chain where disambiguation helps)
  2. Quantity at Year-End
  3. Year-End Unit Rate (reporting currency) — **strictly one consistent unit price per identical asset**
  4. Rate Source (official security/Valor ID, verified market feed, or on-chain/cost basis)
  5. Year-End Wealth Value (reporting currency)
  6. Tax-Year Income (reporting currency, where applicable)
- Include separate, clearly titled sub-tables for **Liabilities / Debts & Deductible Borrowing Interest**, **Unlisted / Private Stakes**, and **Contingent / Zero-Valued Bankrupt Claims** (with registry citations) where applicable, followed by exact grand totals.
- Keep optional taxpayer identifying headers (name, address, tax ID) outside LLM context in `pii.local.json` and inject them only during local HTML/PDF rendering.

### Deliverable B — 1-Page Executive / Advisor Summary (`executive-summary`)
Designed for the taxpayer, tax lawyer, or fiduciary to read in 2 minutes:
- Headline totals (gross crypto wealth, liabilities, net crypto wealth, settled tax-year income, deductible borrowing interest, plus exact CHF/USD impact of any open legal options).
- Short numbered list of **action items required to close** (missing exchange statements, unconfirmed sibling wallets, pending legal classification choices).
- **Separate, concise prior-year / legal flag** whenever current-year findings reveal multi-year positions, unreported staking income, or professional-trading risk that counsel should review before filing.

### Deliverable C — Optional Client Operational Memo (`client-memo`)
Separate from the tax-office package. Include **only** genuinely worthwhile actionable findings (for example, forgotten claimable rewards, unclaimed old DEX balances, or undelegated restaked assets whose net value clearly exceeds gas, slippage, and effort), always noting any tax event triggered by claiming/withdrawing.
- **Do not** bloat this memo with generic security lectures, address-poisoning warnings, or spam-airdrop trivia.

### Deliverable D — Separate Audit Trail / *Prüfprotokoll* (`audit-trail`, 2–3 pages)
A concise, standalone verification document that a tax inspector or reviewing partner can trust without reading raw code:
- **Pages 1–2 (Plain language, jargon-free):** Emphasize **coverage and controls**, never vanity metrics (avoid "we ran 47,218 RPC calls!"). Clearly state:
  - Scope (tax year, exact UTC/local cutoff timestamp, number of verified self-custody wallets and custodial exchanges);
  - How completeness was established (multi-chain sweep, derived non-EVM checks, contract-interaction coverage, money-flow check for missing exchanges and sibling wallets);
  - Verification controls (two independent operators/methods per material balance, state-vs-history reconciliation, exact-arithmetic tie-out, independent verification pass);
  - Clear provenance breakdown (what is *chain-proven*, what is *third-party attested*, and what is *taxpayer-attested*);
  - Explicit disclosure of every **limitation or gap** (unreachable chains, single-source endpoints, pruned archives, pending confirmations).
- **Page 3 (Technical Appendix):** Compact reference table of active networks, exact cutoff block/ledger/round numbers, block hashes, timestamps, operator pairs used, valuation sources, archive manifest SHA-256 root/head hash, and local archive reproduction steps.

### Optional Deliverable E — Decision Memo & Tax-Office Disclosure Annex (`Beilage`)
When the case involves genuine legal ambiguities (for example non-rebasing staking token yield or unclaimed rewards timing):
- Provide a **1-page neutral decision memo** for the taxpayer/advisor laying out each option with exact amounts at stake, statutory/guidance basis, strongest counterargument, and procedural consequences — **without** inventing unsourced labels like "common practice".
- Once the taxpayer/advisor chooses an approach that warrants proactive transparency, prepare a brief, factual **disclosure annex** (*Beilage*) suitable to attach to the tax return so all material facts are disclosed cleanly on the record.

## 3. Standalone Package QA & Rendering Discipline

1. **Never overwrite delivered files:** During iterative reviews with a user or advisor, write revised drafts to new versioned filenames (or a new subfolder) while reviewing. When assembling the final handover folder, include **only** the clean final files with clean filenames.
2. **Zero revision chatter:** Scrub all final deliverables of phrases like *"New in v2"*, *"Updated from earlier draft"*, *"Fixed bug from yesterday"*, stale section cross-references, internal script names, or developer jargon. Every final document must read as if authored once, cleanly, for its target reader.
3. **Programmatic exact-arithmetic tie-out:** Never transcribe numbers by hand into Markdown or HTML tables. Generate every table and grand total directly from the canonical per-wallet JSON files using exact decimal arithmetic (`Decimal` / scaled `BigInt`), rounding consistently at the presentation layer and verifying that displayed line items sum cleanly to displayed subtotals and grand totals (explaining or reconciling any 1-centime/1-cent rounding variance explicitly rather than letting totals silently disagree).
4. **Visual PDF inspection (mandatory when producing PDFs):**
   - Generate PDFs cleanly (for example via headless Chromium:
     ```bash
     chrome --headless=new --disable-gpu --no-pdf-header-footer --print-to-pdf=output.pdf input.html
     ```
     using clean `@page` margins, `break-inside: avoid` on summary cards and table rows, and repeating `<thead>` headers across pages).
   - Convert every PDF page to PNG (e.g., via `pdftoppm -png -r 150` or `qlmanage` / `sips` / `pypdfium2`) and **visually inspect every single page** before delivering: check for clipped tables, awkward page splits, overflowing long hashes/addresses, orphaned headings, or misaligned numeric columns.
   - Verify no raw API keys, local machine paths, or unintended PII leaked into any rendered page.
