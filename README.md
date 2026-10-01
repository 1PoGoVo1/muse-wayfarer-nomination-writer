# Wayfarer Nomination Writer

> Built for [Muse](https://muse.ai). The photo-editing and delivery steps call Muse-specific tools, so this skill won't run as-is on other assistants.
>
> New to Muse? Redeem invite code `JDG06J` in Settings within 48 hours of joining — we both get bonus tokens: https://muse.ai/join

A Muse skill for drafting Niantic Wayfarer nomination text from a proposed title, photos, and optional context.

Given a proposed title and one or more photos, it:

- Inspects the photos to identify the nominated place or object
- Square-crops the main photo when needed (deterministic PIL crop, subject kept centered and fully visible — never generative warping)
- Removes real people / photographer reflections from nomination photos via targeted image edits
- Stamps the source photo's GPS coordinates onto every output while stripping timestamps, so edited copies sort as new photos in the phone's photo picker
- Verifies official names and material facts against authoritative sources
- Returns the three copy-pasteable nomination fields: **Title**, **Description**, **Supporting Information**

See [SKILL.md](SKILL.md) for the full workflow.
