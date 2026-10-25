# pi-digits

Decimal digits of pi for [Pi Explorer](https://github.com/andyvr0o-ui/piexplorer), stored as
files in the release [`digits-v1`](https://github.com/andyvr0o-ui/pi-digits/releases/tag/digits-v1).

A Cloudflare Worker (code in [`piexplorer/worker/`](https://github.com/andyvr0o-ui/piexplorer/tree/main/worker))
copies them from Google's public y-cruncher file (`source` in `manifest.json`) when Pi Explorer
searches past the end: the app asks for more (`wanted` in `manifest.json`) and the Worker
adds a file every 5 minutes until it gets there. The app never uploads digits itself.

## Change how many digits may be kept

Edit `config.json` on github.com and change `targetDigits`: the store never grows past it
(at most 1000 files = 95 billion digits). Lowering it does not delete files.

## File format

`pi_NNNN.bin` (NNNN = k from 0000) is an exact copy of bytes `200 + k × 40,000,000` to
`200 + (k+1) × 40,000,000 − 1` of the source file: 5,000,000 little-endian 8-byte words, each
holding 19 digits (the word's value, padded to 19 digits with leading zeros). Word 0 of file 0
is `1415926535897932384` = decimal places 1–19.

Decimal place `p` (1-based, after the "3.") is in word `w = (p − 1) div 19`, file
`w div 5,000,000`, at byte `8 × (w mod 5,000,000)`. Each file holds 95,000,000 digits.

`manifest.json`: `files` = number of complete files (`pi_0000.bin` onward), `digits` =
`files × 95,000,000`. Only files counted there are complete.

## The shared Pi Map

`found.json` lists numbers Pi Explorer users found: `{"version":1,"finds":[{"d":"140392","p":1233,"n":3,"t":"2026-10-23","f":"9c1e04b2a7d35f60"}]}`
(`d` digits, `p` 0-based index so the number starts at decimal place `p + 1`, `n` times found,
`t` first found, `f` (on the first find of those digits only) a one-way mark of the first finder's random app id, different for every
find, so only that phone can recognise it). The app sends `POST /found?digits=D&place=P` to the Worker, which reads those
places from Google's file itself and adds them only when they match. Only digits and places
are kept, nothing that says who found them.
