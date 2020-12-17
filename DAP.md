
Manta DAP Scheme Design
=======================

## Cryptographic Primitives
- Pseudorandom functions. For a seed $x$, we derive ${\sf PRF}_x^{addr}(\cdot)$, ${\sf PRF}_x^{sn}(\cdot)$, and ${\sf PRF}_x^{pk}(\cdot)$. Assume $\mathcal{H}$ be SHA256.
- ${\sf COMM}, a non-interactive commitment scheme that is both hiding and binding. Here, we use pedersen hash for commitment.


