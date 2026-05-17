# Treasure Hunt

**Platform:** TJCTF 2026
**Category:** Web
**Difficulty:** Easy
**Author:** elliott
**Points:** 109 (446 solves)
**Repo:** `ctf-writeups/TJCTF-2026/web/`

---

## Challenge Description

> Let's go hunt down some treasure! The flag is split into 4 parts.
> I'll give you the first one right here: `tjctf`

The challenge hands you the first piece of the flag and tells you to find
the other three across the website at `treasure-hunt.tjc.tf`.

---

## First Look

Opened the site. A pirate-themed page with some text and a ship PNG.
There was a "Learn More" button — clicked it and it took me to a second
page with a penguin PNG.

The challenge description already said the flag is split into 4 parts and
gave me the first one: `tjctf`. So I needed three more pieces hidden
somewhere on or around this site. The obvious places to check in order
were: page source, cookies, and the image files themselves.

---

## Finding Part 2 — Page Source

Opened DevTools on the second page (the penguin page) and looked through
the HTML. In the source I found a hidden paragraph element:

```html
<p hidden="">_and_</p>
```

The `hidden` attribute makes the element invisible to the user but it
is still in the DOM and fully visible in DevTools. This is a common
beginner mistake — hiding something from the visual display does not
hide it from anyone who opens inspect.

**Part 2: `_and_`**

---

## Finding Part 3 — Cookies

Went to DevTools → Application tab → Cookies → selected the site URL.

My session cookie was there, and alongside it was another cookie with a
value that looked like part of a flag:

```
{s1lv3r
```

**Part 3: `{s1lv3r`**

---

## Trying the Image Files

At this point I had three pieces: `tjctf`, `{s1lv3r`, `_and_`.

The flag format for TJCTF is `tjctf{...}` so assembling what I had gave:
`tjctf{s1lv3r_and_` — which needed one more piece to close the flag.

I tried to find part 4 in the image files. Downloaded both PNGs and ran
the standard forensics toolkit on them:

```bash
file ship.png
exiftool ship.png
strings ship.png
binwalk ship.png
pngcheck ship.png
```

Checked response headers in the Network tab too. Nothing obvious came
out of any of these. The fourth part was either very well hidden or I
was looking in the wrong place — but I had been spending a significant
amount of time on the images and decided to try something different.

---

## Educated Guess

I had: `tjctf{s1lv3r_and_???}`

The theme is pirate treasure. Silver and gold are the two classic
pirate treasures. `s1lv3r` was already leet-speak for silver.

I guessed:

```
tjctf{s1lv3r_and_g0ld}
```

Submitted it. It was correct.

---

## What This Challenge Teaches

**The flag was hidden across four locations that cover the basic web
reconnaissance checklist:**

| Part | Where It Was | Technique |
|---|---|---|
| `tjctf` | Challenge description | Given |
| `{s1lv3r` | Browser cookie | Cookie inspection |
| `_and_` | Hidden HTML element | Source code inspection |
| `g0ld}` | Presumably in the images | Forensics / guesswork |

The first three parts reinforce the core web recon habit: when you land
on a web challenge, immediately check source, cookies, and headers before
doing anything else. These are the fastest possible checks and they
frequently contain exactly what you need.

**`hidden` in HTML does not mean hidden from attackers.**
The `hidden` attribute and CSS `display: none` both prevent the element
from rendering visually. But they do absolutely nothing to protect the
data — it is still transmitted to the browser and sitting in the DOM.
Sensitive data should never be hidden this way. It should simply not
be sent to the client at all.

**Cookies are not private storage.**
Any value stored in a browser cookie is readable by the user (and any
attacker with access to that browser). Cookies are appropriate for
session tokens and preferences, not for hiding parts of secrets.

---

## Honest Note on Part 4

I did not definitively find part 4 through technical analysis — I guessed
it correctly from context. The pirate theme and the pattern `s1lv3r_and_`
made `g0ld}` the obvious completion. In a real scenario, the fourth part
likely lived in image metadata or steganographic data in one of the PNGs
that I may have overlooked during my binwalk and exiftool pass. This is
documented honestly because the methodology matters more than the outcome.

---

## Flag

`tjctf{s1lv3r_and_g0ld}`
