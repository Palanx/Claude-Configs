---
name: unity-editor-gotchas
description: >
  Unity Editor behaviours that look like code bugs but are not — environment symptoms
  that send people hunting through a codebase that was never wrong. Read BEFORE
  investigating rendering, lighting, shading, or reimport symptoms reported from the
  Editor, and whenever a symptom appears only in the Editor and not in a build:
  "todo se ve negro", "black meshes", "lighting broke", "solo pasa en el editor",
  "se rompió después de darle play", unexplained reimports, or a symptom that
  appeared without a code change.
---

# Unity Editor gotchas

Environment behaviour, not code. Do not chase these in the codebase.

The tell is always the same: a symptom that appears without a corresponding change, or
that only reproduces in the Editor. Check this list before opening a single script — each
entry here cost someone hours first.

## Everything renders black in the Editor (reviewed 2026-08-10)

Editing any asset or script while in Play Mode makes Unity reimport on focus return,
which shows up in `Editor.log` as `Asset Pipeline Refresh ... Initiated by RefreshV2`.
That reimport invalidates the lighting references of the scenes already loaded, so meshes
go black from that moment on.

Setting `Script Changes While Playing` to *Recompile After Finished Playing* does **not**
prevent it: that only defers assembly reload, not the asset import.

- **Fix while working:** turn off `Preferences > Asset Pipeline > Auto Refresh` during
  render QA, or just restart Play Mode.
- **Diagnosis first:** if black meshes show up, grep `Editor.log` for a `RefreshV2`
  before blaming the code.

It cost an afternoon once, chased through the scene unload path, until the timestamps in
the log gave it away — the reimport had happened right before the first black frame.

## Adding an entry

One `##` per symptom, phrased as the symptom rather than the cause — that is how it will
be searched for. State the mechanism, then what does *not* fix it (the plausible wrong
fix people try first), then the actual workaround and the log line that confirms the
diagnosis. Date it with the last time it was confirmed against a Unity version.
