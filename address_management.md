# Address / Wallet Spec

## Overview

Manta uses two kind of addresses

- _public addresses_: substrate standard addresses with Manta's SS58 prefix (Manta's SS58 prefix will not be rolled out in the first stage of the testnet, i.e. testent alpha)
  _TODO: add the specific address types we support here_
- _shielded addresses_: a one time, private address that uses for privitated coins

Testnet addresses should also have distinct prefixes to prevent confusion between testnet and mainnet addresses.

## Address Hierarchy

Manta wallets should conform to the BIP 44 spec (“Multi-Account Hierarchy for Deterministic Wallets”) https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki
This is a standard best practice for wallet design across all cryptocurrencies. ETH and DOT for example have registered BIP 44 coin types.

Summary: We have a deterministic key derivation function `KDFpriv: SK \times N → SK`. For example, `KDFPriv(sk, i) → sk_i` that computes child key #i from a parent secret key (from which we can also derive an address). The master secret key is the root node of a tree-shaped wallet. “Derivation paths” represent the path to some leaf (sk) and look like this:
m / 44' / 0' / 0' / 0 / 0
Each part of the path provides some information. The above example indicates “the first non-change address in the primary bitcoin account.”

## Address Hierarchy (Manta token)

https://github.com/satoshilabs/slips/blob/master/slip-0044.md

We should register our own coin type. If Manta token's coin type is (for example) 615, the key path to a manta wallet will be:

`m / 44' / 615' / 0'`

Since public Manta tokens are not UTXO-based, it should never be necessary to generate more than one address, or to generate a change address. So (following the above example), our 1 address will have this path:

`m / 44' / 615' / 0' / 0 / 0`

## Address Hierarchy (private tokens)

Private tokens on Manta network should have a single BIP 44 coin ID, different from the Manta token coin ID. Private DOT, private wrapped ETH, and private Manta will all have the same BIP 44 coin ID and the same account path. This is possible and convenient because all private tokens on Manta network have the same cryptographic backend.

Storing all private tokens in the same account provides two advantages. First, it allows us to follow BIP 44 without adding a new coin type for every different private asset on Manta network. Second, it makes recovering wallets from mnemonic is simpler and faster, because wallet software only needs to check one keypath.

## Mnemonics

https://wiki.polkadot.network/docs/en/learn-accounts

Manta wallets should support mnemonics from the BIP39 dictionary, and map mnemonics to secrets in the same way as Subkey and Polkadot-JS:

> Subkey and Polkadot-JS based wallets use the BIP39 dictionary for mnemonic generation, but use the entropy byte array to generate the private key, while full BIP39 wallets (like Ledger) use 2048 rounds of PBKDF2 on the mnemonic

## Account Recovery

1. A user enters a recovery phrase into a Manta wallet.
2. The wallet generates a master secret key from the mnemonic.
3. The wallet derives our Manta token address and simply asks polkadot.js for its balance.
4. The wallet derives the Manta private token account root key.
5. The wallet derives private token addresses starting from index 0, and checks if each address has ever received a UTXO. If the wallet sees N\* addresses in a row that have never received a UTXO, it stops checking.

\*"N" is commonly reffered to as the "gap limit," with 20 as a standard value
