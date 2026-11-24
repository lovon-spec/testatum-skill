# testatum

Evidence-grade crypto tax accounting as a verifiable AI agent skill.

`testatum` teaches a frontier AI agent to audit crypto wallets from primary blockchain state, verify every material figure from two independent sources, archive every raw response with SHA-256 hashes, trace every contract interaction and cross-chain flow, keep API keys and taxpayer PII out of chat via an ephemeral localhost form, and produce standalone, filing-ready tax reports and an audit trail (*Prüfprotokoll*).

## Layout

```text
skills/testatum/
├── LICENSE
├── SKILL.md
└── references/
    ├── providers.md
    ├── discovery-and-evm.md
    ├── non-evm.md
    ├── valuation-and-offchain.md
    ├── reporting-and-audit-trail.md
    └── jurisdictions/
        └── ch.md
```

The canonical skill tree for the Agent Skills Registry (Intendancy) is `skills/testatum/`.

## Install

Once registered on the Agent Skills Registry (Gnosis Chain), install with verified exact-bytes checks via `intend`:

```sh
intend update
intend install testatum --dir "<skills-dir>/testatum" --lockfile "$LOCK"
```

Or for local use before registration, copy `skills/testatum/` into your agent runtime's skills directory (for example `~/.claude/skills/testatum/`).

## License

MIT. See [skills/testatum/LICENSE](skills/testatum/LICENSE).
