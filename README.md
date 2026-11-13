# prep-listing-photo

A Claude skill that turns product photos (usually clothing laid flat) into eBay-ready images: background removed, pure white behind, a subtle drop shadow, centred on a 2000x2000 square JPEG.

## Install

Copy the folder into your skills directory (for example `~/.claude/skills/prep-listing-photo/`), keeping `SKILL.md` at its root.

## Requirements

- Python with Pillow and `rembg[cpu]` (uses the `isnet-general-use` model)
