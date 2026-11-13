# prep-listing-photo

A Claude skill that turns product photos into clean, eBay-ready listing images.

- Removes the background.
- Adds a pure white backdrop and a subtle drop shadow.
- Centres each item on a 2000 x 2000 square JPEG.

## Features

- **Background removal** using the `isnet-general-use` model.
- **Orientation fix** for upside-down shots, applied before the shadow is drawn.
- **Configurable shadow** with adjustable opacity and blur, or none at all.
- **Listing-ready output** with a 6% margin and JPEG quality 92.
- **Transparent PNG option** for use outside a white background.

## Output specifications

| Setting | Default |
| --- | --- |
| Canvas size | 2000 x 2000 px |
| Background | `#FFFFFF` |
| Margin | 6% each side |
| Format | JPEG, quality 92 |
| Shadow | On (opacity 0.28) |

## Requirements

- Python 3
- Pillow
- `rembg[cpu]`

## Installation

1. Copy this folder into your skills directory, for example `~/.claude/skills/prep-listing-photo/`.
2. Keep `SKILL.md` at the root of the folder.
3. Install the cutout tool:

   ```bash
   pip install --break-system-packages -q "rembg[cpu]"
   ```

## Usage

Attach your photos to a chat and ask Claude to prep them for a listing. For example:

> Prep these photos for eBay.

Claude will:

1. Check each photo's orientation.
2. Remove the background and centre the item.
3. Check the results visually.
4. Return the files, named by view, with the front shot first.

## Customisation

| To change | Edit |
| --- | --- |
| Shadow strength | `opacity` (0.28 default; 0.15 is subtle, 0.4 is noticeable) |
| Shadow softness | `blur` |
| No shadow | `SHADOW = False` |
| Canvas size | `S` |
| Margin | `pad` |
| Transparent PNG | Save the cutout with its mask as alpha instead of pasting on white |

## Notes

- Use `isnet-general-use`. The `birefnet-general` model needs about 1 GB of memory and can be killed in constrained environments.
- The first run downloads the model from GitHub.
