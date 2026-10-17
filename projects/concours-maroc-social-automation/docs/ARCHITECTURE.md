# Technical Notes — Concours Maroc Social Automation

## Logical stages

1. **Read listings**: fetch official index, normalize and deduplicate `source_url`.
2. **Read history**: consult `concours_maroc_drafts` and calculate `needsInstagram` and `needsTikTok` independently.
3. **Fetch a single listing**: extract listing metadata, official PDF URL and published deadlines.
4. **Extract with OpenAI**: the AI must distinguish individual vacancy counts from multi-vacancy PDF decrees.
5. **Quality gate**: verify counts, deadlines, grade, specialties and diplomas. Escalate ambiguity and discrepancies.
6. **Build HTML poster**: 1080 x 1350 branded layout, longer conditions in the caption.
7. **Render media**: HCTI creates an image; validate the returned image URL.
8. **Buffer**: separate Instagram and TikTok branches, each with its own authorization and duplicate check.
9. **Persist success**: on successful API response, upsert the Buffer ID in the relevant Data Table column.

## Data Table schema

| Column | Type | Purpose |
| --- | --- | --- |
| `source_url` | text | Unambiguous identifier of the individual official listing |
| `instagram_draft_id` | text, nullable | Stored Buffer Instagram post/draft ID |
| `tiktok_draft_id` | text, nullable | Stored Buffer TikTok post/draft ID |
| `statut` | text | Workflow recording status |

Treat the Buffer post ID as a **submission record**, not proof the social platform delivered the post. Verify actual status with Buffer / the platform.

## Publishing safeguards

- Default template is `saveToDraft: true`, channel configurations disabled or preview-only.
- Daily trigger in the exported reference is intentionally disconnected.
- On production setups, apply *per-platform* checks, monitor workflow failures, and audit the queue before enabling unattended publication.
- Do not mark an API error as `COHERENT`, and do not silently normalize materially different grades or eligibility criteria.

## Release note

This repository holds a sanitized reference of n8n V4 from October 2026, plus a small fix preserving `needsInstagram` and `needsTikTok` metadata after reading the individual listing. The live n8n instance may have later changes; use a fresh authenticated export to synchronize it.