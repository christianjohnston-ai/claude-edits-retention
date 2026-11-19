# Gotchas and footguns

Most of these were seen on a real Mac driving Edits through its automation server (November 2026). The last three
(reading graphs, thin data, Edits updates) come from building the mind reader and from how often Edits changes, not
from a failed run.

## Claude Code can't see Edits

**Symptom:** no `metapilot-native` tools, or "ECONNREFUSED" for that server.
**Why it happens:** Edits must be open with the Automation Server switched on, and Claude Code loads tools when a session
starts. A session opened before the setup command, or before Edits was opened, has none of them.
**Fix:** open Edits, check Settings, then Automation, then start a NEW Claude Code session (`/exit`, then `claude`).

## The tools vanished in the middle of a session

**Symptom:** calls that worked a minute ago now fail, or the tools are gone.
**Why it happens:** Edits quit (it has quit on its own once), and its tools go with it. They come back only after Edits
is open again and the session reconnects (the person can type `/mcp` and reconnect, or start a new session).
**Fix:** ask the person to reopen Edits, reconnect, then read `appState` and `elements` before carrying on. Nothing is
lost: the project is saved in Edits.

## Most tools are missing

**Symptom:** only about 30 tools, none for cutting or captions.
**Why it happens:** the tool list follows Edits' screen. The home screen has about 30; the editor about 270.
**Fix:** `newProject` (or `openProject` on a project the person named), then list the tools again.

## "Sandbox bookmark unavailable" on import

**Symptom:** `addMedia` fails for a file in the person's own folders.
**Why it happens:** Edits is sandboxed and can only read some folders.
**Fix:** copy the video into `~/Movies/Edits/Imports/` and import it from there.

## A tool says "ArgumentParser.CommandError"

**Symptom:** a call fails with that message.
**Why it happens:** an argument has the wrong name. `seek` takes `seconds`, not `time`. Many schemas also mark every
argument as required, even filters (send an empty string where you mean "any").
**Fix:** read the tool's schema and call it again with its exact argument names.

## A trim made the clip LONGER

**Symptom:** after `trimClip` with a negative amount, the clip grew.
**Why it happens:** unknown; it happened once and is not worth finding out on someone's video.
**Fix:** use `selectById`, `seek {seconds}`, then `trimStart` or `trimEnd`, and `splitAt` for cuts.

## `splitClip` cut in the wrong place

**Symptom:** the split landed in the middle of the clip, not at the time you wanted.
**Why it happens:** `splitClip` only splits at the clip's midpoint, even though Edits' own beat recipe names it.
**Fix:** `splitAt {id, times}`.

## The export is 60 fps, or refused

**Symptom:** `setExportSettings` refuses ("incomplete preset row", "resolution must be WxH", "codec h264 is not
supported"), or the file comes out at 60 fps.
**Why it happens:** it needs the full row (resolution as width x height, resolutionType, aspectRatio, quality), exports
HEVC only, and uses the PROJECT's frame rate. New projects start at 60 fps.
**Fix:** set the canvas to 30 fps before cutting, then send the full export row from edits-tools.md.

## Export returns at once but there is no file yet

**Symptom:** `export` answers immediately.
**Why it happens:** exporting runs in the background.
**Fix:** poll `exportStatus` until it says complete. A 4-minute 1080p video took about a minute.

## The export is louder than expected at the peaks

**Symptom:** the exported file's true peak was slightly above 0 dB once, even when the input was quieter.
**Why it happens:** Edits' audio encoder. Not something the automation can change.
**Fix:** keep music and effects well under the voice; if the person cares about exact loudness, measure the export.

## Two tools crash Edits

**Symptom:** Edits closes.
**Why it happens:** `dragSpeedPanelCurve` and `selectSpeedPanelPreset` are marked deprecated and crashed it twice.
**Fix:** never call them. Use `setSpeedCurve`.

## Music sounds wrong for the mood

**Symptom:** a "chill" search brings back something loud.
**Why it happens:** tracks have no mood or tempo field, so searches match titles and tags loosely, and the royalty-free
tab is small.
**Fix:** search a few mood words, preview the top picks with the person, and let them choose.

## Edits' music may not be cleared outside Instagram

**Symptom:** the person wants to post the same file on TikTok or YouTube.
**Why it happens:** Edits' library is Meta's. Its terms limit audio to Meta's own apps, and licensed songs are what
other platforms' copyright checks catch.
**Fix:** use the royalty-free tab for anything that leaves Instagram, and tell the person to check the terms themselves.

## The person's own project changed

**Symptom:** a project's size or state changed after you opened it.
**Why it happens:** opening a project touches its cache.
**Fix:** never open, edit or delete the person's projects. Work in a new project; if they want one edited, they
duplicate it first.

## A retention graph can't be read precisely

**Symptom:** you're unsure whether a point is 42% or 45%.
**Why it happens:** a screenshot is a picture of a chart.
**Fix:** read your best value and move on. The method ignores small differences; only big, repeated patterns become
verdicts.

## Almost everything says "not enough data yet"

**Symptom:** the profile table is mostly empty.
**Why it happens:** too few past videos, too few views per video, or few beats of each label.
**Fix:** say so plainly. Add more past videos (10+ is good), and keep the habit of adding each new one in Part 4.

## Edits updated and a tool changed

**Symptom:** a tool from edits-tools.md is missing or takes different arguments.
**Why it happens:** Edits updates often; this skill was written against version 456.
**Fix:** trust the live tool list and `getCapabilities` over this folder, and tell the person what changed.
