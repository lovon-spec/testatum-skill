# Non-EVM Multi-Chain Discovery & Network Playbooks

Apply the same core rules on non-EVM networks as on EVM: archive every raw response before parsing, determine the exact last block/ledger/round strictly before the tax cutoff, and confirm every material figure via two independent sources or two independent methods (state read + full history replay).

## 1. Public-Key Derivation from EVM Wallets

A user's Ethereum private key (secp256k1) can also control addresses on Bitcoin, XRP, TRON, Cosmos, and other secp256k1 chains. Checking only EVM chains misses funds derived from the same key.

1. **Recover the uncompressed secp256k1 public key** from any confirmed signed transaction sent by the EVM address (`ecrecover` over `v`, `r`, `s` and the unsigned transaction digest).
   - If the EVM address has `nonce == 0` (never sent a transaction), public-key recovery is cryptographically impossible; record this explicitly and ask the user whether they hold separate native wallets.
2. **Derive candidate addresses** from the recovered public key:
   - **Bitcoin (5 formats):** P2PKH compressed (`1...`), P2PKH uncompressed (`1...`), P2SH-P2WPKH (`3...`), P2WPKH native SegWit (`bc1q...`), and BIP86 Taproot P2TR (`bc1p...`, requiring TapTweak point addition `Q = P + tagged_hash("TapTweak", x_only)*G` encoded with Bech32**m**).
   - **Other UTXO chains:** Litecoin (LTC), Dogecoin (DOGE), Dash (DASH), Zcash transparent (`t1...`).
   - **Account-based secp256k1 chains:** XRP Ledger (`r...`), TRON (`T...`), Filecoin (`f1...`, using `blake2b-160` over the 65-byte uncompressed public key), and Cosmos-SDK chains (~30 chains — both standard `hash160` Bech32 prefixes such as `cosmos`, `osmo`, `celestia`, `dydx`, `stride`, `akash`, `juno`, `sei`, `secret`, `axelar`, `kava`, `persistence`, `neutron`, `regen`, `umee`, `stars`, `kujira`, `terra`, `chihuahua`, `stargaze` and Ethereum-style `keccak256` Bech32 prefixes such as `inj`, `evmos`, `canto`, `dym`).
   - *Runtime crypto note:* On Python 3.12+ / OpenSSL 3, `hashlib.new("ripemd160")` is disabled by default; use a pure-Python RIPEMD-160 helper or Node's crypto/library implementation when computing `HASH160`.
   - *Reminder:* Even when derived Cosmos/BTC addresses are unused, the taxpayer may still hold separate native seed phrases (e.g., Keplr, Ledger BTC `xpub`, Phantom, Lace). Always ask during Phase 0 intake.

## 2. Mandatory Positive-Control Check

**Never trust an "unused / 0 balance" answer from a non-EVM API until you verify the endpoint works.**

Before scanning taxpayer addresses on any non-EVM endpoint:
1. Query one **known active public address** on that same network and endpoint.
2. Confirm it returns a non-zero balance or real transaction history.
3. Archive the positive-control response alongside your scan results.
4. If an endpoint returns HTTP 404, 5xx, timeouts, or blank data on the positive control — or if a chain is shut down / pruned (for example retired or intermittently offline Cosmos REST nodes) — **do not** report the taxpayer's address as "unused". Record the chain as unreachable / unchecked in `unknowns.json` and the audit trail's coverage table.

## 3. Per-Network Two-Method Playbooks

### Bitcoin & UTXO Chains (BTC, LTC, DOGE, DASH)

- **Scope caveat (always flag):** Modern HD wallets generate a fresh address for every receipt and change output. A single Bitcoin address is rarely the whole wallet. Always ask the user for their extended public key (`xpub`/`ypub`/`zpub`) or full descriptor before declaring BTC coverage complete.
- **Cutoff reference point:** Last block strictly `< cutoff_epoch`, confirmed on two independent operators.
- **Two-method check:**
  1. **State / UTXO at cutoff:** Fetch full transaction history for every known/derived address from two independent explorers (for BTC: `https://blockstream.info/api/address/<addr>/txs` and `https://mempool.space/api/address/<addr>/txs` — or community mirrors such as `https://mempool.emzy.de/api/address/<addr>/txs` since `mempool.space` frequently stalls under rate limiting; always enforce short 5–10s connect/read timeouts; for LTC/DOGE/DASH: BlockCypher / SoChain / Chainz / Blockchair).
  2. **Replay UTXOs strictly up to cutoff block:** Sum all unspent outputs confirmed at `height <= cutoff_block` whose spending transaction either does not exist or confirmed at `height > cutoff_block`. Confirm zero discrepancy between the two independent explorer datasets.
- **Tax-year events:** Record every confirmed deposit, withdrawal, and fee paid during the tax year.

