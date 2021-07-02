# MIP1: Wallet and Address specification

Contents of this specification are provisional until launch of Manta mainnet.

## Addresses

Manta uses two kind of addresses

- _public addresses_: Substrate standard addresses with Manta's SS58 prefix (Manta's SS58 prefix will not be rolled out in the first stage of the testnet, i.e. testent alpha)
  _TODO: add the specific address types we support here_
- _shielded addresses_: Addresses that may receive private coins on Manta network, e.g. private DOT. Addresses are specific to coin type e.g. it is not possible to send private DOT to a private WETH address. Address re-use is a privacy leak.

Testnet addresses should also have distinct prefixes to prevent confusion between testnet and mainnet addresses.

## Wallets

### Background (BIP44)

https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki

BIP 44 wallet structure is a standard best practice for wallet design across many cryptocurrencies. ETH and DOT for example have registered BIP 44 coin types.

Summary: We have a deterministic key derivation function `KDFpriv: SK \times N → SK`. For example, `KDFPriv(sk, i) → sk_i` that computes child key #i from a parent secret key (from which we can also derive an address). The master secret key is the root node of a tree-shaped wallet. “Derivation paths” represent the path to some leaf (sk) and look like this:

`m / WALLET_PROTOCOL_ID=44' / ACCOUNT_NUMBER' / COIN_TYPE_ID' / (IS_CHANGE ? 0 : 1) / 0`

So for example: `m / 44' / 0' / 0' / 0 / 0` represents the first non-change Bitcon address/keypair in the first account of a BIP-44 wallet


### Manta and BIP 44

To avoid confusion between public coins and their private equivalents on Manta network, Manta will use a very slightly modified wallet schema. The third level of the keypath, i.e. `*'` in `m / MANTA_PURPOSE_ID' / *'` will indicate Manta network asset ID instead of BIP 44 coin IDs. (see https://github.com/bitcoin/bips/blob/master/bip-0043.mediawiki). 

The purpose ID corresponding to MIP1 is 3681947.

In all other respects, Manta's wallet protocol will be identical to BIP44


### Child Key Derivation

The nth hardened child of secret key s shall be **Blake2s(s || n)** where n is an unsigned 32 bit integer.

Note that this derivation algorithm differs both from BIP32 algorithm and the Substrate standard algorithm.

TODO: specify unhardened derivation


### Wallet integrations

TODO


### Mnemonics

https://wiki.polkadot.network/docs/en/learn-accounts

Manta wallets should support mnemonics from the BIP39 dictionary, and map mnemonics to secrets in the same way as Subkey and Polkadot-JS:

> Subkey and Polkadot-JS based wallets use the BIP39 dictionary for mnemonic generation, but use the entropy byte array to generate the private key, while full BIP39 wallets (like Ledger) use 2048 rounds of PBKDF2 on the mnemonic

### Account Recovery

1. A user enters a recovery phrase into a Manta wallet.
2. The wallet generates a master secret key from the mnemonic.
3. The wallet derives our Manta token address and simply asks polkadot.js for its balance.
4. The wallet derives the Manta private token account root key for each private asset.
5. For each private asset account, the wallet derives private token addresses starting from index 0, and checks if each address has ever received a UTXO. If the wallet sees N\* addresses in a row that have never received a UTXO, it stops checking.

\*"N" is commonly reffered to as the "gap limit," with 20 as a standard value
