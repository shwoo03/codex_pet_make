---
name: codex-pet-best-practices
description: "Use this whenever the user wants to create, repair, QA, package, tune animation speed, organize, or refresh a custom Codex pet from images, references, mascot ideas, or existing pet artifacts. This skill captures practical Codex pet best practices including the fixed atlas contract, identity-preserving prompts, smallest-scope row repair, validation with hatch-pet scripts, local packaging under CODEX_HOME, and selecting custom pets in Settings Appearance Pets."
---

# Codex Pet Best Practices

Use this skill as the practical checklist for custom Codex pets. For production generation, repair, deterministic processing, and packaging, prefer the installed `hatch-pet` skill and its scripts:

```bash
SKILL_DIR="${CODEX_HOME:-$HOME/.codex}/skills/hatch-pet"
```

Read `${CODEX_HOME:-$HOME/.codex}/skills/hatch-pet/SKILL.md` and the relevant files under `references/` before starting a pet run. This skill summarizes the habits that prevent broken pets; it does not replace the hatch-pet pipeline.

## Fixed App Contract

Codex pets use one fixed atlas. Do not add arbitrary states, rows, columns, frames, labels, timing metadata, gutters, or custom manifest fields and expect the app to play them.

- Atlas: `1536x1872`, transparent-capable PNG or WebP.
- Grid: 8 columns x 9 rows.
- Cell: `192x208`.
- Unused cells after a row's final frame: fully transparent.
- Package: `${CODEX_HOME:-$HOME/.codex}/pets/<pet-id>/pet.json` plus `spritesheet.webp`.
- Selection: refresh or select custom pets in Codex at `Settings > Appearance > Pets`.

Manifest shape:

```json
{
  "id": "pet-id",
  "displayName": "Pet Name",
  "description": "One short sentence.",
  "spritesheetPath": "spritesheet.webp"
}
```

Rows and timing are fixed by the app:

| Row | State | Used columns | Durations |
| --- | --- | ---: | --- |
| 0 | `idle` | 0-5 | 280, 110, 110, 140, 140, 320 ms |
| 1 | `running-right` | 0-7 | 120 ms each, final 220 ms |
| 2 | `running-left` | 0-7 | 120 ms each, final 220 ms |
| 3 | `waving` | 0-3 | 140 ms each, final 280 ms |
| 4 | `jumping` | 0-4 | 140 ms each, final 280 ms |
| 5 | `failed` | 0-7 | 140 ms each, final 240 ms |
| 6 | `waiting` | 0-5 | 150 ms each, final 260 ms |
| 7 | `running` | 0-5 | 120 ms each, final 220 ms |
| 8 | `review` | 0-5 | 150 ms each, final 280 ms |

For animation speed tuning, do not change the frame count or durations. Tune perceived speed through motion amplitude, pose spacing, easing implied by the drawings, and loop smoothness. Smaller pose deltas feel calmer; larger clear deltas feel faster. Avoid motion trails, speed lines, shadows, dust, and detached effects as fake speed cues.

## Workflow Checklist

1. Identify the pet id, display name, source references, required traits, forbidden traits, and style.
2. Preserve identity constraints explicitly: examples include `no eyebrows`, required eyebrows, exact props, crown shape, ear color, body markings, material, and expression language.
3. Organize all run artifacts under one pet id folder before generating or repairing.
4. Use `hatch-pet` to prepare the run and follow its `imagegen-jobs.json` dependencies.
5. Generate or repair the smallest useful visual unit: base only when the base identity is wrong; otherwise a single row.
6. Process rows with hatch-pet scripts, produce `spritesheet.webp`, `validation.json`, `review.json`, contact sheet, and preview GIFs.
7. Visually inspect the contact sheet and per-row previews. Deterministic validation is necessary but not sufficient.
8. Package `pet.json` and `spritesheet.webp` into `${CODEX_HOME:-$HOME/.codex}/pets/<pet-id>/`.
9. Tell the user to refresh/select the custom pet in `Settings > Appearance > Pets`.

## Artifact Layout

Keep artifacts grouped by pet id so repairs and installs are traceable:

```text
artifacts/pets/<pet-id>/
+-- source/
|   +-- reference.png
+-- final/
|   +-- spritesheet.png
|   +-- spritesheet.webp
+-- qa/
|   +-- validation.json
|   +-- review.json
|   +-- contact-sheet.png
|   +-- run-summary.json
|   +-- previews/
|       +-- idle.gif
|       +-- running-right.gif
|       +-- ...
+-- install/
    +-- pet.json
    +-- spritesheet.webp
```

The installed package lives separately:

```text
${CODEX_HOME:-$HOME/.codex}/pets/<pet-id>/
+-- pet.json
+-- spritesheet.webp
```

## Prompt Guidance For Reference-Image Pets

Make the base image the canonical identity reference, then ground each row in that base and the user's reference images when the generation path supports references.

Strong pet prompts:

- Describe a compact full-body sprite that fits inside a `192x208` cell.
- Lock stable traits: silhouette, face, proportions, palette, material, props, markings, and expression style.
- State required absences as identity constraints, not afterthoughts: `no eyebrows`, `no hat`, `no text`, `no shadow`.
- Keep row action specific: idle breathing, rightward drag movement, greeting wave, jump arc, failed reaction, waiting-for-input pose, active task focus, review focus.
- Prefer pose and silhouette changes over effects.
- Ask for flat chroma background for extraction, with no guide marks, visible grid, labels, scenery, floor, or text.

