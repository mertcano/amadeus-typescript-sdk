# @amadeus-protocol/sdk

Official TypeScript/JavaScript SDK for Amadeus Protocol - Core utilities for serialization, cryptography, transaction building, and API client.[cite: 76]

[![npm version](https://img.shields.io/npm/v/@amadeus-protocol/sdk)](https://www.npmjs.com/package/@amadeus-protocol/sdk)[cite: 76]
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)[cite: 76]
[![TypeScript](https://img.shields.io/badge/TypeScript-5.7+-blue.svg)](https://www.typescriptlang.org/)[cite: 76]
[![Node.js](https://img.shields.io/badge/Node.js-%3E%3D20.0.0-brightgreen.svg)](https://nodejs.org/)[cite: 76]
[![Bundle Size](https://img.shields.io/bundlephobia/minzip/@amadeus-protocol/sdk)](https://bundlephobia.com/package/@amadeus-protocol/sdk)[cite: 76]

## Installation

```bash
npm install @amadeus-protocol/sdk
# or
yarn add @amadeus-protocol/sdk
# or
pnpm add @amadeus-protocol/sdk
# or
bun add @amadeus-protocol/sdk
```[cite: 76]

> **ESM-only**: this package is published as pure ESM (`"type": "module"`).[cite: 76] Use Node.js 20+ with `"type": "module"` in your `package.json`, or any modern bundler (Vite, webpack, esbuild, Metro).[cite: 76] For CommonJS consumers, run via [`tsx`](https://github.com/privatenumber/tsx).[cite: 76] See [Troubleshooting](https://docs.ama.one/sdk/9.-troubleshooting.md) for the `1.0.x` `ERR_MODULE_NOT_FOUND` issue and upgrade path.[cite: 76]

## What's New in 1.3.1

- **Lossless Amount Conversions (`toAtomicAmaString`, `fromAtomicAmaString`)**: Convert arbitrary-precision amounts without binary float rounding or `Number.MAX_SAFE_INTEGER` (~9,007,199 AMA) limitations.
- **Monotonic Nonce Generation**: Eliminates same-millisecond transaction hash collisions and txpool drops on the node.
- **BLS12-381 Scalar Normalization**: Accurately reduces 64-byte private key seeds to 32-byte secret scalars via `reduce512To256LE`.
- **Vault KDF Metadata & DoS Capping**: Vault encryption payloads record KDF algorithm and iteration counts, with an upper bound cap (`10,000,000`) preventing thread-freezing CPU exhaustion on unauthenticated payloads.

See the [CHANGELOG](./CHANGELOG.md) for full release history.[cite: 76]

## What's New in 1.2.0

- **Staking (`LockupVault`)** — read an account's staking position in one call:
  `sdk.staking.getPosition(publicKey)` returns every vault plus the rolled-up totals
  (total staked, accrued yield, stake-weighted APY, maturity/unlock state, stake per validator)[cite: 76]
- **`contract.getPrefixEntries()`** — prefix reads that return raw `[key, value]` byte pairs.[cite: 76]
  `/api/contract/get_prefix` answers with a VecPack _binary_ body, which `getPrefix()` mangles
  by parsing it as text[cite: 76]
- **`client.postBinary()`** — POST returning raw bytes, for the binary endpoints[cite: 76]
- **`decodeContractState()` fix** — a prefix matching nothing comes back as an empty VecPack
  _list_, not an empty map.[cite: 76] It used to throw; it now decodes to `[]`.[cite: 76] This is the common case
  for any account that has never staked[cite: 76]
- **`ChainStats`** — added the fields the node actually returns: `total_locked`, `total_supply`,
  `supply_computed_at_height`, and `validators` (with per-validator `commission_bps` and stake)[cite: 76]

See the [CHANGELOG](./CHANGELOG.md) for full release history.[cite: 76]

## What's New in 1.1.0

- **`contract.view()`** — read-only contract execution[cite: 76]
- **`chain.getByFilter()`** / **`chain.getKpi()`** — filtered tx queries + protocol KPIs[cite: 76]
- **`proof.getContractStateProof()`** — merkle proofs for contract state[cite: 76]
- **`submitAndWait(txPacked, { finalized: true })`** — wait for finality instead of confirmation[cite: 76]
- **NFT contract** — `NFT_ABI`, `buildNftTransfer/Mint/CreateCollection`, `TransactionBuilder.nftTransfer/nftMint/nftCreateCollection`[cite: 76]
- **ESM fix** — published `dist/*.js` now resolves correctly under raw `node` (the `1.0.x` `ERR_MODULE_NOT_FOUND` bug)[cite: 76]

See the [CHANGELOG](./CHANGELOG.md) for full release history.[cite: 76]

## Features

- **Canonical Serialization (VecPack)**: Deterministic encoding/decoding for cryptographic operations[cite: 76]
- **Cryptographic Operations**: BLS12-381 key generation, signing, and verification[cite: 76]
- **Password-Based Encryption**: Secure AES-GCM encryption with PBKDF2 key derivation for wallet data[cite: 76]
- **Transaction Building**: Create and sign Amadeus protocol transactions[cite: 76]
- **Token Conversions**: Convert between atomic units and human-readable amounts with exact string paths[cite: 76]
- **Encoding Utilities**: Base58 and Base64 encoding/decoding for addresses, keys, and binary data[cite: 76]
- **Staking**: Read LockupVault positions — totals, per-vault detail, and stake-weighted APY[cite: 76]
- **API Client**: Full-featured HTTP client for interacting with Amadeus nodes[cite: 76]
- **Type Safety**: Complete TypeScript definitions for all APIs[cite: 76]
- **Zero Dependencies**: Uses native fetch (no axios or other HTTP libraries)[cite: 76]

## Quick Start

### Using the SDK

```typescript
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
```[cite: 76]

## Usage

### API Client

```typescript
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
```[cite: 76]

### Key Generation

```typescript
import { generateKeypair, derivePublicKeyFromSeedBase58 } from '@amadeus-protocol/sdk'

// Generate a new keypair
const keypair = generateKeypair()
console.log(keypair.publicKey) // Base58 public key
console.log(keypair.privateKey) // Base58 private key (seed)

// Derive public key from existing seed
const publicKey = derivePublicKeyFromSeedBase58(keypair.privateKey)
```[cite: 76]

### Transaction Building

#### Using TransactionBuilder Class (Recommended)

```typescript
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
```[cite: 76]

#### ABI-Driven (Lockup, LockupPrime, etc.)

The recommended pattern for built-in contracts.[cite: 76] Pass any ABI to `builder.contract(abi)` and get fully-typed function calls:[cite: 76]

```typescript
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
```[cite: 76]

#### NFT (transfer, mint, create_collection)

The `Nft` built-in contract has dedicated builder methods.[cite: 76] NFT amounts are integer counts, **not** AMA atomic units.[cite: 76]

```typescript
const builder = new TransactionBuilder(privateKey)

// Create a collection (caller becomes owner)
builder.nftCreateCollection({ collection: 'AGENTIC', soulbound: false })

// Mint tokens (collection owner only)
builder.nftMint({ recipient: '5Kd3N...', amount: 10, collection: 'AGENTIC', token: '1' })

// Transfer
builder.nftTransfer({ recipient: '5Kd3N...', amount: 1, collection: 'AGENTIC', token: '1' })
```[cite: 76]

Static variants: `TransactionBuilder.buildSignedNftTransfer/Mint/CreateCollection(input)` — each takes the same params plus `senderPrivkey`.[cite: 76]

### Signing Transactions

The SDK supports two patterns.[cite: 76] Pick whichever fits your workflow.[cite: 76]

#### Pattern 1 — Auto-signed (high-level, recommended)

The `TransactionBuilder` instance methods build **and** sign in one call.[cite: 76] Best for app code where you have the private key in hand.[cite: 76]

```typescript
import { TransactionBuilder, LOCKUP_PRIME_ABI, toAtomicAmaString } from '@amadeus-protocol/sdk'

const builder = new TransactionBuilder('5Kd3N...') // Base58 seed

// Coin transfer
const a = builder.transfer({ recipient: '5Kd3N...', amount: 10.5, symbol: 'AMA' })

// ABI-driven (any contract)
const b = builder.contract(LOCKUP_PRIME_ABI).lock({
	amount: toAtomicAmaString(100),
	tier: '30d'
})

// NFT
const c = builder.nftTransfer({
	recipient: '5Kd3N...',
	amount: 1,
	collection: 'AGENTIC',
	token: '1'
})

// All return { txHash, txPacked } ready for sdk.transaction.submit(txPacked)
```[cite: 76]

#### Pattern 2 — Manual: build a `ContractCall`, sign separately

Build a `ContractCall` with a standalone helper or `createContract(ABI)`, then sign it independently with `TransactionBuilder.signCall(privkey, call)`.[cite: 76] Useful when:[cite: 76]

- You want to inspect or log the call before signing[cite: 76]
- The signing key lives somewhere else (HSM, separate process, separate machine)[cite: 76]
- You want to batch-build and sign at the end[cite: 76]

```typescript
import {
	TransactionBuilder,
	createContract,
	LOCKUP_PRIME_ABI,
	buildCoinTransfer,
	buildNftTransfer,
	toAtomicAmaString
} from '@amadeus-protocol/sdk'

// Build a ContractCall — three ways:

// A. Standalone helper (Coin)
const callA = buildCoinTransfer({ recipient: '5Kd3N...', amount: 10.5, symbol: 'AMA' })

// B. Standalone helper (NFT)
const callB = buildNftTransfer({
	recipient: '5Kd3N...',
	amount: 1,
	collection: 'AGENTIC',
	token: '1'
})

// C. ABI-driven, any contract
const lockupPrime = createContract(LOCKUP_PRIME_ABI)
const callC = lockupPrime.lock({ amount: toAtomicAmaString(100), tier: '30d' })

// Inspect if you want
console.log('Will call:', callC.contract, callC.method, callC.args)

// Sign with any private key (no builder instance needed)
const { txHash, txPacked } = TransactionBuilder.signCall('5Kd3N...', callC)
```[cite: 76]

#### Build-unsigned-then-sign (debugging)

If you need to inspect the full unsigned transaction (nonce, signer, action) before signing:[cite: 76]

```typescript
const builder = new TransactionBuilder('5Kd3N...')

const unsigned = builder.buildTransfer({ recipient: '5Kd3N...', amount: 10.5, symbol: 'AMA' })
console.log('Nonce:', unsigned.tx.nonce)
console.log('Action:', unsigned.tx.action)

const { txHash, txPacked } = builder.sign(unsigned)
```[cite: 76]

#### Using Static Methods

```typescript
import {
	TransactionBuilder,
	fromBase58,
	toAtomicAmaString,
	getPublicKey,
	deriveSkAndSeed64FromBase58Seed
} from '@amadeus-protocol/sdk'

// Option 1: Build and sign transfer in one step (convenience)
const { txHash, txPacked } = TransactionBuilder.buildSignedTransfer({
	senderPrivkey: '5Kd3N...', // Base58 encoded seed
	recipient: '5Kd3N...', // Base58 encoded recipient address
	amount: 10.5, // Amount in human-readable format
	symbol: 'AMA' // Token symbol
})

// Option 2: Build unsigned transfer, then sign
const { seed64 } = deriveSkAndSeed64FromBase58Seed('5Kd3N...')
const signerPubKey = getPublicKey(seed64)

const unsignedTx = TransactionBuilder.buildTransfer(
	{ recipient: '5Kd3N...', amount: 10.5, symbol: 'AMA' },
	signerPubKey
)
const { txHash, txPacked } = TransactionBuilder.sign(unsignedTx, '5Kd3N...')

// Option 3: Build custom unsigned transaction, then sign
const unsignedTx = TransactionBuilder.build(
	signerPubKey,
	'Coin', // Contract name
	'transfer', // Method name
	[fromBase58('5Kd3N...'), toAtomicAmaString(10.5), 'AMA']
)
const { txHash, txPacked } = TransactionBuilder.sign(unsignedTx, '5Kd3N...')

// Option 4: Build and sign custom transaction (convenience)
const { seed64, sk } = deriveSkAndSeed64FromBase58Seed('5Kd3N...')
const signerPubKey = getPublicKey(seed64)

const { txHash, txPacked } = TransactionBuilder.buildAndSign(signerPubKey, sk, 'Coin', 'transfer', [
	fromBase58('5Kd3N...'),
	toAtomicAmaString(10.5),
	'AMA'
])
```[cite: 76]

### Serialization

```typescript
import { encode, decode } from '@amadeus-protocol/sdk'

// Encode data to canonical format
const data = {
	foo: 'bar',
	count: 42,
	items: [1, 2, 3]
}
const encoded = encode(data)

// Decode data from canonical format
const decoded = decode(encoded)
```[cite: 76]

### Token Conversions

```typescript
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
```[cite: 76]

### Mnemonics (BIP39)

```typescript
import { generateMnemonic, validateMnemonic, mnemonicToSeedBase58 } from '@amadeus-protocol/sdk'

// Generate a 12-word mnemonic
const mnemonic = generateMnemonic()

// Validate a mnemonic
if (validateMnemonic(mnemonic)) {
	// Derive a Base58 seed (compatible with TransactionBuilder)
	const seedBase58 = mnemonicToSeedBase58(mnemonic)
}
```[cite: 76]

### Encoding Utilities

```typescript
import { toBase58, fromBase58, uint8ArrayToBase64, base64ToUint8Array } from '@amadeus-protocol/sdk'

// Base58 encoding
const encoded = toBase58(new Uint8Array([1, 2, 3]))
const decoded = fromBase58('5Kd3N...')

// Base64 encoding
const base64 = uint8ArrayToBase64(new Uint8Array([1, 2, 3]))
const bytes = base64ToUint8Array(base64)
```[cite: 76]

### Password-Based Encryption

```typescript
import { encryptWithPassword, decryptWithPassword } from '@amadeus-protocol/sdk'

// Encrypt sensitive data (e.g., private keys) with PBKDF2 + AES-GCM
const encrypted = await encryptWithPassword('sensitive wallet data', 'my-password')
// Returns: { encryptedData, iv, salt, kdf, iterations }

// Decrypt the data
const decrypted = await decryptWithPassword(encrypted, 'my-password')
// Returns: 'sensitive wallet data'
```[cite: 76]

## API Reference

### Constants

- `AMADEUS_PUBLIC_KEY_BYTE_LENGTH`: Byte length of public key (48)[cite: 76]
- `AMADEUS_SEED_BYTE_LENGTH`: Byte length of seed (64)[cite: 76]
- `AMA_TOKEN_DECIMALS`: Number of decimal places (9)[cite: 76]
- `AMA_TOKEN_DECIMALS_MULTIPLIER`: Multiplier for conversions (10^9)[cite: 76]
- `AMA_TRANSFER_FEE`: Network transfer fee (0.02)[cite: 76]
- `EXPLORER_URL`: Default explorer URL[cite: 76]
- `NODE_API_URL`: Default node API URL[cite: 76]

### Serialization

- `encode(term: SerializableValue): Uint8Array` - Encode value to canonical format[cite: 76]
- `decode(bytes: Uint8Array | number[]): DecodedValue` - Decode from canonical format[cite: 76]

### Crypto

- `generateKeypair(): KeyPair` - Generate a new keypair[cite: 76]
- `generatePrivateKey(): Uint8Array` - Generate a random 64-byte seed[cite: 76]
- `getPublicKey(seed64: Uint8Array): Uint8Array` - Derive public key from seed[cite: 76]
- `derivePublicKeyFromSeedBase58(base58Seed: string): string` - Derive public key from Base58 seed[cite: 76]
- `deriveSkAndSeed64FromBase58Seed(base58Seed64: string)` - Derive secret key and seed[cite: 76]

### Mnemonics (BIP39)

- `generateMnemonic(): string` - Generate a 12-word BIP39 mnemonic[cite: 76]
- `validateMnemonic(mnemonic: string): boolean` - Validate a BIP39 mnemonic[cite: 76]
- `mnemonicToSeedBase58(mnemonic: string): string` - Derive a Base58 seed from a mnemonic[cite: 76]
- `encodeVaultSecret`, `decodeVaultSecret`, `detectInputType` - Vault helpers[cite: 76]

### Encoding

- `toBase58(buf: Uint8Array): string` - Encode bytes to Base58[cite: 76]
- `fromBase58(str: string): Uint8Array` - Decode Base58 to bytes[cite: 76]
- `uint8ArrayToBase64(bytes: Uint8Array): string` - Convert bytes to Base64[cite: 76]
- `base64ToUint8Array(base64: string): Uint8Array` - Convert Base64 to bytes[cite: 76]
- `arrayBufferToBase64(buffer: ArrayBuffer): string` - Convert ArrayBuffer to Base64[cite: 76]
- `base64ToArrayBuffer(base64: string): ArrayBuffer` - Convert Base64 to ArrayBuffer[cite: 76]
- `uint8ArrayToArrayBuffer(bytes: Uint8Array): ArrayBuffer` - Convert Uint8Array to ArrayBuffer[cite: 76]
- `arrayBufferToUint8Array(buffer: ArrayBuffer): Uint8Array` - Convert ArrayBuffer to Uint8Array[cite: 76]

### Conversion

- `toAtomicAma(ama: number | string): number` - Convert AMA to atomic units (safe integer ceiling at ~9M AMA)[cite: 74]
- `toAtomicAmaString(ama: number | string): string` - Convert AMA to exact atomic-unit decimal string without ceiling[cite: 74]
- `fromAtomicAma(atomicAma: number | string): number` - Convert atomic units to AMA (safe integer ceiling at ~9M AMA)[cite: 74]
- `fromAtomicAmaString(atomicAma: bigint | number | string): string` - Convert atomic units to exact AMA decimal string without ceiling[cite: 74]

### Encryption

- `encryptWithPassword(plaintext: string, password: string, iterations?: number): Promise<EncryptedPayload>` - Encrypt data with password (AES-GCM + PBKDF2, default 100,000 iterations)[cite: 74, 76]
- `decryptWithPassword(payload: EncryptedPayload, password: string): Promise<string>` - Decrypt data with password (supports recorded KDF metadata and enforces DoS upper-bound iterations)[cite: 74, 76]
- `generateSalt(): Uint8Array` - Generate random salt (16 bytes)[cite: 76]
- `generateIV(): Uint8Array` - Generate random IV (12 bytes)[cite: 76]
- `deriveKey(password: string, salt: Uint8Array | ArrayBuffer, iterations?: number): Promise<CryptoKey>` - Derive AES-GCM key from password (capped between 1,000 and 10,000,000 iterations)[cite: 74, 76]

### Transaction Building

- `TransactionBuilder` - Class for building and signing transactions[cite: 76]
    - **Constructor:** `new TransactionBuilder(privateKey?: string)` - Create a new builder instance[cite: 76]
    - **ABI-driven (recommended):**[cite: 76]
        - `contract(abi)` - Returns a typed, signer-bound contract interface; each ABI function becomes a method that builds and signs in one step[cite: 76]
    - **Generic instance methods:**[cite: 76]
        - `build(contract, method, args, signerPk?): UnsignedTransactionWithHash`[cite: 76]
        - `sign(unsignedTx, signerSk?): BuildTransactionResult`[cite: 76]
        - `buildAndSign(contract, method, args, signerPk?, signerSk?): BuildTransactionResult`[cite: 76]
        - `buildFromCall(call): UnsignedTransactionWithHash`[cite: 76]
        - `buildAndSignCall(call): BuildTransactionResult`[cite: 76]
    - **Coin transfer:**[cite: 76]
        - `buildTransfer(input, signerPk?): UnsignedTransactionWithHash`[cite: 76]
        - `transfer(input): BuildTransactionResult`[cite: 76]
    - **NFT (Nft contract):**[cite: 76]
        - `nftTransfer({ recipient, amount, collection, token }): BuildTransactionResult`[cite: 76]
        - `nftMint({ recipient, amount, collection, token }): BuildTransactionResult`[cite: 76]
        - `nftCreateCollection({ collection, soulbound? }): BuildTransactionResult`[cite: 76]
    - **Lockup / LockupPrime convenience methods:**[cite: 76]
        - `lockupUnlock({ vaultIndex }): BuildTransactionResult`[cite: 76]
        - `lockupPrimeLock({ amount, tier }): BuildTransactionResult`[cite: 76]
        - `lockupPrimeUnlock({ vaultIndex }): BuildTransactionResult`[cite: 76]
        - `lockupPrimeDailyCheckin({ vaultIndex }): BuildTransactionResult`[cite: 76]
    - **Static methods:** `signCall`, `buildFromCall`, `buildAndSignCall`, `buildSignedTransfer`, `buildSignedNft{Transfer,Mint,CreateCollection}`, `buildSignedLockup{Unlock}`, `buildSignedLockupPrime{Lock,Unlock,DailyCheckin}`[cite: 76]

### Contract ABIs

`as const` ABI definitions for built-in contracts.[cite: 76] Pass any ABI to `createContract(abi)` or `builder.contract(abi)` for fully-typed function calls.[cite: 76]

- `LOCKUP_ABI` - `Lockup` (vesting) — `unlock(vaultIndex)`[cite: 76]
- `LOCKUP_PRIME_ABI` - `LockupPrime` — `lock(amount, tier)`, `unlock(vaultIndex)`, `daily_checkin(vaultIndex)`[cite: 76]
- `NFT_ABI` - `Nft` — `transfer`, `mint`, `create_collection`[cite: 76]

Standalone builders that return a `ContractCall`:[cite: 76]

- `buildCoinTransfer({ recipient, amount, symbol })`[cite: 76]
- `buildNftTransfer({ recipient, amount, collection, token })`[cite: 76]
- `buildNftMint({ recipient, amount, collection, token })`[cite: 76]
- `buildNftCreateCollection({ collection, soulbound? })`[cite: 76]
- `createContract(abi).fn(params)` - generic ABI-driven builder[cite: 76]
- `buildContractCall(abi, fn, params)` - lower-level ABI-driven builder[cite: 76]

### Staking (LockupVault)

Amadeus staking is the `LockupVault` contract (`bic:lockup_vault:`): AMA locked for a tier duration,
backing a validator and earning a locked-in APY.[cite: 76] Not to be confused with `Lockup` (vesting) or
`LockupPrime` (points).[cite: 76]

```ts
const position = await sdk.staking.getPosition('5Kd3N...')

position.totalStaked // AMA staked = principal + yield compounded into the vaults
position.totalAccrued // yield compounded so far
position.weightedApyPercent // stake-weighted average APY
position.vaultCount // how many vaults
position.nextMatureEpoch // when the soonest lock expires
position.vaults // per-vault detail (tier, amount, APY, validator, maturity)
```[cite: 76]

- `sdk.staking.getPosition(pk, opts?)` — vaults plus rolled-up totals[cite: 76]
- `sdk.staking.getVaults(pk, opts?)` — just the vaults[cite: 76]
- `sdk.staking.getAllVaults(opts?)` — every vault on the chain (explorer-style views)[cite: 76]
- `sdk.staking.getValidatorCommissions(opts?)` — commission per validator, epoch-resolved[cite: 76]
- `sdk.staking.getCurrentEpoch()` — epoch from the chain tip[cite: 76]

Pass `{ currentEpoch }` across several calls so they resolve against one epoch and skip the extra
round trip.[cite: 76] The building blocks are exported too, for callers that own their own HTTP layer:
`buildOwnerVaultsKeyPrefix`, `decodeContractState`, `parseLockupVaultEntries`,
`summarizeLockupVaults`, `epochFromHeight`, `flatToAma`.[cite: 76]

Amounts are atomic `bigint` (`*Flat`) so nothing is lost to float rounding — real positions exceed
`2^53` atomic units.[cite: 76] Use `flatToAma()` or the pre-computed `number` fields.[cite: 76]

### Full API Reference

For complete documentation including request/response types, error handling, and end-to-end examples, see [docs.ama.one/sdk](https://docs.ama.one/sdk/1.-introduction.md).[cite: 76]

## Examples

See the [`examples/`](./examples/) directory for comprehensive usage examples:[cite: 76]

- **Basic Usage** - SDK initialization, key generation, queries[cite: 76]
- **Transaction Flow** - Complete transaction building and submission flow[cite: 76]
- **API Usage** - All API endpoints demonstrated[cite: 76]

```bash
# Run examples
npm run example:basic
npm run example:tx
npm run example:api
```[cite: 76]

## Testing

The SDK includes comprehensive test coverage:[cite: 76]

```bash
# Run tests
npm test

# Run tests in watch mode
npm run test:watch

# Run tests with coverage
npm run test:coverage
```[cite: 76]

## TypeScript Support

This package is written in TypeScript and includes full type definitions.[cite: 76] All exports are typed and documented.[cite: 76]

## Error Handling

All errors are thrown as `AmadeusSDKError` instances with descriptive messages:[cite: 76]

```typescript
import { AmadeusSDK, AmadeusSDKError } from '@amadeus-protocol/sdk'

try {
	const balance = await sdk.wallet.getBalance('invalid')
} catch (error) {
	if (error instanceof AmadeusSDKError) {
		console.error('SDK Error:', error.message)
		console.error('Status:', error.status)
		console.error('Response:', error.response)
	}
}
```[cite: 76]

## License

MIT[cite: 76]

## Contributing

Contributions are welcome![cite: 76] Please ensure all code follows the existing style and includes appropriate tests.[cite: 76]
