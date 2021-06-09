Walllet / Address Management Spec


Mnemonics 

https://wiki.polkadot.network/docs/en/learn-accounts

Manta applications should support mnemonics from the BIP39 dictionary, and map mnemonics to secrets in the same way as Subkey and Polkadot-JS:

“””
Subkey and Polkadot-JS based wallets use the BIP39 dictionary for mnemonic generation, but use the entropy byte array to generate the private key, while full BIP39 wallets (like Ledger) use 2048 rounds of PBKDF2 on the mnemonic
“””

User should never have to save any information besides the mnemonic to access a wallet. 


Address Hierarchy

Manta applications should conform to the BIP 44 spec (“Multi-Account Hierarchy for Deterministic Wallets”) https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki
This is very standard, Polkadot ETH and most other significant coins do so.

Summary: We have some deterministic function CKDpriv((pk, i) → (pk_i) that computes child key #i  from a parent private key (from which we can also derive an address). The master private key is the root node of a tree-shaped wallet. “Derivation paths” represent the path to some leaf (pk) and look like this:
m / 44' / 0' / 0' / 0 / 0
Each part of the path provides some information. The above example indicates “the first non-change address in the primary bitcoin account.” 

Address Hierarchy (Manta token)

We will want to register our own coin type. E.g. if our coin type is 615, the key path to a manta wallet will look like.

 https://github.com/satoshilabs/slips/blob/master/slip-0044.md

In practice, our applications should only use a few standard paths. 


Address Hierarchy (private tokens on manta swap)



Address Derivation

[Question: Is one pk good for many addresses?, if so, maybe no need to derive child private keys. But if child key derivation is cheap, we may still want to do so anyway]

[Rephrase: Given a PK and infinite randomness, can we derive infinite shielded addresses?]

It’s not possible for us to do key derivation exactly as described by BIP 32, since it is specific to Bitcoin’s ECDSA crypto. Whatever derivation scheme we use, it should be secure, publicly documented, and deterministic.


Version Bytes

https://wiki.trezor.io/Version_bytes

Manta token addresses and private tokens on manta swap should register their own distinct version bytes.
