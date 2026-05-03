# Flag in Flame

**Platform:** picoCTF 2025 (CMU-Africa Edition)
**Category:** Forensics
**Difficulty:** Easy
**Author:** Prince Niyonshuti N.
**Points:** Not disclosed

---

## Challenge Description

> The SOC team discovered a suspiciously large log file after a recent breach.
> When they opened it, they found an enormous block of encoded text instead of
> typical logs. Could there be something hidden within? Your mission is to
> inspect the resulting file and reveal the real purpose of it. The team is
> relying on your skills to uncover any concealed information within this
> unusual log. Be prepared — the file is large, and examining it thoroughly is crucial.

**File provided:** `logs.txt`

---

## Initial Observations

Downloaded the file and opened it in a text editor. It was enormous — thousands
of lines of what appeared to be random characters. No normal log structure:
no timestamps, no severity levels, no hostnames. Just a massive block of
character data.

The challenge description directly hints: "Use Base64 to decode the data and
generate the image file." The log file is not actually a log — it is a Base64
encoded image disguised as log data.

This scenario has real-world relevance. Attackers have used Base64-encoded
payloads stored in log-like files as a data exfiltration and malware delivery
technique precisely because log files are often overlooked and their large
size deters manual inspection.

---

## Tools Used

- **CyberChef** — Base64 decoding and image rendering
- **Text editor / hex viewer** — initial file inspection

---

## Methodology

**Step 1 — Open and assess the file**

Opened `logs.txt` in a text editor. The content was uniform — no line-to-line
variation in character type, no structure visible. The character set was
consistent with Base64: letters, numbers, `+`, `/`, and `=`.

**Step 2 — Base64 decode the entire file in CyberChef**

Loaded the full `logs.txt` content into CyberChef → From Base64 operation.

Output: an image file. The entire "log" was a Base64-encoded image.
CyberChef's Render Image button displayed it immediately.

**Step 3 — Analyze the image**

The rendered image contained a graphic and a long string of text printed at
the bottom. The string did not look like normal text — it looked like hex:

```
7069636F4354467B666F72656E736963735F616E616C797369735F69735F616D617A696E675F62653836303237397D
```

Pure hexadecimal — characters only in the range `[0-9a-f]`, even length,
no spaces. Classic hex-encoded string.

**Step 4 — Hex decode in CyberChef**

Pasted the hex string into CyberChef → From Hex operation:

```
7069636F4354467B666F72656E736963735F616E616C797369735F69735F616D617A696E675F62653836303237397D
→ picoCTF{forensics_analysis_is_amazing_be860279}
```

---

## Key Concepts Learned

**Multi-layer encoding is the standard, not the exception.**
This challenge required two decoding steps: Base64 (file → image) then hex
(image text → flag). Real-world malware routinely uses 3, 4, or more encoding
layers. The habit of asking "is this output itself encoded?" after every decode
is essential. The hex string in the image was visually distinctive — uniform
character set, even length — but recognizing it requires pattern recognition
built from seeing hex encoding repeatedly.

**Encoded data disguised as legitimate file types is a real threat.**
A Base64-encoded payload stored in a `.txt` file named `logs.txt` will pass
a file extension check, pass a file type check (it is a valid text file), and
blend in with a directory full of actual log files. The only detection approach
is content inspection — examining what the file actually contains, not just
what it claims to be. This is why SIEM rules that alert on unusually large
text files or text files with uniformly high entropy are valuable.

**Log file analysis is a core SOC skill — and attackers know it.**
The challenge framing is not accidental. A real SOC analyst receiving this
"log file" during an incident investigation would be expected to identify
that it is not a normal log, determine that it is Base64 encoded, and decode
it to reveal the hidden content. That workflow is identical to what this
challenge required.

**Hex identification by character set.**
Hexadecimal strings only contain `[0-9a-f]` (or uppercase `[0-9A-F]`).
When text in an image or file contains only these characters and has an even
length, hex decoding is the first thing to try. The string
`7069636F...` starting with `7069` is particularly recognizable —
`70 69 63 6F` decodes to `p i c o` — the start of `picoCTF`.

---

## Flag

`picoCTF{forensics_analysis_is_amazing_be860279}`
