# Hidden in Plainsight

**Platform:** picoCTF 2025 (CMU-Africa Edition)
**Category:** Forensics / Steganography
**Difficulty:** Easy
**Author:** Yahaya Meddy
**Points:** Not disclosed

---

## Challenge Description

> You're given a seemingly ordinary JPG image. Something is tucked away out
> of sight inside the file. Your task is to discover the hidden payload and
> extract the flag.

**File provided:** `img.jpg`

---

## Initial Observations

An ordinary-looking JPG with nothing unusual visible. "Hidden in Plainsight"
immediately signals steganography — data hidden inside an image file. The
challenge title is also a hint at the technique: steghide is a tool that
hides data in plain sight inside image files.

First step as always: metadata before anything else.

---

## Tools Used

- `file` — file type verification
- `exiftool` — image metadata extraction
- **CyberChef** — Base64 decoding (twice)
- `steghide` — steganography extraction tool (`sudo apt install steghide -y`)

---

## Methodology

**Step 1 — Basic file check**

```bash
file img.jpg
# Output: JPEG image data, standard
```

Nothing unusual about the file type. No embedded signatures visible.

**Step 2 — Metadata inspection with exiftool**

```bash
exiftool img.jpg
```

Most fields were standard JPEG metadata. One field stood out immediately:

```
Comment: c3RlZ2hpZGU6Y0VGNmVuZHZjbVE9
```

Looks like Base64 — the character set and structure are consistent with it.

**Step 3 — First Base64 decode**

Pasted into CyberChef → From Base64:

```
c3RlZ2hpZGU6Y0VGNmVuZHZjbVE9
→ steghide:cEF6endvcmQ=
```

The output revealed two things at once: the tool to use (`steghide`) and
what appeared to be a passphrase (`cEF6endvcmQ=`). At this point I tried
using `cEF6endvcmQ=` directly as the steghide passphrase.

**Step 4 — First steghide attempt (wrong passphrase)**

```bash
steghide extract -sf img.jpg -p "cEF6endvcmQ="
```

Failed. Steghide rejected the passphrase.

This is where I got stuck for a bit. The output of the first decode was
`steghide:cEF6endvcmQ=` — I had been using only the part after the colon,
but that part itself ends in `=`, which is a Base64 padding character.
The actual passphrase was itself Base64-encoded.

**Step 5 — Second Base64 decode**

Went back to CyberChef and decoded the passphrase portion again:

```
cEF6endvcmQ=
→ pAzzword
```

The comment field contained two layers of Base64 encoding:
- Outer layer decodes to `steghide:<encoded_passphrase>`
- Inner layer decodes to the actual passphrase: `pAzzword`

**Step 6 — steghide extraction with correct passphrase**

```bash
steghide extract -sf img.jpg -p "pAzzword"
# Output: wrote extracted data to "flag.txt"

cat flag.txt
# picoCTF{h1dd3n_1n_1m4g3_f051f2e8}
```

---

## Key Concepts Learned

**Double encoding is a deliberate misdirection technique.**
The first decode giving something that looks usable (`cEF6endvcmQ=`) is a trap.
It looks like a passphrase. It is not. The second decode gives the actual passphrase.
In real forensics and malware analysis, multi-layer encoding is used specifically
to frustrate automated tools and analysts who stop at the first decoded output.
The lesson: always check if a decoded result is itself encoded before using it.

**steghide — a real steganography tool with real-world relevance.**
Steghide embeds data inside JPEG or BMP files by replacing the least significant
bits of image data with payload bits. The embedded data is encrypted with the
passphrase. Without the correct passphrase, the existence of hidden data is
not detectable without statistical analysis. This technique is used in actual
malware distribution — images containing hidden payloads that look completely
normal to any image viewer.

**Metadata can contain instructions, not just information.**
The comment field in this image did not contain a flag or a hint — it contained
the tool name and encoded passphrase needed to extract the actual flag. This is
a more sophisticated use of metadata than the Riddle Registry challenge.
In real malware analysis, metadata fields containing tool names, keys, or
command strings are significant threat intelligence indicators.

**Methodology for stuck points.**
When the first steghide attempt failed, the correct response was to go back
and re-examine the decoded output rather than try different tools. The data
was already telling me what to do — I just hadn't decoded it completely yet.

---

## Flag

`picoCTF{h1dd3n_1n_1m4g3_f051f2e8}`
