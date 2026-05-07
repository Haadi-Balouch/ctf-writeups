# StegoRSA

**Platform:** picoCTF 2026
**Category:** Cryptography / Steganography
**Difficulty:** Easy
**Author:** Yahaya Meddy
**Repo:** `ctf-writeups/picoCTF-2026/crypto/`

---

## Challenge Description

> A message has been encrypted using RSA. The public key is gone... but someone
> might have been careless with the private key. Can you recover it and decrypt
> the message?

**Files provided:** `flag.enc`, `image.jpg`

---

## First Look

Two files — an encrypted flag and a JPG image. The challenge says the private
key might have been hidden carelessly. Hidden data inside an image is steganography,
and by this point I had already solved Hidden in Plainsight which used exiftool
to find data in image metadata. That was my first instinct here.

```bash
exiftool image.jpg
```

Most fields looked normal — camera model, dimensions, timestamps. Then I hit
the Comment field:

```
Comment: 2d2d2d2d2d424547494e2050524956415445204b45592d2d2d2d2d0a...
```

A very long string of hex characters. The `2d` repetitions at the start were
a giveaway — `2d` in ASCII is the `-` character, and `-----BEGIN PRIVATE KEY-----`
starts with exactly five dashes. This was hex-encoded PEM data.

---

## Extracting the Private Key

Pasted the hex string into CyberChef → From Hex operation. The output was
exactly what I expected:

```
-----BEGIN PRIVATE KEY-----
MIIEvQIBADANBgkqhkiG9w0BAQEFAASCBKcwggSjAgEAAoIBAQCtVf6CDwFYrEnH
k4R/8NMG0Sb8m6mVUy4y1LAX8nbYlJI6llRp0gvi1wRbsoDdmZ48oCFBd26rlc01
...
-----END PRIVATE KEY-----
```

A complete RSA private key hidden in the image's Comment metadata field,
hex-encoded so it wouldn't look obviously like a key to a casual inspection.

I saved this output to a new file:

```bash
# Pasted CyberChef output into a new file
nano image.txt
# Pasted the full -----BEGIN PRIVATE KEY----- block, saved and exited
```

---

## Decrypting the Flag

With the private key recovered, I used OpenSSL to decrypt `flag.enc`:

```bash
openssl rsautl -decrypt -inkey image.txt -in flag.enc
```

Output:

```
picoCTF{rs4_k3y_1n_1mg_51611ab8}
```

---

## What This Vulnerability Is

**Hiding a private key in an image's metadata is not security.**
A private key needs to be kept genuinely secret — not encoded and stuffed into
a publicly distributed file. The hex encoding provides zero protection; any
tool that reads metadata would surface it. The person who created this challenge
was demonstrating exactly how attackers think: when you can't find the private
key in the obvious places, look at the assets that come with the ciphertext.

**The broader lesson — steganography and cryptography don't mix well.**
RSA is a strong encryption algorithm. The weakness here was not the math —
it was key management. A perfectly implemented RSA encryption is useless if
the private key is embedded in a publicly downloadable image. This is a
principle I've seen across multiple challenges: cryptographic strength means
nothing if the key handling is careless.

---

## What I Learned

This challenge was the first time I combined two techniques I had learned
separately — metadata extraction from images (exiftool, from Hidden in
Plainsight) and cryptographic decryption (openssl). Neither step alone was
new. What was new was recognizing that the image was the container for the key
rather than the flag itself, and that the hex comment was a PEM key before I
even decoded it.

Recognizing `2d2d2d2d2d` as five dashes — the start of a PEM header — is the
kind of pattern recognition that builds up from repeated exposure to hex data.
That clicked faster than I expected.

---

## Flag

`picoCTF{rs4_k3y_1n_1mg_51611ab8}`
