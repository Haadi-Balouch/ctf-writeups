# Binary Digit

**Platform:** picoCTF 2026
**Category:** Forensics
**Difficulty:** Easy
**Author:** Yahaya Meddy
**Points:** Not disclosed

---

## Challenge Description

> This file doesn't look like much... just a bunch of 1s and 0s. But maybe
> it's not just random noise. Can you recover anything meaningful from this?

**File provided:** `digits.bin`

---

## Initial Observations

Downloaded the file and opened it in a text editor. The entire file was
nothing but a wall of `1`s and `0`s — pure binary. No structure visible
at the text level, no readable strings, just raw binary data.

First instinct: this is either binary-encoded text or binary-encoded image data.
Given the challenge name "Binary Digit" and the forensics category, image encoding
felt more likely — hiding an image as raw binary is a classic forensics technique.

---

## Tools Used

- **CyberChef** — browser-based data transformation tool

---

## Methodology

**Step 1 — Upload the file to CyberChef**

Opened CyberChef (gchq.github.io/CyberChef) and uploaded `digits.bin` directly.

**Step 2 — Try the "Render Image" operation**

CyberChef has a "Render Image" button that attempts to interpret the input data
as image bytes and display it visually. Clicked it and immediately got a rendered
image — the binary data was the raw pixel data for an image file.

**Step 3 — Read the flag**

The rendered image displayed the flag text directly on the image.

---

## Key Concepts Learned

**Binary data can represent anything — context determines interpretation.**
A `.bin` file is just raw bytes. The same data can be a text document, an image,
an executable, or anything else depending on how you interpret it. This challenge
reinforces a core forensics habit: when a file doesn't make sense in its default
form, try interpreting it differently.

**CyberChef's "magic" operations save significant time.**
Rather than writing a script to convert binary to an image format and open it,
CyberChef did the entire pipeline in two clicks. Knowing the tool's capabilities
is as important as knowing the underlying technique.

**Binary encoding of images**
Images are fundamentally just grids of pixel values — numbers. Those numbers
can be represented in binary (1s and 0s). Stripping all formatting and saving
just the raw binary digits is a simple way to obscure what the data actually is
while keeping it fully recoverable by anyone who knows to try binary-to-image
conversion.

---

## What I Would Do Differently

If CyberChef's Render Image had not worked, the next step would have been
writing a Python script to convert the binary string to bytes and then save
as a PNG or BMP file, then open it manually. It is worth knowing the manual
approach even when a tool automates it.

---

## Flag

`picoCTF{h1dd3n_1n_th3_b1n4ry_a59b2b0a}`
