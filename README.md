Blockchain Technology
================================


MASS coin is a peer-to-peer currency that enables instant, near-zero cost
payments to anyone in the world. Mass Coin is a global payment network
that is fully decentralised over blockchain. Mathematically proven
cryptography secures the network and empowers individuals to control their
own finances with uncompromised security

  i. Proof of work
  =================

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

The proof-of-work also solves the problem of determining representation in
majority decision making. If a majority of CPU power is controlled by honest
nodes, the honest chain will grow the fastest and outpace any competing
chains.


ii. Scrypt hash
 =================
 
 When generating MASS coins, you hash a block header over and over again,
changing it slightly every time. Each iteration results in entirely different
hashes. A block header contains these fields: Version, hashPrevBlock,
hashMerkleRoot, Time, Bits, Nonce.

The body of the block contains the transactions; these are hashed only 

The Bits field is a representation of
the current target, using a special
floating-point encoding. This encoding
uses three bytes for the mantissa, and
the leading byte as a base-256
exponent. Only the 5 lowest bits are
used

The Nonce starts at 0 and is incremented for each hash. Whenever it
overflows, the extraNonce portion of the generation transaction is
incremented, which changes the Merkle root

Given just those fields, people would frequently generate the exact sequence
of hashes as each other and the fastest miner would almost always win.
However, it is [almost always] impossible for two people to have the same
Merkle root, because the first transaction in your block is a generation "sent"
to one of your unique MASS coin addresses. Since your block is different
from everyone else's blocks, you are [almost always] guaranteed to produce
different hashes. Every hash you calculate has the same chance of winning
as every other hash calculated by the network.

MASS coin uses scrypt (with parameters N=1024, r=1, p=1) for computing
the proof-of-work hashes which are checked against the target, and
SHA-256d (SHA-256 applied twice) for all other purposes. When
computing hashes, you need to be particularly careful about byte-order.
Note that the scrypt hash, which is a 256-bit number, has many leading zero
bits when stored or printed as a big-endian value (leading digits are the most
significant digits).
Block explorers usually display hashes in big-endian notation.
The following Python code will calculate the SHA-256d hash of Block
100000. The header is built from the six fields described above,
concatenated together as little-endian values in hex notation:

>>> import hashlib
>>> header_hex = ("01000000" +
... "ae178934851bfa0e83ccb6a3fc4bfddff3641e104b6c4680c31509074e699be2" +
... "bd672d8d2199ef37a59678f92443083e3b85edef8b45c71759371f823bab59a9" +
... "7126614f" +
... "44d5001d" +
... "45920180")
>>> header_bin = header_hex.decode('hex')
>>> hash = hashlib.sha256(hashlib.sha256(header_bin).digest()).digest()
>>> hash.encode('hex_codec')
'60ce4639bf63532b27e8f8b036b9846f5d2ae18556289f80e38b85a5df4910e1'
>>> hash[::-1].encode('hex_codec')
'e11049dfa5858be3809f285685e12a5d6f84b936b0f8e8272b5363bf3946ce60'

Alternatively, using the scrypt package:
===========

>>> import scrypt
>>> pow_hash = scrypt.hash(header_bin, header_bin, 1024, 1, 1, 32)
>>> pow_hash[::-1].encode('hex_codec')
'000000003b4ba52ab765631e20a04b88cd27f0b66d3509fb2da7781fae6d7901'

Note that the scrypt hash, which is a 256-bit number, has many leading zero
bits when stored or printed as a big-endian value (leading digits are the most
significant digits).

Block explorers usually display hashes in big-endian notation.

iii. Transaction
===============


In MASS coin transactions are a faster. This is due to the block processing
that is more efficient. The key difference for end-user being the 15 second
time to generate a block.

The faster block time of MASS coin reduces the risk of double spending
attacks in the case of both networks have the same hashing power.

MASS coin can handle a higher volume of transaction thanks to its faster
block generation.

iv. Traceability 
============

