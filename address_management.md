# Address / Wallet Spec

## Overview

Manta uses two kind of addresses

- _public addresses_: substrate standard addresses with Manta's SS58 prefix (Manta's SS58 prefix will not be rolled out in the first stage of the testnet, i.e. testent alpha)
  _TODO: add the specific address types we support here_
- _shielded addresses_: a one time, private address that uses for privitated coins

Testnet addresses should also have distinct prefixes to prevent confusion between testnet and mainnet addresses.

## Background (BIP 44)

https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki

BIP 44 wallet structure is a standard best practice for wallet design across many cryptocurrencies. ETH and DOT for example have registered BIP 44 coin types.

Summary: We have a deterministic key derivation function `KDFpriv: SK \times N → SK`. For example, `KDFPriv(sk, i) → sk_i` that computes child key #i from a parent secret key (from which we can also derive an address). The master secret key is the root node of a tree-shaped wallet. “Derivation paths” represent the path to some leaf (sk) and look like this:

`m / 44' / 0' / 0' / 0 / 0`

Each part of the path provides some information. The above example indicates “the first non-change address in the primary bitcoin account.”

## Manta and BIP 44

Manta network can support hundreds of different coins. These coins may or may not have their own BIP 44 IDs. However, Manta cannot use existing BIP 44 IDs for private tokens on manta network. Private DOT on manta network and regular DOT (for example) have very different cryptographic backends; conflating them could cause needless confusion.

Manta devs could register a BIP 44 ID for each private token on the network (see https://github.com/satoshilabs/slips/blob/master/slip-0044.md), but this will be an endless annoyance. Furthermore, Manta network already has its own asset ID system; it would be pointless and confusing to duplicate it.

Instead of BIP 44, Manta could use a very slightly modified wallet schema. The third level of the keypath, i.e. `*'` in `m / 44' / *'` will indicate Manta network asset ID instead of BIP 44 coin IDs. We can reseve `0` or `MAX_INT` for public Manta tokens as a special case. We can choose our own unique purpose field ID i.e. `*'` in `m / *'` to designate Manta network's address hierarchy (see https://github.com/bitcoin/bips/blob/master/bip-0043.mediawiki).

## Address Hierarchy (Manta token)

Since public Manta tokens are not UTXO-based, it should never be necessary to generate more than one address, or to generate a change address. So our 1 address will have this path:

`m / MANTA_PURPOSE_ID' / 0' / 0' / 0 / 0`

## Address Hierarchy (private tokens)

Private tokens on manta network will have keypaths in this form:

`m / MANTA_PURPOSE_ID' / MANTA_ASSET_ID' / 0' / {IS_CHANGE ? 1 : 0} / ADDRESS_NUMBER`

## Mnemonics

https://wiki.polkadot.network/docs/en/learn-accounts

Manta wallets should support mnemonics from the BIP39 dictionary, and map mnemonics to secrets in the same way as Subkey and Polkadot-JS:

> Subkey and Polkadot-JS based wallets use the BIP39 dictionary for mnemonic generation, but use the entropy byte array to generate the private key, while full BIP39 wallets (like Ledger) use 2048 rounds of PBKDF2 on the mnemonic

## Account Recovery

1. A user enters a recovery phrase into a Manta wallet.
2. The wallet generates a master secret key from the mnemonic.
3. The wallet derives our Manta token address and simply asks polkadot.js for its balance.
4. The wallet derives the Manta private token account root key for each private asset.
5. For each private asset account, the wallet derives private token addresses starting from index 0, and checks if each address has ever received a UTXO. If the wallet sees N\* addresses in a row that have never received a UTXO, it stops checking.

\*"N" is commonly reffered to as the "gap limit," with 20 as a standard value
