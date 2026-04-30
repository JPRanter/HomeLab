# Dolphin Thumbnails Were Fine. The JPEGs Were Not.

If Dolphin isn't generating previews for some images but Gwenview opens them without complaint, the problem probably isn't Dolphin.

## The Setup

Nobara Linux, KDE Plasma on Wayland, Dolphin as the file manager. Image collection on a Samba share, browsed over SMB via KIO. Most images previewed correctly. A seemingly random subset showed the generic orange image icon regardless of what was tried.

The obvious suspects got ruled out in order: thumbnail plugins present, size limit not set, cache tiers populated, KIO working, no fail cache entries. Dolphin was generating thumbnails for the files it liked and silently skipping others. No error. No log output worth reading.

## The Troubleshooting Path

First assumption: Wayland sandboxing was interfering with the KIO thumbnail worker on network shares. Reasonable guess. Wrong.

Launching Dolphin forced to XCB to rule out Wayland produced this in the terminal immediately:

```
qt.gui.imageio.jpeg: Not a JPEG file: starts with 0x52 0x49
```

`0x52 0x49` is `RI` — the start of a RIFF container. Those files were WebP. Named `.jpg`, saved as WebP, and Qt's JPEG decoder was correctly refusing to touch them.

A second pass found PNG impostors in the same collection doing the same thing.

## Why Gwenview Didn't Care

Gwenview uses KImageFormats and Qt image plugins, which sniff the actual file magic bytes and decode by content regardless of extension. Dolphin's thumbnailer takes the `.jpg` extension at face value, invokes the JPEG decoder, hits a RIFF or PNG header, and bails silently.

Same files, same formats, different tools, different tolerance for lies.

## Why Some Previewed and Others Didn't

The files that previewed correctly were genuine JPEGs. The ones that didn't were WebP or PNG files saved with a `.jpg` extension — a byproduct of downloading images and changing the extension manually without converting the actual format. The size distribution was coincidental. The pattern that looked like a size threshold was just the ratio of real JPEGs to impostors in the collection.

## The Fix

Find the impostors and convert them in place:

```bash
find /path/to/collection -type f \( -iname "*.jpg" -o -iname "*.jpeg" \) | while read f; do
    TYPE=$(file "$f")
    if echo "$TYPE" | grep -q "Web/P"; then
        magick "$f" -quality 95 "$f"
    elif echo "$TYPE" | grep -q "PNG"; then
        magick "$f" -quality 95 "$f"
    fi
done
```

Then clear the thumbnail cache so Dolphin regenerates from the corrected files:

```bash
rm -rf ~/.cache/thumbnails/
```

A shell script wrapping this logic with a path argument and automatic cache clearing is available in this repo.

Note on quality: WebP-to-JPEG and PNG-to-JPEG are both lossy conversions. There is no lossless path to JPEG from either format. Quality 95 is a reasonable trade-off — minimal visible degradation, correct format, Dolphin happy.

## What Was Actually Broken

The thumbnailing pipeline. The file manager. The KIO worker. Wayland. The SMB mount. None of it.

The files were mislabeled. Everything downstream behaved exactly as it should.

---

*Written with assistance from Claude Sonnet 4.6 (Anthropic) as part of an ongoing homelab documentation practice. The architecture decisions, the direction, and the opinions are mine. Claude helped build it faster and document it more clearly than I would have alone.*
