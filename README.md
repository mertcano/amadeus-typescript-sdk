# @amadeus-protocol/sdk

Official TypeScript/JavaScript SDK for Amadeus Protocol - Core utilities for serialization, cryptography, transaction building, and API client.

[![npm version](https://img.shields.io/npm/v/@amadeus-protocol/sdk)](https://www.npmjs.com/package/@amadeus-protocol/sdk)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.7+-blue.svg)](https://www.typescriptlang.org/)
[![Node.js](https://img.shields.io/badge/Node.js-%3E%3D20.0.0-brightgreen.svg)](https://nodejs.org/)
[![Bundle Size](https://img.shields.io/bundlephobia/minzip/@amadeus-protocol/sdk)](https://bundlephobia.com/package/@amadeus-protocol/sdk)

## Installation

```bash
npm install @amadeus-protocol/sdk
# or
yarn add @amadeus-protocol/sdk
# or
pnpm add @amadeus-protocol/sdk
# or
bun add @amadeus-protocol/sdk

ESM-only: this package is published as pure ESM ("type": "module"). Use Node.js 20+ with "type": "module" in your package.json, or any modern bundler (Vite, webpack, esbuild, Metro). For CommonJS consumers, run via tsx. See Troubleshooting for the 1.0.x ERR_MODULE_NOT_FOUND issue and upgrade path.

What's New in 1.3.1
Lossless Amount Conversions (toAtomicAmaString, fromAtomicAmaString): Convert arbitrary-precision amounts without binary float drift or Number.MAX_SAFE_INTEGER (~9,007,199 AMA) limitations.

Monotonic Nonce Generation: Eliminates same-millisecond transaction collisions and silent drops in the node's txpool.

BLS12-381 Scalar Normalization: Correctly normalizes 64-byte seeds to 32-byte secret scalars via reduce512To256LE.

Vault KDF Metadata & DoS Capping: Vault encryption payloads now persist KDF algorithm and iteration counts, with an upper bound cap (10,000,000) preventing thread-freezing CPU exhaustion on untrusted payloads.

See the CHANGELOG for full release history.

What's New in 1.2.0
Staking (LockupVault) — read an account's staking position in one call:
sdk.staking.getPosition(publicKey) returns every vault plus the rolled-up totals
(total staked, accrued yield, stake-weighted APY, maturity/unlock state, stake per validator)

contract.getPrefixEntries() — prefix reads that return raw [key, value] byte pairs.
/api/contract/get_prefix answers with a VecPack binary body, which getPrefix() mangles
by parsing it as text

client.postBinary() — POST returning raw bytes, for the binary endpoints

decodeContractState() fix — a prefix matching nothing comes back as an empty VecPack
list, not an empty map. It used to throw; it now decodes to []. This is the common case
for any account that has never staked

ChainStats — added the fields the node actually returns: total_locked, total_supply,
supply_computed_at_height, and validators (with per-validator commission_bps and stake)

Features
Canonical Serialization (VecPack): Deterministic encoding/decoding for cryptographic operations

Cryptographic Operations: BLS12-381 key generation, signing, and verification

Password-Based Encryption: Secure AES-GCM encryption with PBKDF2 key derivation for wallet data

Transaction Building: Create and sign Amadeus protocol transactions

Token Conversions: Convert between atomic units and human-readable amounts with exact string paths

Encoding Utilities: Base58 and Base64 encoding/decoding for addresses, keys, and binary data

Staking: Read LockupVault positions — totals, per-vault detail, and stake-weighted APY

API Client: Full-featured HTTP client for interacting with Amadeus nodes

Type Safety: Complete TypeScript definitions for all APIs

Zero Dependencies: Uses native fetch (no axios or other HTTP libraries)

Quick Start
Using the SDK
TypeScript
import { AmadeusSDK } from '@amadeus-protocol/sdk'

// Initialize SDK (uses default node URL if not specified)
const sdk = new AmadeusSDK({
	baseUrl: '[https://mainnet-rpc.ama.one/api](https://mainnet-rpc.ama.one/api)'
})

// Query chain
const tip = await sdk.chain.getTip()
console.log('Current height:', tip.entry.header.height)

// Query wallet balance
const balance = await sdk.wallet.getBalance('5Kd3N...', 'AMA')
console.log('Balance:', balance.balance.float)

// Submit transaction
const result = await sdk.transaction.submit(txPacked)
Usage
API Client
TypeScript
import { AmadeusSDK } from '@amadeus-protocol/sdk'

const sdk = new AmadeusSDK({
	baseUrl: '[https://mainnet-rpc.ama.one/api](https://mainnet-rpc.ama.one/api)',
	timeout: 30000 // Optional: custom timeout
})

// Chain API
const stats = await sdk.chain.getStats()
const tip = await sdk.chain.getTip()
const entry = await sdk.chain.getByHash('5Kd3N...')
const { txs, cursor } = await sdk.chain.getByFilter({ contract: 'Coin', function: 'transfer' })
const { kpi } = await sdk.chain.getKpi()

// Wallet API
const balance = await sdk.wallet.getBalance('5Kd3N...', 'AMA')
const allBalances = await sdk.wallet.getAllBalances('5Kd3N...')

// Transaction API
const result = await sdk.transaction.submit(txPacked)
const confirmed = await sdk.transaction.submitAndWait(txPacked)
const finalized = await sdk.transaction.submitAndWait(txPacked, { finalized: true })
const tx = await sdk.transaction.get('5Kd3N...')

// Contract API
const contractData = await sdk.contract.get(key)
const richlist = await sdk.contract.getRichlist()
const { success, result } = await sdk.contract.view({
	contract: 'LockupPrime',
	function: 'view_balance',
	args: ['my_vault']
})

// Epoch API
const scores = await sdk.epoch.getScore()
const emission = await sdk.epoch.getEmissionAddress('5Kd3N...')

// Peer API
const nodes = await sdk.peer.getNodes()
const trainers = await sdk.peer.getTrainers()

// Proof API
const validatorProof = await sdk.proof.getValidators(entryHash)
const stateProof = await sdk.proof.getContractStateProof(stateKey)
Key Generation
TypeScript
import { generateKeypair, derivePublicKeyFromSeedBase58 } from '@amadeus-protocol/sdk'

// Generate a new keypair
const keypair = generateKeypair()
console.log(keypair.publicKey) // Base58 public key
console.log(keypair.privateKey) // Base58 private key (seed)

// Derive public key from existing seed
const publicKey = derivePublicKeyFromSeedBase58(keypair.privateKey)
Transaction Building
Using TransactionBuilder Class (Recommended)
TypeScript
import { TransactionBuilder, fromBase58, toAtomicAmaString } from '@amadeus-protocol/sdk'

// Instance-based usage (convenient for multiple transactions)
const builder = new TransactionBuilder('5Kd3N...') // Base58 encoded seed

// Option 1: Build and sign in one step (convenience, lossless string amount)
const { txHash, txPacked } = builder.transfer({
	recipient: '5Kd3N...', // Base58 encoded recipient address
	amount: '10000000', // Supports arbitrary-precision amounts > 9M AMA
	symbol: 'AMA' // Token symbol
})

// Option 2: Build unsigned, then sign (more control)
const unsignedTx = builder.buildTransfer({
	recipient: '5Kd3N...',
	amount: 10.5,
	symbol: 'AMA'
})
const { txHash, txPacked } = builder.sign(unsignedTx)

// Option 3: Build custom transaction and sign
const unsignedTx = builder.build('Coin', 'transfer', [
	fromBase58('5Kd3N...'), // Recipient bytes
	toAtomicAmaString(10.5), // Amount in atomic units
	'AMA'
])
const { txHash, txPacked } = builder.sign(unsignedTx)

// Option 4: Build and sign custom transaction (convenience)
const { txHash, txPacked } = builder.buildAndSign('Coin', 'transfer', [
	fromBase58('5Kd3N...'),
	toAtomicAmaString(10.5),
	'AMA'
])
ABI-Driven (Lockup, LockupPrime, etc.)
TypeScript
import {
	TransactionBuilder,
	LOCKUP_PRIME_ABI,
	LOCKUP_ABI,
	toAtomicAmaString
} from '@amadeus-protocol/sdk'

const builder = new TransactionBuilder('5Kd3N...')

// LockupPrime — auto-typed methods derived from the ABI (lossless string)
builder.contract(LOCKUP_PRIME_ABI).lock({ amount: toAtomicAmaString(100), tier: '30d' })
builder.contract(LOCKUP_PRIME_ABI).unlock({ vaultIndex: '3' })
builder.contract(LOCKUP_PRIME_ABI).daily_checkin({ vaultIndex: '7' })

// Lockup
builder.contract(LOCKUP_ABI).unlock({ vaultIndex: '5' })
Token Conversions
TypeScript
import {
	toAtomicAma,
	toAtomicAmaString,
	fromAtomicAma,
	fromAtomicAmaString
} from '@amadeus-protocol/sdk'

// Numbers (bounded by ~9,007,199 AMA safe integer ceiling)
const atomic = toAtomicAma(1.5) // Returns 1500000000
const ama = fromAtomicAma(1500000000) // Returns 1.5

// Exact strings (arbitrary precision, no ceiling)
const exactAtomic = toAtomicAmaString('21000000') // Returns '21000000000000000'
const exactAma = fromAtomicAmaString('21000000000000000') // Returns '21000000'
Password-Based Encryption
TypeScript
import { encryptWithPassword, decryptWithPassword } from '@amadeus-protocol/sdk'

// Encrypt sensitive data (AES-GCM with PBKDF2)
const encrypted = await encryptWithPassword('sensitive wallet data', 'my-password')
// Returns: { encryptedData, iv, salt, kdf, iterations }

// Decrypt the data
const decrypted = await decryptWithPassword(encrypted, 'my-password')
API Reference
Conversion
toAtomicAma(ama: number | string): number - Convert AMA to atomic units (safe integer ceiling at ~9M AMA)

toAtomicAmaString(ama: number | string): string - Convert AMA to exact atomic-unit decimal string without ceiling

fromAtomicAma(atomicAma: number | string): number - Convert atomic units to AMA (safe integer ceiling at ~9M AMA)

fromAtomicAmaString(atomicAma: bigint | number | string): string - Convert atomic units to exact AMA decimal string without ceiling

Encryption
encryptWithPassword(plaintext: string, password: string, iterations?: number): Promise<EncryptedPayload> - Encrypt data with password (AES-GCM + PBKDF2, default 100,000 iterations)

decryptWithPassword(payload: EncryptedPayload, password: string): Promise<string> - Decrypt data with password

generateSalt(): Uint8Array - Generate random salt (16 bytes)

generateIV(): Uint8Array - Generate random IV (12 bytes)

deriveKey(password: string, salt: Uint8Array | ArrayBuffer, iterations?: number): Promise<CryptoKey> - Derive AES-GCM key from password

Testing
Bash
npm test
License
MIT


---

### 7. `CHANGELOG.md`

`CHANGELOG.md` dosyasının en üstüne eklenecek `1.3.1` sürüm notu:

```markdown
## [1.3.1] - 2026-10-04

### Added
- **`toAtomicAmaString(ama)`**: Exact decimal string conversion to atomic units without binary floating-point rounding or `Number.MAX_SAFE_INTEGER` (~9,007,199 AMA) limitations.
- **`fromAtomicAmaString(atomicAma)`**: Lossless conversion of arbitrary-precision atomic amounts (`bigint`, `string`, or `number`) into human-readable AMA decimal strings.
- **Vault KDF DoS Protection**: Enforced `MAX_PBKDF2_ITERATIONS = 10_000_000` upper bound check in `decryptWithPassword` to eliminate CPU exhaustion attacks via unauthenticated payload headers.

### Changed
- **Builder Wiring**: `buildCoinTransfer` in `src/contracts/coin.ts` and staking builders in `src/transaction-builder.ts` now utilize `toAtomicAmaString`, enabling transfers and delegations of arbitrary size.
- **Vault Parsing**: `parseLockupVaultData` and `parseVaultData` now use `fromAtomicAmaString` internally, ensuring vaults exceeding ~9M AMA are accurately resolved rather than returning `null`.
- **Monotonic Nonces**: `generateNonce()` now utilizes a monotonic counter across same-millisecond calls, preventing transaction hash collisions and txpool drops on the node.
- **KDF Metadata**: `encryptWithPassword` records `kdf` and `iterations` inside `EncryptedPayload`, while maintaining `100,000` as the default for mobile wallet interoperability.
8. src/index.ts ve docs/API.md Kontrolü
Projenin dışa açılan API yüzeyinde eksik kalmaması için iki küçük kontrolü yapmayı unutma:

src/index.ts: İçinde export * from './conversion' varsa fromAtomicAmaString otomatik dışa aktarılır. Eğer fonksiyonlar tek tek export ediliyorsa fromAtomicAmaString adını da export listesine ekle.

docs/API.md: Varsa, README.md'deki gibi toAtomicAmaString ve fromAtomicAmaString fonksiyon imzalarını ekle.