Avoid:

- Detached sparkles, motion arcs, speed lines, dust, loose tears, floating symbols, punctuation, speech bubbles, UI, code, logos, or text.
- Cast shadows, drop shadows, contact shadows, glows, halos, smears, blur, floor patches, or landing marks.
- Chroma-key-adjacent colors in the pet, props, highlights, or effects.
- New props or accessories that are not part of the base identity.
- Whole-atlas regeneration when one row is failing.

## Repair Playbook

Repair the smallest failing scope first:

1. Fix deterministic extraction before regenerating art when previews show size popping but the source strip itself is stable.
2. Regenerate one bad row when the row has identity drift, clipped poses, wrong state semantics, guide marks, backgrounds, detached effects, or inert motion.
3. Regenerate the base only when the canonical pet identity is wrong.
4. Regenerate the whole atlas only when identity or layout is broadly broken.

Row-specific notes:

- `idle`: calm blink, breathing, or tiny bob. It must not read as waving, running, jumping, reviewing, working, or reacting dramatically.
- `running-right` and `running-left`: directional movement only. Mirror `running-left` only when flipping preserves identity, prop meaning, side markings, lighting, and temporal cadence.
- `running`: active task work, thinking, typing, scanning, or focused effort. It is not foot-running.
- `waiting`: expectant user-input or approval pose, distinct from idle and review.
- `review`: focused inspection through posture, eyes, head tilt, or hand/paw position. Avoid adding papers, magnifiers, symbols, or code unless already part of the base pet.
- `failed`: readable error or deflated reaction; attached opaque tears, stars, or smoke can work, but detached effects should fail QA.

When a row feels too fast or too slow, regenerate or edit the row prompt for amplitude and pose spacing. Because timings are fixed, speed changes come from the drawings: calmer rows use smaller deltas and steadier baselines; energetic rows use clearer pose changes while staying inside the cell.

## Validation Commands

Use the hatch-pet scripts instead of hand-rolled geometry checks:

```bash
SKILL_DIR="${CODEX_HOME:-$HOME/.codex}/skills/hatch-pet"
RUN_DIR="/absolute/path/to/run"

python "$SKILL_DIR/scripts/extract_strip_frames.py" \
  --decoded-dir "$RUN_DIR/decoded" \
  --output-dir "$RUN_DIR/frames" \
  --states all \
  --method auto

python "$SKILL_DIR/scripts/inspect_frames.py" \
  --frames-root "$RUN_DIR/frames" \
  --json-out "$RUN_DIR/qa/review.json" \
  --require-components

python "$SKILL_DIR/scripts/compose_atlas.py" \
  --frames-root "$RUN_DIR/frames" \
  --output "$RUN_DIR/final/spritesheet.png" \
  --webp-output "$RUN_DIR/final/spritesheet.webp"

python "$SKILL_DIR/scripts/validate_atlas.py" \
  "$RUN_DIR/final/spritesheet.webp" \
  --json-out "$RUN_DIR/final/validation.json"

python "$SKILL_DIR/scripts/make_contact_sheet.py" \
  "$RUN_DIR/final/spritesheet.webp" \
  --output "$RUN_DIR/qa/contact-sheet.png"

python "$SKILL_DIR/scripts/render_animation_previews.py" \
  --frames-root "$RUN_DIR/frames" \
  --output-dir "$RUN_DIR/qa/previews"
```

If preview GIFs show extraction-induced size popping and the source strips are visually stable, rerun extraction deliberately with stable slots, then re-run inspection, composition, validation, contact sheet, and previews:

```bash
python "$SKILL_DIR/scripts/extract_strip_frames.py" \
  --decoded-dir "$RUN_DIR/decoded" \
  --output-dir "$RUN_DIR/frames" \
  --states all \
  --method stable-slots

python "$SKILL_DIR/scripts/inspect_frames.py" \
  --frames-root "$RUN_DIR/frames" \
  --json-out "$RUN_DIR/qa/review.json" \
  --require-components \
  --allow-stable-slots
```

Package after validation:

```bash
PET_ID="$(jq -r '.pet_id' "$RUN_DIR/pet_request.json")"
DISPLAY_NAME="$(jq -r '.display_name' "$RUN_DIR/pet_request.json")"
DESCRIPTION="$(jq -r '.description' "$RUN_DIR/pet_request.json")"
PET_DIR="${CODEX_HOME:-$HOME/.codex}/pets/$PET_ID"

mkdir -p "$PET_DIR"
cp "$RUN_DIR/final/spritesheet.webp" "$PET_DIR/spritesheet.webp"
jq -n \
  --arg id "$PET_ID" \
  --arg displayName "$DISPLAY_NAME" \
  --arg description "$DESCRIPTION" \
  '{id: $id, displayName: $displayName, description: $description, spritesheetPath: "spritesheet.webp"}' \
  > "$PET_DIR/pet.json"
```

Acceptance requires:

- `final/validation.json` reports `ok: true`.
- `qa/review.json` has no errors.
- Contact sheet shows the same pet identity in every row.
- Preview GIFs have correct state semantics, no wrong facing direction, no reversed directional cadence, no extraction-induced popping, and no inert idle loop.
- Installed package contains exactly the matching `pet.json` and `spritesheet.webp`.
