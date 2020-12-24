
Manta DAP Scheme Design
=======================

## Cryptographic Primitives
- Pseudorandom functions. For a seed $x$, we derive ${\sf PRF}_x^{addr}(\cdot)$, ${\sf PRF}_x^{sn}(\cdot)$, and ${\sf PRF}_x^{pk}(\cdot)$. Assume $\mathcal{H}$ be SHA256. We define $\sf PRF$ as follows: 
$$ 
\begin{aligned}
\sf PRF^{addr}_x(z)=\mathcal{H}(x||00||z)\\
\sf PRF^{sn}_x(z)=\mathcal{H}(x||01||z)\\
\sf PRF^{pk}_x(z)=\mathcal{H}(x||10||z)
\end{aligned}
$$
- ${\sf COMM}$, a non-interactive commitment scheme that is both hiding and binding. We can divide ${\sf COMM}$ into two following patterm: 
$$ 
\begin{aligned}
\sf k:=\sf COMM_{r}(a_{pk}||\rho\\
\sf cm:=\sf COMM_{s}(v||k)
\end{aligned}
$$ Here, we use pedersen hash for commitment. We define $\sf COMM_r(a_{pk}||\rho)= \mathcal{H}(r||[\mathcal{H}(a_{pk}||\rho)]_{128})$, where $[\cdot]_{128}$ denotes that we cut the 256-bit string to 128-bit. Also $\sf cm$ could be computed by: $\sf cm:=\sf COMM_s(v||k)=\mathcal{H}(k||0^{192}||v)$
- Signature, for validity of the commitiment.
- Encrytption