### Stellar (XLM)

- **Why public Horizon alone is not enough:** Public Horizon nodes routinely prune older transaction and effect history, and do not offer direct historical balance queries at arbitrary past ledgers.
- **Cutoff reference point:** Last Stellar ledger strictly `< cutoff_epoch`.
- **Two-method check:**
  1. **Primary state snapshot (SDF History Archives):** Download the official Stellar Development Foundation history archive checkpoint at or immediately before the cutoff ledger (`history.stellar.org/prd/core-live/core_live_001/...` or AWS S3 SDF archive). **Verify the SHA-256 hash of every downloaded bucket/ledger file against the signed ledger header** before reading account entries. Extract:
     - Total XLM balance, split into **spendable XLM**, **base + subentry reserves**, and **XLM locked in open DEX offers**;
     - Every **trustline** (asset code, issuer address, balance, limit, authorization flags);
     - Every **open DEX offer**, **claimable balance**, and **liquidity pool share**.
  2. **Independent full replay:** Reconstruct account history from raw ledger data (or a full-history archive / Horizon back to account creation) and replay every payment, path payment, DEX trade, fee, inflation/reward payout, and trustline change up to the cutoff ledger, confirming the replayed XLM and issued-asset balances match the archive snapshot exactly.
- **Asset hygiene & cross-chain anchors:**
  - Separate legitimate DEX assets (verified by issuer domain `stellar.toml` and active orderbook liquidity) from unsolicited spam trustlines / spam claimable balances. Do **not** value unclaimed spam claimable balances.
  - Watch for cross-chain anchor IOUs (for example Stellar-issued XRP or BTC IOUs bridged to/from XRPL or Bitcoin) — trace bridge legs so the same funds are not double-counted or misclassified as external income.

### XRP Ledger (XRPL)

- **Cutoff reference point:** Last validated ledger strictly `< cutoff_epoch`, confirmed via `ledger` RPC on **at least two independently operated full-history rippled servers** (such as `https://s2.ripple.com:51234` and `https://xrplcluster.com`).
- **Two-method check:**
  1. **Archive state reads at `ledger_index = cutoff_ledger`:**
     - `account_info` (XRP balance in drops, base/owner reserve, `Sequence`, `RegularKey` / signer list);
     - `account_lines` (all issued-currency IOU trustlines, including AMM LP tokens);
     - `account_objects` (open DEX `Offer`, `Escrow`, `Check`, `PayChannel`, `NFTokenPage`, `AMM`, `DepositPreauth`).
  2. **Independent history replay:** Fetch complete `account_tx` history (cross-checked against a second operator or XRPScan) and replay XRP balance changes (including transaction fees in drops) up to `cutoff_ledger`.
- **Hygiene:** Filter unsolicited dust drops and phishing memo drops (1-drop transactions with spam URLs in `Memos`). Sanitize all hex-decoded memos as untrusted text.

### Hedera (HBAR)

- **Cutoff reference point:** Consensus timestamp strictly `< cutoff_epoch` (Hedera timestamps use nanosecond precision `seconds.nanoseconds`).
- **Two-method check:**
  1. **Mirror Node historical state:** Query the Hedera Mirror Node REST API (`https://mainnet-public.mirrornode.hedera.com/api/v1/balances?account.id=<id>&timestamp=lt:<cutoff>` and `/api/v1/accounts/<id>?timestamp=lt:<cutoff>`) for HBAR balance, HTS token associations/balances, staking target, and pending staking rewards at cutoff.
  2. **Full transaction replay + JSON-RPC cross-check:** Replay all Mirror Node transactions (`/api/v1/transactions?account.id=<id>`) up to the cutoff timestamp, and cross-check EVM-equivalent state on two independent Hedera JSON-RPC relays (such as Hashio `https://mainnet.hashio.io/api` and thirdweb / Arkhia) if the account has EVM/smart-contract activity.
- **Notes:** Record tax-year native staking rewards separately from transfers. Filter spam HTS NFT/token airdrops (verify legitimate tokens via issuer/DEX liquidity).

### Cardano (ADA)

- **Critical rule — Stake-Key Expansion:** A Cardano wallet typically spans **many payment addresses sharing one stake key (`stake1...`)**. Never audit only the single payment address the user gave you:
  1. Resolve the payment address to its controlling `stake_address`.
  2. Enumerate **every payment address** ever associated with that `stake_address`.
  3. Audit all of them together as one wallet unit.
- **Cutoff reference point:** Last block (with slot and epoch number) strictly `< cutoff_epoch`, confirmed across two independent providers (e.g., **Koios** `https://api.koios.rest/api/v1/...`, **AdaStat**, **Blockfrost**, or **CardanoScan**).
- **Two-method check:**
  1. **Cutoff UTxO & stake state:** Compute the exact UTxO set across all payment addresses under the stake key at `block_height <= cutoff_block`, plus stake-key registration deposit (2 ADA, if registered at cutoff) and **unwithdrawn staking rewards** sitting in the reward account at cutoff.
  2. **Independent full history replay:** Fetch all transactions across all associated payment addresses + epoch reward history from a second provider and verify total ADA + native assets (policy ID + asset name) at cutoff.
