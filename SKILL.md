---
name: "write-wayfarer-nomination"
description: "Draft Niantic Wayfarer nomination text from a proposed title, one or more photos, and optional explanation; inspect nomination photos for real people and return people-free iPhone Photos compatible copies when needed. Use for Wayspot or PokéStop nominations, the familiar three copyable fields (Title, Description, Supporting Information), revisions, or people removal from Wayfarer nomination photos. Do not use for unrelated photo editing or Pokémon GO questions."
---

# Wayfarer nomination writer

## Tooling

- Photo edits: `media.generate_image` in edit mode. Pass the photo as a `kind: image` prompt entry plus a text instruction naming only what to remove; set `output_format: "jpg"` so the result is a high-quality JPEG the iPhone Photos app can import. Save edited copies under `~/workspace/wayfarer/edited/` with the source filename stem plus `-nopeople.jpg`; never overwrite the original.
- Delivery: attach each edited image in the reply on its own line as `![<short caption>](sandbox://workspace/wayfarer/edited/<file>.jpg)`. The user opens and downloads from there.
- Geometric work (square crop, resize, rotate): Python PIL via `python3` — deterministic and pixel-exact. Never use `media.generate_image` for geometric transforms; it regenerates pixels and can warp the subject.
- Fact checks: `browser.search` for official names, artists, or historical claims when they would materially improve wording.

## Prepare

