---
layout: post
title:  "Hash Functions survival guide"
date:   2018-10-20 16:03:00 +0200
tags: Tips
---

Hash functions are one of the building blocks behind robust solutions to solve architectural problems every developer will encounter in their career, like how to securely store passwords or how to distribute work across many servers.

To be honest, while their concepts are well known, they can also easily be misused.

This article hopes to shed some light on how to safely leverage them for some standard use-cases.

### A note on Terminology
Depending on your community or programming language of choice, you may know some constructs and applications under different terms.

This article will use the following nomenclature:

A _hash function_ refers to the implementation in which, given an arbitrary input, returns a deterministic output

This function implements a _hash algorithm_

A _hash_ (which you may know as a _digest_ or _hash code_) refers to the function’s output

![A hash functions implements a hash algorithm to generate a hash code]({{ "/assets/tips-hash-1.png" | absolute_url }})


## Do : know how a hash function work

A hash function is a deterministic function in which, given an arbitrary large input, returns a deterministic, fixed-size output. E.g, the [MurmurHash3 function](https://en.wikipedia.org/wiki/MurmurHash#Algorithm), in its 32 bits variant, will map any input to a 32 bit hash code :

| Input                  | Input size | Output     |
|------------------------|------------|------------|
| The "foo" ASCII string | 3B         | `0x1764df7f` |
| The entire [Hamlet](https://gist.github.com/provpup/2fc41686eab7400b796b) play | 188kB      | `0x2b881d54` |
| The [lenna](https://upload.wikimedia.org/wikipedia/en/7/7d/Lenna_%28test_image%29.png) test image   | 468kB      | `0x001fad14` |

This implies that an infinite input space is mapped to a finite number of outputs. Therefore, there must be some inputs which return the same hash code. These are known as **collisions**. Most use cases leverage the pseudo-uniqueness of hash codes, so you want to avoid them.

Since a function which always returns 42 can be considered a hash function under these terms, industrial-use hash algorithms are expected to meet the following criteria :

1.  **Uniformity** : Their output should be evenly distributed into the output space. E.g, still using the MurmurHash3 as an example :

    | Input (ASCII String) | Output (32bit unsigned int) |
    |-----------------------|-----------------------------|
    | foo                   | 4138058784                  |
    | fou                   | 2660153266                  |
    | boo                   | 389335901                   |
2. **Determinism** : Given an input, the hash function shall always return the same output

Each algorithm has a given length, which refers to the number of bits of the hash codes it generates. E.g, SHA-256 generate 256 bits hash codes, and murmur3 can be configured to generate 32 bits or 128 bits hash codes. Generally, short hashes are faster to generate, and lighter to manipulate, but have a greater collision risk.

## Do : know the difference between crypto and non-crypto hash algorithms

A subset of hash algorithms can be considered **cryptographic hash algorithms**, if they meet these additional criteria :
1. **Preimage resistance** : Given a hash code, it should be hard to generate a message having the same hash code
2. **Second-preimage resistance** : Given a message, it should be hard to find a second message having the same hash code
3. **Collision resistance** : It should be hard to find two distinct messages having the same hash code

By “hard”, we mean that brute forcing all inputs should be the fastest approach.

The **SHA-2** family are a widely-implemented and respected collection of cryptographic hash algorithms. When in doubt, they should not be a bad choice. MD5 and SHA-1 should be avoided, as their collision resistance has been broken by the advancement of computation speed.

Concerning **non-crypto hashes**, your language of choice should already have an existing implementation (e.g : ruby has Object#hash, based on the murmur2 algorithm). If that is the case, **trust it and use it**. More than the implementation, you want to leverage the years of optimization work on multiple hardware architectures.

If your language does not have a native implementation of a good non-crypto hash algorithm, you may use one of the following ones :

- Murmur3
- FNV Hash
- CityHash
- xxHash

## Don’t : roll your own implementations

This article will reference some well known and standard hash algorithms.
While these algorithm are open, you should avoid reimplementing them, and try to use existing implementations :

- Standard hash algorithms are likely to have been optimized down to the hardware level
- Crypto algorithms have been audited by people way smarter than you or me

If your platform does not come with native implementations, [libsodium](https://download.libsodium.org/doc/) is a very well made library which contains all the functions you may need.

## Don’t : use a crypto hash algorithm for non-sensitive uses

Crypto or non-crypto, every good hash function gives you a strong uniformity guarantee. Crypto hashes are however slower, and tend to generate larger codes (256 bits or more)

Using them to implement a bucketing strategy for 100 servers would be over-engineering. Just use a simple, fast, non-crypto algorithm for it.

## Do : use the right encoding for your use case

Hash functions typically return binary hash codes.

Depending on your use case, you should know which encoding is better suited for a use case.

For example, the hash code of the ASCII string "foo" can be encoded as :
- A **uint32** `4138058784` easy to modulo for bucketing
- A **hex code** `0xf6a5c420` for machine-to-machine exchange
- A **binary code** `0bx11110110101001011100010000100000` for storage
- A **Base64 code** `IMSl9g` for lightweight HTTP transfer

## Do : leverage hashes for bucketing

The most well known application of hash functions are **hash tables**. A hash table is a data structure mapping keys to value, enabling fast data lookup by partitioning its values into a select number of **buckets**. The hash code of the key can be transformed into a deterministic bucket index in constant time. You’re effectively trading a small constant hash computation cost for a 0(1) lookup complexity.

![Sample hash table algorithm]({{ "/assets/tips-hash-2.png" | absolute_url }})


The cool property of hash tables is that they ensure a good data repartition without any pre-existing knowledge of the data going into it. You can use the same approach for any **deterministic stateless pseudo-random attribution** : e.g balancing jobs on workers, payloads on servers, or riders on drivers.

## Don’t : use a naive bucketing strategy for a variable amount of buckets

A standard bucketing strategy consists of modulo-ing the data hash code with the number of buckets. As the hash function is uniform, this guarantees an eventual uniform repartition of resources between the buckets.

This strategy is not robust when faced with a varying amount of buckets, especially when migration costs are high, like in distributed hash tables. When having N buckets, adding or removing one means that almost all the data needs to be redistributed between the buckets.

Using a division instead of a modulo is better, but still implies that all buckets need to reallocate data.

The right solution for this is using [consistent hashing](https://en.wikipedia.org/wiki/Consistent_hashing). While this diminishes the uniformity of the data repartition, you gain the guarantee that adding / removing a bucket means that only one bucket needs to have its data migrated.

## Don’t : use a raw hash for message signing

Another common use of hash functions is for message signing. This can done using a hash-based message authentication code, or **HMAC**.

While a cryptographic hash function can assert the integrity of a message (i.e the message has not been tampered with during transfer), a HMAC also asserts its authenticity (i.e the messages comes from the sender). This mechanism is enabled via a shared secret key between the sender and the recipient.

![HMAC uses a key as a parameter for digest generation]({{ "/assets/tips-hash-3.png" | absolute_url }})


At first glance, transmitting the hash of a concatenation of the message and the secret key may seem to do the same job.

However, many hash algorithms, including SHA-2, are vulnerable against length extension attacks (meaning that - knowing the hash code and message length - it is possible to generate data which when appended to the message will keep the hash code valid).

The HMAC algorithm provides immunity against this kind of attacks. Its design makes it secure even when used with insecure hash functions (MD5 or SHA-1).

In short, when you need a stateless way to validate that a payload comes from a trusted sender, and has not been tampered with, send a HMAC-SHA2 code alongside it. If you want to protect yourself against replay attacks, it is as easy as including a timestamp in the payload.

Common use cases in the wild include API authentication (e.g AWS) and JSON web tokens.

## Don’t : use SHA for password hashes

Storing passwords in clear text is evil. While security best practices and password managers are on the rise, the vast majority of users reuse their passwords on multiple services. It is your duty to ensure that, in the event your database is comprised, the impact on your users is limited.

At a minimum, the passwords you store should be

- Cryptographically hashed (so while you can easily verify a match, the original plaintext one is not deducible from the hash)
- Salted (in order to be robust against rainbow table attacks)

While these practices are not bad per-ser, a standard hash function won’t do. Hash algorithms, even cryptographic ones, are so ubiquitous in today’s tech that their implementation must be **fast**.

Since your adversary knows both the hash and salt, and can with his GPU compute a billions SHA-256 hash codes per second, you have to use a hash function specifically **designed to be slow**.

You should use

- _Argon2_ : the most recent one, winner of the 2015 password hashing competition
- _BCrypt_ : an older one, more vulnerable to hardware-optimize attacks, but more vetted and maybe already part of your standard library

In short, when storing passwords, generate a secure and long hash, hash it with a slow hash function, and store only the hash and the salt.


## Conclusion

We hope these tips helped expand your knowledge of what hash functions can and cannot do. Hopefully you won’t make the same mistakes we did before learning these.

This article is only scratching the surface of the subject. Security is a complicated field, and there are still some debate in the community on certain points (e.g whether you should use SHA-3 or not).

If you wish to dig deeper, there are resources on the [security](https://security.stackexchange.com) and [crypto](https://crypto.stackexchange.com.) stackexchanges.

### More resources

- [Comparison of password hashing algorithms](https://security.stackexchange.com/questions/211/how-to-securely-hash-passwords/31846#31846)
- [Why / how should you hash passwords](https://crackstation.net/hashing-security.htm)
- [The libsodium crypto primitives](https://download.libsodium.org/doc/)
- [A guide to consistent hashing](https://www.toptal.com/big-data/consistent-hashing)
- [A guide to HMAC API authentication](https://blueprintinteractive.com/blog/securing-your-api-hmac-authentication)
- [An introduction to bloom filters](https://hackernoon.com/probabilistic-data-structures-bloom-filter-5374112a7832)
- [The current NIST policy on hash functions](https://csrc.nist.gov/Projects/Hash-Functions/NIST-Policy-on-Hash-Functions)

<style>
  h2 {
    margin-top: 3em;
  }
  h3 {
    margin-top: 2em;
  }
  table {
    width: 100%;
    border: 1px solid #cbcbcb;
    border-collapse: collapse;
    border-spacing: 0;
    margin: 1.5em 0em;
  }
  thead {
    background-color: #e0e0e0;
    color: #000;
    text-align: left;
    vertical-align: bottom;
  }
  td:first-child,  th:first-child {
    border-left-width: 0;
  }
  td, th {
    border-left: 1px solid #cbcbcb;
    border-width: 0 0 0 1px;
    font-size: inherit;
    margin: 0;
    overflow: visible;
    padding: .5em 1em;
  }
</style>