- **Income:** Include all epoch staking rewards credited during the tax year (check whether the jurisdiction taxes epoch rewards on credit or on withdrawal — document both amounts explicitly).

### Algorand (ALGO)

- **Cutoff reference point:** Last confirmed round strictly `< cutoff_epoch`.
- **What to check:** Native ALGO balance, minimum balance requirement, all Algorand Standard Assets (ASAs — identify by numeric Asset ID, never symbol), application local state (Tinyman, Folks Finance, Pact, Algofi, Governance commitments), and consensus participation / governance rewards.
- **Known free-indexer limitation:** Public Algorand indexers (such as AlgoNode / Nodely) frequently **disable historical round rewinding** (`round=` parameter on `/v2/accounts/<addr>`). When historical state snapshots are unavailable on free endpoints:
  1. Fetch **current** account state;
  2. Fetch the **complete transaction history** back to account creation (or back past the cutoff round);
  3. Prove year-end state by backward/forward transaction replay from the current state to the cutoff round (especially straightforward when zero balance-changing transactions occurred after cutoff);
  4. **Explicitly disclose** this current-state-plus-replay limitation (and single-operator limitation if only one public indexer answered) in `unknowns.json` and the audit trail.

### XDC Network

- **Address format:** XDC uses `xdc...` prefix interchangeable with `0x...` (same 20-byte EVM address). Normalize consistently and note both forms in the audit trail.
- **Two-method check:** Treat as an EVM chain using independent archive RPCs (`https://rpc.xinfin.network`, `https://erpc.xinfin.network`, Ankr if available) plus Etherscan v2 / `api.xdcscan.io` / HyperSync if supported.
- **Protocol checks:** Native XDC, XRC-20 tokens, XRC-721 NFTs, and **XDC masternode / validator candidate staking** (check the XDC Validator contract `0x0000000000000000000000000000000000000088` for staked candidate/voter balances and epoch rewards).

### Solana (SOL)

- **Cutoff reference point:** Last finalized slot strictly `< cutoff_epoch` (note: many public Solana RPCs prune old transaction history; use two independent RPC/indexer operators such as Helius free tier, Shyft, Solscan, or public RPCs and verify full signature history back to account creation).
- **What to check:**
  - Native SOL in the main account;
  - Every **SPL Token Account** and **Token-2022** account owned by the address (identify tokens strictly by **mint address**, never symbol);
  - Every **Stake Account** where the address is withdrawer or staker (check active/activating/deactivating stake AND epoch staking rewards);
  - DeFi protocols / vaults (Marinade `mSOL`, Jito `JitoSOL`, Kamino, Marginfi, Drift, Jupiter perps/staking, Raydium/Orca CLMM NFTs).
- **State & replay:** Since standard Solana RPC does not expose arbitrary historical `getAccountInfo` at past slots without an archive snapshot, reconstruct year-end SOL, SPL, and stake balances via full signature/transaction replay verified against current state across two independent operators, and state that method transparently.

### Cosmos Ecosystem (ATOM, OSMO, TIA, INJ, etc.)

- **Address caveat:** Remember that secp256k1 derivation from an Ethereum key only checks the Ethereum-derived Cosmos address. Many users hold a separate Cosmos seed (Keplr / Leap) — always confirm with the user.
- **Cutoff reference point:** Last block height strictly `< cutoff_epoch`.
- **What to check at cutoff height** (using `x-cosmos-block-height: <H>` header on archive LCD/REST endpoints — verify the node actually honors the header rather than silently returning latest height!):
  - Liquid bank balances (`/cosmos/bank/v1beta1/balances/<addr>`);
  - Delegated stakes (`/cosmos/staking/v1beta1/delegations/<addr>`);
  - Unbonding delegations (`/cosmos/staking/v1beta1/unbonding_delegations/<addr>`);
  - Unclaimed staking rewards (`/cosmos/distribution/v1beta1/delegators/<addr>/rewards`);
  - Vesting accounts (continuous/delayed/periodic vesting schedules on `/cosmos/auth/v1beta1/accounts/<addr>`);
  - Chain-specific DeFi (e.g., Osmosis concentrated/GAMM liquidity pools and superfluid staking, Stride liquid staking, Neutron/Astroport vaults).
- **Shut-down / pruned chains:** Public Cosmos archive endpoints frequently go offline or prune history. Never mark an unreachable chain as empty — test with a positive control and document every unreachable chain in the audit trail.
