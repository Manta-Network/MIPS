# About this spec

Verison 0.0.1. 
__WIP__.
All rights reserved.

# Terminology

## Cryptography
A __curve__ is an underlying elliptic curve that we build our cryptography on. Specifically, we will use the following curves:
* `Curve25519` which enables base coin account system;
* `JubJubCurve` which enables alter coin account systems, and many other cryptography primitives;
* `BLS12-381` which enables pairing and provides zero-knowledge proof sysgtem.

A __group__ is a prime subgroup of a curve, for which the discrete log problem is hard.
We may abuse the notion of group and curve when the content it clear.

## Base Coin

A __base_coin_address__, is a group element over the prime subgroup of `curve25519`.
It is mostly efficiently expressed in the form of 32 bytes Blob, i.e., `[u8; 32]`.
We may additionally create a mapping from this 32 bytes Blob to any string _NO SHORTER THAN_ 32 bytes, with, for example, error checking codes;
and store the new string instead. 
This is beyond the purpose of this spec, for this version.

This protocol does not have an upper bound of the number of accounts.

A __base_coin_value__, is a non-negative integer, within range `[0, MAX_BASE_COIN]`, that represents the number of (micro-)tokens that a given account may hold.
It is stored in an `u32` data type, with a fix-point represetation of `6` digits.
For example `123_456u32` represents a value of `0.123456` tokens; `123_456_789u32` represents a value of `123.456789` tokens.

`MAX_BASE_COIN` is the maxmimum number of tokens allowed by the protocol. It is currently set to `TBD`.

A __base_coin_account__, is a pair of the address and value, i.e.,

``` rust
struct BaseCoinAccount {
    address:    [u8; 32],
    value:      u32,
}
```

A __base_coin_secret_key__, also known as the __spending_key__, associated with a given account, is
a field element over `curve25519`. 
It is stored in the 
form of 32 bytes Blob, i.e., `[u8; 32]`.

A __base_coin_ledger__ is a list of active accounts, i.e., 
``` rust
type BaseCoinLedger = Vec<BaseCoinAccount>;
```

## Private Coin

A __private_coin_address__, is a group element over the prime subgroup of `JubJubCurve`.
It is mostly efficiently expressed in the form of 32 bytes Blob, i.e., `[u8; 32]`.
We may additionally create a mapping from this 32 bytes Blob to any string _NO SHORTER THAN_ 32 bytes, with, for example, error checking codes;
and store the new string instead. 
This is beyond the purpose of this spec, for this version.

This protocol does not have an upper bound of the number of accounts.

A __private_coin_value__, is a non-negative integer, within range `[0, MAX_PRIV_COIN]`, that represents the number of (micro-)tokens that a give account may hold.
It is stored in an `u32` data type, with a fix-point represetation of `6` digits.
For example `123_456u32` represents a value of `0.123456` tokens; `123_456_789u32` represents a value of `123.456789` tokens.

`MAX_PRIV_COIN` is the maxmimum number of tokens allowed by the protocol. It is currently set to `TBD`.

A __private_coin_account__, is a pair of the address and value, i.e.,

``` rust
struct PrivCoinAccount {
    address:    [u8; 32],
    value:      u32,
}
```

A __private_coin_commitment__ is a commitmemnt to a certein `PrivCoinAccount`.
It binds the account to the commiment, and hides the content of the comitment in the meantime.


A __private_coin_secret_key__, also known as the __spending_key__, associated with a given account, is
a field element over `JubJubCurve`. 
It is stored in the 
form of 32 bytes Blob, i.e., `[u8; 32]`.

A __private_coin_ledger__ is a list of active accounts, i.e., 
``` rust
type PrivCoinLedger = Vec<PrivCoinAccount>;
```


A __priv_coin_ledger_state__ is a snapshot of the current view of a `ledger`.
It is currently implemented via a merkle tree with Pedersen hash, over `JubJubCurve`.
The state is therefore the root of the merkle tree, 
represetned by a pair of curve elements, stored in `[u8; 64]`.
 
## Transactions
### Base coin transcation
A __base_coin_transaction__ allows one to transfer base coins from one account to another, 
it involves of 
a `base_coin_sender`, `base_coin_receiver`, and a valid `value`.
The transcation is valid, if 
* the `base_coin_transaction` is signed by `base_coin_sender.address`'s secret key;
* `0 <= value <= base_coin_sender.value - BaseCoinTxFee`.

``` rust
struct BaseCoinTx {
    sender: [u8; 32],       // a curve25519 address
    receiver: [u8; 32],     // a curve25519 address
    value: u32
    signature: [u8; 64],
}
```

### Private coin transcation
A __priv_coin_transaction__ allows one to transfer private coins from one account
`priv_coin_sender` to a receiver `priv_coin_receiver`, and put the balance reminder of the
sender to a new account `priv_coin_reminder`.
All three accounts are in the form of commitments. 
A transaction consists of the following fields:

``` rust
struct PrivCoinTx {
    old_ledger_state: PrivCoinLedgerState,
    sender: PedersenCommitment,
    sender_proof: MerkleProof,
    
    new_ledger_state: PrivCoinLedgerState,
    receiver: PedersenCommitment,
    receiver_proof: MerkleProof,
    reminder: PedersenCommitment,
    reminder_proof: MerkleProof,

    tran_proof: ZKProof,    
}
```

### Mint transaction
A __mint_transcation__ converts base coins into private coins.
It creates a `BaseCoinTx` that sends the base coins into the `pool`, i.e., `BaseCoinPool`;
it creates a `priv_coin_receiver`, and a proof that this receiver is included in the updated ledger.

``` rust
struct MintTx {
    sender: [u8; 32],       // a curve25519 address
    value: u32,
    signature: [u8; 64],

    old_ledger_state:   PrivCoinLedgerState,
    new_ledger_stae:    PrivCoinLedgerState,

    receiver: PedersenCommitment,
    receiver_proof: MerkleProof,
}
```
