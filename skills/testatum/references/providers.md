# Data Providers, Free-Tier API Keys & Known Quirks

Always prefer primary blockchain nodes and official archives, backed by at least two independent operators. Guide the user to obtain free-tier keys where they unlock reliable historical state and multi-chain indexing, and rely on keyless public endpoints where keys are unavailable.

## Recommended Free-Tier API Keys

Collect these via the ephemeral localhost form (never in chat) into `<case-dir>/.env`:

| Env Var | Provider | What It Unlocks | Notes & Quirks |
|---|---|---|---|
| `ANKR_API_KEY` | Ankr | Archive EVM RPC (`https://rpc.ankr.com/<chain>/<key>`) across Ethereum, Arbitrum, Optimism, Base, Polygon, BSC, Avalanche, Fantom, Gnosis, etc.; plus Multichain Advanced API (`ankr_getAccountBalance`, `ankr_getTransactionsByAddress`, `ankr_getTokenTransfers`) | Very reliable archive state & cross-check. **Caveat:** `ankr_getAccountBalance` can return un-rebased balances for tokens like `stETH` — always verify holdings via `balanceOf` at the cutoff block. |
| `HYPERSYNC_API_KEY` | Envio HyperSync | Fast full-history queries (`https://<chainId>.hypersync.xyz/query`, Bearer token) across 50+ EVM chains | Essential for multi-chain token/tx/NFT discovery (including Gnosis and BSC where free Etherscan plans don't work). **Critical quirk:** run **sequentially** (1–3 chains at a time max); heavy concurrency causes silent empty response bodies. Always query ERC-1155 transfers both inbound AND outbound. |
| `ETHERSCAN_API_KEY` | Etherscan v2 | Unified v2 API (`txlist`, `tokentx`, `tokennfttx`, `token1155tx`, `txlistinternal`, `txsBeaconWithdrawal`, `getsourcecode`) | Free tier covers Ethereum mainnet reliably (~3 req/s; sleep 300–350 ms between calls). **Quirks:** (1) Free tier excludes several chains (such as Gnosis and BSC); (2) **Silently caps page size at 1,000 rows** even when `offset=10000` is requested — never stop paging early; stop only on an empty page or a verified block-window step. |
| `ALCHEMY_API_KEY` | Alchemy | Independent second-operator archive EVM RPC on enabled networks | Good second operator beside Ankr. Check which networks are enabled on the user's app before relying on it (disabled chains return 403). |
| `ZERION_API_KEY` | Zerion | Independent cross-check for current multi-chain portfolios and DeFi positions | Useful sanity check to catch forgotten dust/DeFi protocols. **Never a source of truth:** Zerion can misreport balances (e.g., wrong `eETH` counts) and misses Beacon Chain validators. |
| `COINGECKO_API_KEY` | CoinGecko (Demo / Pro) | Contract-address market prices and metadata | Keyless hits 429 after ~6 req/min. Free/Demo tier rejects history older than **365 days** (HTTP 401 / null) — for older dates, use official tax lists, Binance klines + FX, DefiLlama, or on-chain DEX reads. |

## Keyless Public Endpoints & Fallbacks

Verify live reachability and archive every response before relying on any public endpoint:

- **EVM Archive / State RPCs (independent keyless operators — verify live reachability and archive support before batching):**
  - Ethereum: `https://eth.drpc.org`, `https://rpc.mevblocker.io`, `https://mainnet.gateway.tenderly.co`
  - Gnosis: `https://rpc.gnosischain.com`, `https://rpc.gnosis.gateway.fm` (note: `gnosis.drpc.org` and `gnosis-rpc.publicnode.com` often refuse historical state reads)
  - Arbitrum: `https://arbitrum-one.public.blastapi.io`, `https://arbitrum.gateway.tenderly.co`, `https://arb-pokt.nodies.app` (**caveat:** `arb1.arbitrum.io/rpc` is **pruned**, not archive — use it only for `latest` or block headers, never historical cutoff state!)
  - Optimism: `https://mainnet.optimism.io`, `https://optimism.drpc.org`, `https://optimism.gateway.tenderly.co`
  - Base: `https://mainnet.base.org`, `https://base.drpc.org`, `https://base.gateway.tenderly.co`
  - Polygon: `https://polygon.drpc.org`, `https://polygon.gateway.tenderly.co`
  - Mantle / Fantom / PulseChain / ETHW: `https://rpc.mantle.xyz`, `https://rpcapi.fantom.network`, `https://rpc.pulsechain.com`, `https://mainnet.ethereumpow.org`
  - **PublicNode caveat:** `*.publicnode.com` endpoints serve `latest` and block headers keyless, but require a personal Allnodes token for historical archive queries (otherwise returning HTTP 403).
- **EVM Chain Discovery:** `https://chainid.network/chains.json` (1,500+ mainnets; filter out testnets/dead RPCs and enforce short connect+read timeouts such as 5–10s per request).
- **EVM Explorers (keyless):**
  - Blockscout v2 (`https://eth.blockscout.com/api/v2/...`, `optimism`, `celo`, `scroll`, `api.scan.pulsechain.com`): rich address tags and balances. **Quirks:** Arbitrum, Base, Polygon, and Gnosis Blockscout instances frequently sit behind Cloudflare 403 challenges; on `eth.blockscout.com`, `/addresses/<addr>/transactions?filter=from` can return HTTP 500 (query `/transactions` without `filter=from` instead), and Ethereum Blockscout embeds full tx input in rows (can exceed 250 MB per page on busy addresses) and rate-limits aggressively.
  - Routescan (`https://api.routescan.io/v2/network/mainnet/evm/<chainId>/etherscan/api`): Etherscan-compatible keyless fallback on supported chains (such as Avalanche `43114`, Mantle `5000`, Metis `1088`; does not cover Arbitrum/Optimism/Base).
- **Protocol-Specific Public APIs:**
  - Ethereum Beacon Chain: `https://ethereum-beacon-api.publicnode.com` (validator balances & withdrawals).
  - EigenLayer historical rewards: `https://sidecar-rpc.eigenlayer.xyz/mainnet/rewards/v1/earners/<addr>/summarized-rewards?blockHeight=<block>` and `/historical-rewards?startBlockHeight=<start>&endBlockHeight=<end>`.
  - Merkl rewards: `https://api.merkl.xyz/v4/users/<EIP55_CHECKSUM_ADDR>/rewards?chainId=<id>`.
  - Safe multisigs: `https://safe-transaction-<network>.safe.global/api/v1/owners/<EIP55_CHECKSUM_ADDR>/safes/` — **must** use EIP-55 checksummed address, follow HTTP redirects (`-L`), and use Safe's exact network slugs (`mainnet`, `gnosis-chain`, `arbitrum`, `optimism`, `base`, `polygon`).
  - Kleros Curate (on-chain Gnosis): Tokens `0xee1502e29795ef6c2d60f8d7120596abe3bad990`, Address Tags `0x66260c69d03837016d88c9877e61e08ef74c59f2`, Contract Domain Names `0x957a53a994860be4750810131d9c876b2f52d6e1`.
- **Prices & FX (keyless):**
  - Official Swiss rates: ESTV ICTax (see `references/jurisdictions/ch.md`).
  - DefiLlama: `https://coins.llama.fi/prices/historical/<epoch>/<chain>:<address>` and `/batchHistorical` (queries by contract address; absent for unknown/spam tokens; note DefiLlama's terms are personal/non-commercial — appropriate for individual self-audits, while commercial firms should use licensed feeds or on-chain/official sources).
  - Binance public daily klines: `https://data-api.binance.vision/api/v3/klines` (useful for historical daily USD receipt prices beyond CoinGecko's 365-day window).
  - ECB / Frankfurter FX: `https://api.frankfurter.app/<YYYY-MM-DD>?from=USD&to=CHF` (with prior business-day fallback for weekends/holidays).
  - DexScreener: useful for checking whether illiquid or fork tokens (such as PulseChain copies) actually have tradable pool depth.
- **Non-EVM public endpoints:** see `references/non-evm.md`.

## Operational Resilience & Shell Hygiene

1. **Per-capability node health:** A node is not a single boolean. One public node may serve old `eth_getBalance` and `eth_getCode` while refusing historical `eth_call`. Track failures per `(provider, method, block)` so a node that lacks archive state is skipped cleanly for that method rather than stalling the run.
2. **Distinguish throttling from refusal:** HTTP 429 or JSON-RPC rate-limit errors mean slow down (back off e.g. 5s → 15s → 30s), not "node lacks state" and never "contract reverted".
3. **Distinguish contract reverts from node errors:** Only treat a failed `eth_call` as a contract refusal when the error explicitly indicates EVM execution revert/halt (e.g., `execution reverted`, `invalid opcode`, `out of offset`, `stack underflow`), never on timeouts, rate limits, missing trie state, or node gas-cap errors.
4. **Batch sizing:** When batching JSON-RPC or Multicall3 calls, halve the batch size on size/gas rejection rather than dropping the node. Remember an `eth_call` without a target returns bytecode subject to the 24,576-byte limit.
5. **Respect anti-bot gates:** Never bypass CAPTCHAs or anti-bot challenges (such as Cloudflare Turnstile or drops.bot human checks). Note the blocked source as a limitation or let the user check it manually in their own browser.
6. **Shell gotchas:**
   - In `zsh`, never name a variable `path` (it overwrites `$PATH`), and wrap word-splitting commands in `bash -c`.
   - On some macOS/Linux setups, Python's `urllib` fails SSL verification due to missing local CA bundles; use `node` `fetch` or `curl` if Python TLS fails.
   - Always lowercase EVM addresses for comparisons and storage, except where an API explicitly requires EIP-55 checksummed casing (Safe Transaction Service, Merkl).