1. Inspect every supplied photo and the user's title and explanation. Identify the particular permanent object or place being nominated, its visible features, surroundings, and any readable signage. Do not assume a location or official name from appearance alone.
2. If an official name, artist, historical claim, or distinctive place fact would materially improve the wording, verify it with an authoritative source when feasible. Keep unverified claims out of the nomination. Do not invent dates, artists, public access, community use, or uniqueness.
3. Apply current Wayfarer guidance when needed: a candidate should support exploration, exercise, or socializing; be a permanent tangible identifiable place or object; and have safe pedestrian access. Consult the [official criteria](https://niantic.helpshift.com/hc/en/21-wayfarer/section/166-wayspot-criteria/) for borderline cases. Eligibility is not a guarantee of acceptance. Distinguish the specific nominated feature from the broader venue and possible existing Wayspots.

## Prepare photos

1. Square crop (main photo only): the Wayspot main image displays as a square, and a non-square upload gets stretched, warped, or center-cropped by the app. Check the main photo's aspect ratio with PIL. If it is not square, produce a square crop with the nominated subject centered and fully contained: use the shorter side as the crop side, position the crop window on the subject's center, and verify no part of the subject is cut off. Save the crop as a separate file (`<stem>-square.jpg` under `~/workspace/wayfarer/edited/`, original preserved) and continue all later photo steps on the cropped copy. If the subject cannot fit fully inside a square of the shorter dimension — or no crop keeps it centered without cutting it off — do NOT crop, and add a warning to the Note that a square crop is recommended for the main image to prevent stretching and warping.
2. Check each supplied nomination photo for real people, including small figures in the background and reflections. Do not mistake human figures depicted in a mural, sculpture, sign, or other nominated artwork for bystanders; preserve the artwork.
3. If a photo contains real people, edit it with `media.generate_image` in edit mode: instruct it to remove only those people and reconstruct only the occluded background. Preserve the nominated object, signs and legible text, architecture, landscaping, perspective, framing, lighting, and all other scene details. Do not move, replace, beautify, or invent a different Wayspot or location. Keep the original photo intact and make a separate edited copy for each affected input photo.
4. Inspect each edited result against its original. If the subject or surroundings were materially changed, retry a focused edit when possible (use `resume_from_snapshot_id` rather than re-uploading). If a faithful removal is not possible, say so and recommend a new photo rather than presenting a misleading image as nomination-ready.
5. When no real people appear, leave the photos unchanged. If image editing is unavailable, explain the limitation while still supplying the three text fields.
6. Retain coordinates, drop timestamps: every image you output (square crops and people-removed copies) must carry the source photo's GPS coordinates — Wayfarer nominations use them for location, and both PIL crops and `media.generate_image` edits strip EXIF. Stamp the source's GPS coordinates onto the output, but strip ALL date/time tags (capture timestamps, file timestamps, and GPS date/time stamps) so the edited copy sorts as a new photo in the phone's photo picker instead of burying itself next to the original. Keep Orientation normal (the output is already upright):

```python
from PIL import Image
from PIL.ExifTags import IFD
src_exif = Image.open(original_path).getexif()
out = Image.open(output_path)
new_exif = out.getexif()
gps = src_exif.get_ifd(IFD.GPSInfo)
if gps:
    for tag in (7, 29):  # GPSTimeStamp, GPSDateStamp
        if tag in gps:
            del gps[tag]
    new_exif[IFD.GPSInfo] = gps
# strip timestamps so the edit sorts as a new photo in the picker
for tag in (306, 36867, 36868):  # DateTime, DateTimeOriginal, CreateDate
    if tag in new_exif:
        del new_exif[tag]
new_exif[274] = 1  # Orientation: normal
out.save(output_path, exif=new_exif)
```

If the source photo has no GPS data, say so in the Note rather than silently delivering a coordinate-less image.

## Write the fields

- **Title:** Prefer the official name when supported. Otherwise use a concise, specific, natural name that distinguishes the object, using location context or accurate numbering when useful. Avoid game terms, player references, addresses, emojis, promotional claims, and arbitrary numbering presented as an official name.
- **Description:** Write one or two polished factual sentences about what a visitor sees or why this particular feature matters. This field should stand alone as a public place description. Avoid reviewer appeals, eligibility language, subjective praise, unsupported history, and game references.
- **Supporting Information:** Explain the strongest genuine exploration, exercise, or social value and connect it to visible or verified facts. Help reviewers identify the exact object and understand its placement and pedestrian approach when the evidence supports those claims. For a borderline candidate, address the real concern candidly. Do not claim that something is public, permanent, distinct, or safely accessible unless supported. Avoid voting requests and generic boilerplate.

## Response format

- Always provide a usable best-effort draft, including for lower-chance nominations. Do not withhold all text merely because a candidate seems weak; do not fabricate a persuasive case.
- Output exactly three clearly labeled, separate plain-text fenced code blocks, in this order, with no explanatory text inside the boxes:

  **Title**
  ```text
  Suggested name
  ```

  **Description**
  ```text
  Public description
  ```

  **Supporting Information**
  ```text
  Reviewer context
  ```

- If useful, follow the boxes with a brief **Note** about a meaningful eligibility issue, an uncertain fact to confirm, a duplicate risk, or a photo problem. Keep it separate so all three boxes remain directly copyable. When web research informed a fact, put source links outside the boxes unless a direct URL is specifically useful in Supporting Information.
- If multiple distinct candidates are shown, produce a separate set of three boxes per candidate and label each set. If the user's intent is clear enough for a draft, do not pause for routine clarification; state any consequential uncertainty in the note.
- Place edited images outside the three text boxes. A short caption may identify the source photo or distinguish main and supporting views. Do not put image links or editing notes into Title, Description, or Supporting Information unless they belong in the nomination itself.

## Photo and honesty checks

- Prefer a clear main image centered on the actual subject and a supporting image that includes the subject and recognizable surroundings. If a visible issue may affect review (for example, license plates, blur, a screenshot, or conspicuous editing artifacts), mention it briefly after the boxes. Wayfarer guidance flags prominent people and recognizable faces, but also obviously edited or doctored photos; flag any removal that looks conspicuous and suggest retaking the photo when practical.
- Keep title, description, and support consistent with each other and the photos. Do not suggest moving the pin away from the real object, disguising a private or unsafe location, or influencing reviewers with voting requests.
