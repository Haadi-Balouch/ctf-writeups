# Riddle Registry

**Platform:** picoCTF 2025 (CMU-Africa Edition)
**Category:** Forensics
**Difficulty:** Easy
**Author:** Prince Niyonshuti N.
**Points:** Not disclosed

---

## Challenge Description

> Hi, intrepid investigator! You've stumbled upon a peculiar PDF filled with
> what seems like nothing more than garbled nonsense. But beware! Not everything
> is as it appears. Amidst the chaos lies a hidden treasure — an elusive flag
> waiting to be uncovered. Find the PDF file and uncover the flag within the metadata.

**File provided:** `confidential.pdf`

---

## Initial Observations

Downloaded the PDF and opened it. As the description said — garbled nonsense.
The visible content was meaningless. The challenge title "Riddle Registry" and
the hint about metadata pointed clearly at PDF metadata fields rather than
the visible content.

The description saying "not everything is as it appears" is a direct pointer
away from the visible content. In PDF forensics, metadata fields like Author,
Title, Creator, and Keywords are often overlooked — exactly where a CTF author
would hide something.

---

## Tools Used

- `file` — initial file type verification
- `sha256sum` — file integrity hash before analysis
- `pdfinfo` — PDF metadata extraction (Linux command line)
- **CyberChef** — Base64 decoding

---

## Methodology

**Step 1 — Basic inspection**

```bash
file confidential.pdf
# Output: PDF document, version 1.4

sha256sum confidential.pdf
# Recorded hash for documentation
```

Good forensic habit: always document the file hash before touching it.
This proves the file was not modified during analysis.

**Step 2 — PDF metadata extraction with pdfinfo**

```bash
pdfinfo confidential.pdf
```

Output showed standard PDF metadata fields. In the Author field:

```
Author: cGljb0NURntwdXp6bDNkX20zdGFkYXRhX2YwdW5kIV8zNTc4NzM5YX0=
```

The trailing `=` is an immediate Base64 indicator — Base64 strings commonly
end with `=` or `==` padding characters.

**Step 3 — Base64 decode in CyberChef**

Pasted the Author field value into CyberChef → From Base64 operation:

```
cGljb0NURntwdXp6bDNkX20zdGFkYXRhX2YwdW5kIV8zNTc4NzM5YX0=
→ picoCTF{puzzl3d_m3tadata_f0und!_3578739a}
```

---

## Key Concepts Learned

**PDF metadata is a real-world hiding place, not just a CTF trick.**
PDFs carry an Info dictionary with fields like Author, Title, Subject, Keywords,
Creator, and Producer. These fields are never displayed to the reader but are
fully accessible via tools. In real incident response, PDF metadata has revealed
attacker identity (author names, software used to create weaponized documents),
internal file paths, and organizational information that should not have been disclosed.

**Recognizing encoding by signature.**
The `=` padding at the end of the Author value was the giveaway. Developing
pattern recognition for common encodings — Base64 ends with `=`, hex uses only
`[0-9a-f]`, Base32 ends with `====` — is a fundamental forensics skill.
The faster you recognize the encoding, the faster the decode.

**Always check metadata before digging deeper.**
The chain was: visible content (garbled, useless) → metadata check → encoded string →
decoded flag. The metadata was the first non-trivial layer. In real forensics,
metadata analysis comes before any deeper file examination. It is fast, non-destructive,
and frequently the most productive step.

**pdfinfo vs exiftool**
Both extract PDF metadata. `pdfinfo` is PDF-specific and cleaner for PDF analysis.
`exiftool` works across dozens of file formats and is more comprehensive for
image files. For PDFs specifically, `pdfinfo` gave a cleaner output in this challenge.

---

## What I Would Do Differently

After pdfinfo, the next tools I would try if metadata had nothing:
- `strings confidential.pdf | grep picoCTF` — search for flag pattern in raw strings
- `pdfdetach` — extract any embedded attachments
- `pdf-parser.py` — inspect individual PDF objects for hidden streams

---

## Flag

`picoCTF{puzzl3d_m3tadata_f0und!_3578739a}`
