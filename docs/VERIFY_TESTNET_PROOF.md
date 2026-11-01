# Independently inspect the Hedera testnet proof

This walkthrough is for developers evaluating the public [Hedera HBAR Settlement Scaffold](../README.md). It uses read-only Hedera testnet Mirror Node endpoints, requires no wallet or API key, and does not transfer HBAR.

**What you can inspect:** whether the published EVM transaction succeeded, whether the claimed HCS topic message is present, and what the two independent API responses actually contain.

**What this alone does not prove:** a security audit, production readiness, mainnet operation, or a full cryptographic match between the EVM event's evidence commitment and the HCS payload. Those stronger claims require examining the contract events, the exact hashing scheme, and the local proof artifact.

## Published identifiers

| Item | Recorded value |
| --- | --- |
| Network | Hedera testnet, chain ID 296 |
| Contract | `0x2B45E553ad7e6e727aDb6D8918AF7b4b142e1bd2` |
| EVM transaction hash | `0x900e24a5817b928a9f2f92aceab6ae0c4f0cbe59fb1927ba4ce7c5fee92877c9` |
| HCS topic | `0.0.10639224` |
| HCS sequence number | `1` |
| Published settlement evidence hash | `0x4a6b77fba2515bb66144e03414ae8dace75589b7168d07bf4e481f6fa3de4184` |

Inspect these endpoints directly:

- [Mirror Node: contract result](https://testnet.mirrornode.hedera.com/api/v1/contracts/results/0x900e24a5817b928a9f2f92aceab6ae0c4f0cbe59fb1927ba4ce7c5fee92877c9)
- [Mirror Node: HCS topic message](https://testnet.mirrornode.hedera.com/api/v1/topics/0.0.10639224/messages/1)
- [HashScan: testnet settlement transaction](https://hashscan.io/testnet/transaction/0x900e24a5817b928a9f2f92aceab6ae0c4f0cbe59fb1927ba4ce7c5fee92877c9)

## Reproduce the read-only checks

With Node.js 20.18.3 or later, run the following from a Bash-compatible terminal. It uses only built-in Node.js modules.

```bash
node --input-type=module <<'NODE'
import assert from 'node:assert/strict';

const base = 'https://testnet.mirrornode.hedera.com';
const transactionHash = '0x900e24a5817b928a9f2f92aceab6ae0c4f0cbe59fb1927ba4ce7c5fee92877c9';
const topicId = '0.0.10639224';
const sequenceNumber = 1;

async function getJson(path) {
  const response = await fetch(new URL(path, base), {
    headers: { accept: 'application/json' },
    signal: AbortSignal.timeout(15000),
  });
  if (!response.ok) {
    throw new Error(`Mirror Node HTTP ${response.status}: ${path}`);
  }
  return response.json();
}

try {
  const [contract, hcs] = await Promise.all([
    getJson(`/api/v1/contracts/results/${transactionHash}`),
    getJson(`/api/v1/topics/${topicId}/messages/${sequenceNumber}`),
  ]);
  assert.equal(contract.result, 'SUCCESS', 'EVM result is not SUCCESS');
  assert.equal(contract.hash?.toLowerCase(), transactionHash.toLowerCase(), 'Transaction hash mismatch');
  assert.equal(hcs.topic_id, topicId, 'HCS topic mismatch');
  assert.equal(hcs.sequence_number, sequenceNumber, 'HCS sequence mismatch');
  assert.equal(typeof hcs.message, 'string', 'HCS message is missing');
  const payload = Buffer.from(hcs.message, 'base64').toString('utf8');
  console.log(JSON.stringify({
    network: 'hedera-testnet',
    transaction_hash: contract.hash,
    contract_result: contract.result,
    contract_id: contract.contract_id,
    hcs_topic: hcs.topic_id,
    hcs_sequence: hcs.sequence_number,
    hcs_consensus_timestamp: hcs.consensus_timestamp,
    hcs_message_utf8: payload,
    status: 'READ_ONLY_MIRROR_RECORDS_CHECKED',
  }, null, 2));
} catch (error) {
  console.error(`VERIFICATION_INCOMPLETE: ${error instanceof Error ? error.message : String(error)}`);
  process.exitCode = 1;
}
NODE
```

If an endpoint is unavailable, a field differs, or the request times out, the check is **inconclusive**, not evidence that a settlement failed. Retry after checking Mirror Node availability or inspect the published URLs in a browser. Do not report success unless the actual script completes successfully.

## Follow the implementation

1. **Authorize:** Inspect [OracleGuardedSettlement.sol](../packages/hardhat/contracts/OracleGuardedSettlement.sol) for EIP-712 signed intents, expiry, value binding, and replay prevention.
2. **Guard:** Inspect the same contract's Supra price-range and oracle-freshness checks; local tests use a mock feed.
3. **Settle:** Inspect the recorded contract result from the read-only Mirror Node endpoint above.
4. **Evidence:** Inspect [prove-testnet.ts](../packages/hardhat/scripts/prove-testnet.ts) to understand the evidence commitment, settlement digest, HCS submission, and verification flow.
5. **Qualify:** Run `npm install`, `npm run self-check`, `npm run lint`, `npm test`, and `npm run build` from a clean checkout. These are local checks and do not spend HBAR.

This is an inspectable **testnet engineering demonstration**. The transaction and HCS records are public references; correctness beyond the inspected fields must be established separately. For improvements, see [CONTRIBUTING.md](../CONTRIBUTING.md).
