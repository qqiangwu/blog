# ADT Requirements
+ insert(key, value)
+ delete(key) 
+ search(key) -> value

Goal: O(1) time operations. Ideally, direct access table is the best way to do this.

# Implementation via hash
1. Basic Idea: Hash a bigger universe to a limited domain or hash an item in an universe to another entity in a smaller finite space.
2. Steps:
    + Prehash: prehash an object to an integer
    + Hash: h: U -> {0, 1, 2, ..., m - 1}
3. Problem: Collision
    + Chaining
    + Open addressing
    
### Chaining
1. Assumption - Simple Uniform Hashing:  Each key is equally likely to be hashed to any slot of table, independent of where other keys are hashed.
2. Load factor a = n/m = expected # keys per slot = expected length of a chain
3. Performance: This implies that expected running time for search is Θ(1+α) — the 1 comes from applying
the hash function and random access to the slot whereas the α comes from searching the
list. This is equal to O(1) if α = O(1), i.e., m = Ω(n).

#### Some Details
+ Shrink
+ Table doubling
+ Karp-Rabin Algorithm: text matching

### Hash functions
I won't talk about the detail

1. Division method: h(k) = k mod m (m is prime)
    + division is slow
    + inconvenient to find a prime number
2. Multiplication Method: h(k) = [(a · k) mod 2 w ]  >> (w − r)
    + Multiplication and bit extraction are faster than division.
3. Universe hashing

### Open addressing
Another solution to dealing with collision. 

#### Key idea: Probe
+ h(key, trial-count) : U × {0,1,...,m − 1} → {0,1,...,m − 1} is a permutation of 0 .. m

#### Probing strategies
+ Linear Probing: h(k,i) = (h0(k) +i) mod m => clustering
+ Double Probing: h(k,i) = (h1(k) +i·h2(k)) => clustering
+ Uniform Hashing Assumption: Each key is equally likely to have any one of the m! permutations as its probe sequence

### Cryptographic Hashing
1. Def: 
    + A cryptographic hash function is a deterministic procedure that takes an arbitrary block of data and returns a fixed-size bit string.
    + The data to be encoded is often called the message, and the hash value is sometimes called the message digest or simply digest
2. App:
    + the hash value is not used as hash table index and typically as an identity.
    + the (cryptographic) hash value, such that an accidental or intentional change to the data will change the hash value
    + computer security applications
    + data integrity
    + Password storage
    + File modification detector
    + Digital signatures
3. Desirable Properties
    + One-Way
    + Collision-resistance
    + Target collision-resistance: weaker than CR
4. Impl: There have been many proposals for hash functions which are OW, CR and TCR. Some of these have been broken. MD-5, for example, has been shown to not be CR. There is a competition underway to determine SHA-3, which would be a Secure Hash Algorithm certified by NIST. Cryptographic hash functions are significantly more complex than those used in hash tables. You can think of a cryptographic hash as running a regular hash function many, many times with pseudo-random permutations interspersed.