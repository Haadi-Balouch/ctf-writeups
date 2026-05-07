# Shared Secrets

**Platform:** picoCTF 2026
**Category:** Cryptography
**Difficulty:** Easy
**Author:** Yahaya Meddy
**Repo:** `ctf-writeups/picoCTF-2026/crypto/`

---

## Challenge Description

> A message was encrypted using a shared secret... but it looks like one side
> of the exchange leaked something. Can you piece together the secret and get the flag?

**Files provided:** `message.txt`, `encryption.py`

---

## First Look

Two files — an encrypted message and the Python source code that encrypted it.
When you get source code in a crypto challenge, reading it carefully before
touching anything else is always the right move. The source code tells you
exactly how the encryption works, which tells you exactly how to reverse it.

I opened `encryption.py` and `message.txt` and read through both carefully.

---

## Understanding the Encryption

The source code showed Diffie-Hellman (DH) key exchange being used to generate
a shared secret, which was then used to XOR-encrypt the flag.

The parameters in `message.txt` were:

```
g = 2
p = [a huge ~1048-bit prime]
A = [a large number — g^a mod p, computed by Alice]
b = 5091601777...  [Bob's private exponent]
enc = 2a333935190e1c213e320529693928692e056b38393c6b633b6327
```

In a normal Diffie-Hellman exchange, both sides keep their private exponents
(`a` and `b`) secret. Alice sends `A = g^a mod p` and Bob sends `B = g^b mod p`.
The shared secret is `A^b mod p` (computed by Bob) or `B^a mod p` (computed by Alice)
— both equal the same value.

The attack here was immediately obvious once I saw `b` sitting in the message
file. **Bob's private exponent was leaked.**

In a real DH exchange, the attacker only sees `g`, `p`, `A`, and `B`. Computing
`a` or `b` from those public values is the discrete logarithm problem —
computationally infeasible for a prime this large. But here `b` was handed
to me directly. No discrete log needed. No brute force. Just arithmetic.

---

## Computing the Shared Secret

With `b` known and `A` available, the shared secret is straightforward:

```python
shared = pow(A, b, p)
```

Python's built-in `pow(base, exp, mod)` handles modular exponentiation
efficiently even with enormous numbers.

---

## Decrypting the Flag

Looking at the encryption code, the XOR key was `shared % 256` — just the
lowest byte of the shared secret. The flag was XOR'd byte by byte with this
single key value:

```python
enc_bytes = bytes.fromhex(enc)
flag = bytes([c ^ key for c in enc_bytes])
```

I wrote the full solver script, saved it as `flag.py`, and ran it:

```python
g = 2
p = [value from message.txt]
A = [value from message.txt]
b = [value from message.txt]
enc = "2a333935190e1c213e320529693928692e056b38393c6b633b6327"

shared = pow(A, b, p)
key = shared % 256

cipher = bytes.fromhex(enc)
flag = bytes([c ^ key for c in cipher])
print(flag.decode())
```

```bash
cat flag.py       # verified the script looked correct
python3 flag.py   # executed it
```

Output:

```
picoCTF{dh_s3cr3t_1bcf19a9}
```

---

## What This Vulnerability Is

**Diffie-Hellman is only secure when private exponents stay private.**

The entire security guarantee of DH rests on one assumption: an attacker
observing the public values `g`, `p`, `A`, `B` cannot compute the shared secret
without solving the discrete logarithm problem. The moment `b` leaked into
`message.txt`, that guarantee collapsed completely. The encryption used was
technically DH — but it was DH with the private key sitting in the ciphertext file.

This is a key management failure, not a cryptographic one. The algorithm
itself (DH + XOR) wasn't broken. The implementation leaked the secret it
was supposed to protect.

**XOR with a single byte key is also weak by itself.**
Even if `b` had stayed secret, the shared secret was reduced to `shared % 256`
— a value between 0 and 255. An attacker who somehow obtained one known
plaintext byte could brute-force all 256 possible keys trivially.
Strong shared secrets should be processed through a proper key derivation
function (KDF) rather than a simple modulo operation.

---

## What I Learned

This was my first actual crypto challenge involving a real cryptographic
protocol rather than encoding tricks (Base64, hex, ROT13). A few things
that clicked clearly:

**Reading source code is the most important skill in crypto challenges.**
The source code told me the exact encryption steps in order. Once I understood
the code, the solution was just those steps in reverse.

**Diffie-Hellman in practice — the math is elegant.**
`pow(A, b, p)` in Python runs instantly even for 1000-bit numbers. Python's
arbitrary precision integers and built-in modular exponentiation handle
the scale automatically. The math that looks intimidating on paper is a
one-liner in Python.

**"Leaked" in crypto challenges means leaked in the actual files.**
My first instinct was that the challenge would require some clever attack.
The actual solution was that `b` was just there in the file. Sometimes the
vulnerability is not subtle at all — you just have to read what you're given.

---

## Flag

`picoCTF{dh_s3cr3t_1bcf19a9}`
