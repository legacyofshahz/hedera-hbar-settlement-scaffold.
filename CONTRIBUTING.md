# Contributing to Hedera HBAR Settlement Scaffold

Thanks for helping improve this public, testnet-first reference implementation. Contributions that strengthen reproducibility, security tests, documentation, and developer usability are welcome.

## Good places to start

- Improve regression tests for EIP-712 signatures, expired intents, replay prevention, exact-value enforcement, and oracle freshness/range guards.
- Clarify quick-start instructions or improve non-secret failure diagnostics for clean installs.
- Improve the existing Next.js preflight demonstration while keeping settlement authorization deterministic.
- Review Mirror Node/HCS verification and suggest reproducible evidence checks using published testnet references.

Before starting substantial work, open or review an [issue](https://github.com/legacyofshahz/hedera-hbar-settlement-scaffold./issues) to describe the proposed improvement and avoid duplicate effort.

## Development and verification

1. Fork the repository and create a focused branch.
2. Use Node.js 20.18.3 or later and npm 10 or later.
3. From the repository root, run:

```bash
npm install
npm run self-check
npm run lint
npm test
npm run build
```

4. Open a pull request explaining the change, the checks run, and any limitations. For a guard or security-related change, include a regression test demonstrating the relevant behavior.

## Evidence and security

- The project is **Hedera testnet-first**. Do not describe a passing local test, a submitted transaction, or a testnet proof as production qualification.
- Keep `.env` files, private keys, account identifiers not already public, wallet exports, and credentials out of commits, issues, logs, and screenshots.
- Do not fabricate transaction IDs, HashScan links, Mirror Node responses, or HCS records.
- Share only publicly inspectable testnet identifiers and non-secret hashes. Do not copy raw sensitive business data into HCS.
- Do not weaken existing tests or enable mainnet execution as part of a documentation, onboarding, or UX change.
- For a potentially exploitable vulnerability, avoid publishing exploit details in a public issue. Seek an appropriate private disclosure channel with the maintainer first.

Review [AGENTS.md](./AGENTS.md) for the repository's security and proof invariants before modifying settlement behavior.

## License

By contributing, you agree that your contributions are made available under the repository's [MIT license](./LICENSE).