Once the latest transaction in a coin is buried under enough blocks, the
spent transactions before it can be discarded to save disk space. To
facilitate this without breaking the block's hash, transactions are hashed in a
Merkle Tree, with only the root included in the block's hash. Old blocks can
then be compacted by stubbing off branches of the tree. The interior hashes
do not need to be stored.

v. Mathematical proof
===

The race between the honest chain and an attacker chain can be
characterized as a Binomial Random Walk. The success event is the honest
chain being extended by one block, increasing its lead by +1, and the failure
event is the attacker's chain being extended by one block, reducing the gap
by -1.
The probability of an attacker catching up from a given deficit is analogous
to a Gambler's Ruin problem. Suppose a gambler with unlimited credit starts
at a deficit and plays potentially an infinite number of trials to try to reach
breakeven. We can calculate the probability he ever reaches breakeven, or
that an attacker ever catches up with the honest chain, as follows

p = probability an honest node finds the next block
q = probability the attacker finds the next block
qz = probability the attacker will ever catch up from z blocks behind
    
     | 1 if p<=q
qz = |
     | (q/p)^z  if p>q
     
Given our assumption that p > q, the probability drops exponentially as the
number of blocks the attacker has to catch up with increases. With the odds
against him, if he doesn't make a lucky lunge forward early on, his chances
become vanishingly small as he falls further behind.

We now consider how long the recipient of a new transaction needs to wait
before being sufficiently certain the sender can't change the transaction. We
assume the sender is an attacker who wants to make the recipient believe
he paid him for a while, then switch it to pay back to himself after some time
has passed. The receiver will be alerted when that happens, but the sender
hopes it will be too late.
Rearranging to avoid summing the infinite tail of the distribution...
The receiver generates a new key pair and gives the public key to the
sender shortly before signing. This prevents the sender from preparing a
chain of blocks ahead of time by working on it continuously until he is lucky
enough to get far enough ahead, then executing the transaction at that
moment. Once the transaction is sent, the dishonest sender starts working
in secret on a parallel chain containing an alternate version of his
transaction.
The recipient waits until the transaction has been added to a block and z
blocks have been linked after it. He doesn't know the exact amount of
progress the attacker has made, but assuming the honest blocks took the
average expected time per block, the attacker's potential progress will be a
Poisson distribution with expected value:

 λ = z(p/q)
 
 To get the probability the attacker could still catch up now, we multiply the Poisson density for
 each amount of progress he could have made by the probability he Rearranging to avoid summing the
 infinite tail of the distribution...
 
 Running some results, we can see the probability drop off exponentially with
z.
q=0.1
 z=0 P=1.0000000
 z=1 P=0.2045873
 z=2 P=0.0509779
 z=3 P=0.0131722
 z=4 P=0.0034552
 z=5 P=0.0009137
 z=6 P=0.0002428
 z=7 P=0.0000647
 z=8 P=0.0000173
 z=9 P=0.0000046
 z=10 P=0.0000012
q=0.3
 z=0 P=1.0000000
 z=5 P=0.1773523
 z=10 P=0.0416605
 z=15 P=0.0101008
 z=20 P=0.0024804
 z=25 P=0.0006132
 z=30 P=0.0001522
 z=35 P=0.0000379
 z=40 P=0.0000095
 z=45 P=0.0000024
 z=50 P=0.0000006
  Solving for P less than 0.1%...
P < 0.001
 q=0.10 z=5
 q=0.15 z=8
 q=0.20 z=11
 q=0.25 z=15
 q=0.30 z=24
 q=0.35 z=41
 q=0.40 z=89
 q=0.45 z=340
 
 vi. Network propagation
 ==========
 The steps to run the network are as follows:
1) New transactions are broadcast to all nodes by Gossip Protocol.
2) Every node collects new transactions into a block without any distinction.
3) Every node works on finding a difficult proof-of-work for its block and
 confirm transactions.
4) Successful Node which finds or mines the block, broadcasts the block to
 all nodes.
5) All other Nodes accept the block only if all transactions in it are valid and
 not already spent and also check the previous block hash is included.
6) Nodes express their acceptance of the block by working on creating the
 next block in the chain, using the hash of the accepted block as the
 previous hash.
 
