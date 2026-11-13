---
name: "prep-listing-photo"
description: "Use when the user asks to prep a photo (or photos) for eBay or another listing: remove the background, replace it with white, centre the item on a square canvas."
---

# Prep listing photos

Turn the user's product photos (usually clothing laid flat on a floor or rug) into clean eBay-ready images: background removed, pure white behind, a subtle drop shadow under the item, item centred on a 2000x2000 square JPEG.

## Steps

1. **Find the photos.** Attached images live in `/mnt/user-data/uploads/` (read-only). Work in the scratchpad directory, never on the originals.

2. **Install the cutout tool** if missing:
   ```bash
   pip install --break-system-packages -q "rembg[cpu]"
   ```
   Use the `isnet-general-use` model. Do NOT use `birefnet-general` — it is ~1GB and gets killed for lack of memory in this workspace. The model downloads from GitHub on first use, which is allowed.

3. **Decide orientation first.** Look at each original photo. If a shot is upside down (hood/collar at the bottom, labels inverted), list it in `ROTATE` below so it is turned 180 degrees BEFORE the shadow is drawn — rotating the finished image would put the shadow above the item. Tell the user which shots you turned.

4. **Make the final images** with this script (pass the output folder, then the photo paths). Set `ROTATE` to the basenames (without extension) of any upside-down shots, and `SHADOW = False` if the user asks for no shadow:
   ```python
   import sys, os
   from PIL import Image, ImageOps, ImageFilter
   from rembg import remove, new_session

   ROTATE = set()        # e.g. {'IMG_2298'}
   SHADOW = True
   S, pad = 2000, 0.06   # canvas size, margin on each side
   out = sys.argv[1]; os.makedirs(out, exist_ok=True)
   session = new_session('isnet-general-use')
   for path in sys.argv[2:]:
       name = os.path.splitext(os.path.basename(path))[0]
       im = ImageOps.exif_transpose(Image.open(path)).convert('RGB')
       m = remove(im, session=session, post_process_mask=True).split()[-1]
       m = m.point(lambda v: 0 if v < 20 else v)          # drop faint specks
       if name in ROTATE:
           im, m = im.rotate(180), m.rotate(180)
       bbox = m.point(lambda v: 255 if v > 128 else 0).getbbox()
       im, m = im.crop(bbox), m.crop(bbox)
       w, h = im.size; sc = S * (1 - 2 * pad) / max(w, h)
       nw, nh = round(w * sc), round(h * sc)
       im = im.resize((nw, nh), Image.LANCZOS); m = m.resize((nw, nh), Image.LANCZOS)
       x, y = (S - nw) // 2, (S - nh) // 2
       c = Image.new('RGB', (S, S), 'white')
       if SHADOW:
           # subtle drop shadow: blurred silhouette, nudged down and slightly right
           off, blur, opacity = int(S * 0.012), int(S * 0.018), 0.28
           sh = Image.new('L', (S, S), 0)
           sh.paste(m, (x + off // 2, y + off))
           sh = sh.filter(ImageFilter.GaussianBlur(blur)).point(lambda v: int(v * opacity))
           c.paste(Image.new('RGB', (S, S), (0, 0, 0)), (0, 0), sh)
       c.paste(im, (x, y), m)
       c.save(os.path.join(out, f'{name}_white.jpg'), quality=92)
       print(name, 'done')
   ```

5. **Check the results visually.** Build a small side-by-side contact sheet of the outputs on a grey backing and look at it with Read. Check for:
   - leftover background (rug, floorboards, chair wheels, feet/slippers at the edges)
   - bits of the item cut away (hood, sleeves, drawcords, labels)
   - the shadow sitting just below the item, soft and faint, with the canvas corners still pure white (255,255,255)
   - every shot the right way up
   Fix anything wrong before delivering.

6. **Name the files** by view where it's obvious (e.g. `<item>_1_front.jpg`, `<item>_2_inside.jpg`, `<item>_3_back.jpg`) so they upload in a sensible order — front first, as that becomes the listing's main photo.

7. **Deliver** with SendUserFile (display: render), all files in one call. Finish with a couple of sentences: they're 2000x2000 on pure white with a subtle shadow, and mention any rotation or fix you made. No step recap.

## Defaults and adjustments
- Pure white (#FFFFFF) background, square 2000x2000, 6% margin, JPEG quality 92, subtle shadow on.
- Stronger or softer shadow: raise or lower `opacity` (0.28 default; 0.4 is noticeable, 0.15 is barely there) and `blur`.
- No shadow: `SHADOW = False`.
- Different size, margin or background colour: change `S`, `pad`, or the canvas colour.
- PNG with transparency: save the cutout as PNG with the mask as alpha instead of pasting on white (no shadow).