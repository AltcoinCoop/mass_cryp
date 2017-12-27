Blockchain Technology
================================


MASS coin is a peer-to-peer currency that enables instant, near-zero cost
payments to anyone in the world. Mass Coin is a global payment network
that is fully decentralised over blockchain. Mathematically proven
cryptography secures the network and empowers individuals to control their
own finances with uncompromised security

*  i. Proof of work *

The proof-of-work involves finding for a value that when hashed with scrypt
hashing algorithm, the hash begins with a number of zero bits. The average work
required is exponential in the number of zero bits required and can be verified by
executing a single hash.

For our timestamp network, we implement the proof-of-work by incrementing a
nonce in the block until a value is found that gives the block's hash the required
zero bits. Once the CPU effort has been expended to make it satisfy the proofof-work,
the block cannot be changed without redoing the work. As later blocks
are chained after it, the work to change the block would include redoing all the
blocks after it.
