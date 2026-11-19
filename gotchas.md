# Gotchas and footguns

Most of these were seen on a real Mac driving Edits through its automation server (November 2026), most of them in
the first full run of Part 2 and Part 3 on Edits 456 (November 18, 2026). The entries on reading graphs and on Edits
updates come from building the mind reader and from how often Edits changes, not from a failed run.

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

## Every call takes a minute

**Symptom:** a plain `seek` or `addSoundEffect` takes 30 to 80 seconds, or the MCP client times out.
**Why it happens:** Edits answers only when it has done the work, and on a busy Mac (other heavy programs running)
that can take over a minute. In the first full run, Edits itself sat near 1% CPU while the Mac's load was very high.
**Fix:** wait. Never send the same edit again because the first is slow: you would get two copies. Give the MCP client
a timeout of several minutes, and close other heavy programs before a long edit.

## `words.json` has the wrong field names

**Symptom:** the beats you build from Edits' words come out empty, or a script cannot find `w`, `s` and `e`.
**Why it happens:** `transcribeClip` returns each word as `{text, start, end}`. The skill's files use
`{w, s, e}` (the teardown's format).
**Fix:** convert each word when you save `words.json` or `words_after.json`.

## `findPauses` finds nothing, or finds "pauses" in the middle of words

**Symptom:** with `thresholdDb` -40 it returns no pauses; with a higher threshold it returns stretches that cover
spoken words.
**Why it happens:** a posted Reel usually has music under the voice, so it is never truly silent: its noise floor can
sit near -20 dB. Raise the threshold and quiet words, such as the soft end of a word, fall under it too.
**Fix:** read `audioLevels` first and set `thresholdDb` about 3 dB above its `noiseFloorDb`. Then keep only the part of
each pause that no word from `transcribeClip` overlaps. If nothing of 0.6 seconds is left, the Reel has no pause
beats; that is normal for a video that was already cut tight.

## `quietPoint` chose a point inside a word

**Symptom:** the cut point it returns is before the end of the last word you meant to keep (about 0.2 seconds early in
the test), so the cut would clip the word.
**Why it happens:** `window` is half the search width: the quietest 10 ms anywhere within `seconds` plus or minus
`window` wins, and the closed mouth inside a word ("-ed", "-t") can be the quietest spot.
**Fix:** search the middle of the gap between the two words, with `window` set to half that gap.

## The clip id stopped working after a split

**Symptom:** after `splitAt`, a call with the clip's old id fails or finds nothing.
**Why it happens:** `splitAt` gives every piece a new id, including the first piece.
**Fix:** use the `pieces` in `splitAt`'s answer (or read `clips` again) and take the ids from there.

## The wrong song was added, and it is licensed

**Symptom:** you chose a royalty-free track, but `addSongToComposition` answered with a different title and
`license.type` "licensed".
**Why it happens:** its `track` argument is a title search that adds the first hit. A track id passed as `track` is
searched as text, and some other song comes first.
**Fix:** always pass `audioAssetId` (the id from `listMusicTracks`), then check the answer's `title` and `license.type`.
If it is wrong, `selectById` the new music element, `delete` it, and add again by id.

## Mood searches find no royalty-free music

**Symptom:** `listMusicTracks {"query": "chill", "license": "royaltyFree"}` returns 0 tracks, and `searchMusic` returns
titles with no id or license.
**Why it happens:** tracks have no mood field, the royalty-free tab is small (30 tracks), and `searchMusic` only
returns titles.
**Fix:** list the whole tab with `listMusicTracks {"tab": "royalty-free"}` and choose from titles, lengths and a
preview.

## `estimatedBpm` is far too slow

**Symptom:** `beatMarkers` reports `estimatedBpm` 37.5 for an upbeat song.
**Why it happens:** it counts the markers left visible by `setBeatDensity` (at "medium", the downbeats), not the song's
tempo.
**Fix:** use the marker times themselves. For the tempo, use `audioOnsets`.

## A tool's description names a tool that does not exist

**Symptom:** `transcribeClip`'s description talks about `transcriptJumpCut`, but no such tool is in the list.
**Why it happens:** the descriptions are written for Edits' own builds and run ahead of (or behind) the tool list
you get.
**Fix:** trust the tool list. Cut with `splitAt` and `delete`.

## The export is 60 fps, or refused

**Symptom:** `setExportSettings` refuses ("incomplete preset row", "resolution must be WxH", "codec h264 is not
supported"), or the file comes out at 60 fps.
**Why it happens:** it needs the full row (resolution as width x height, resolutionType, aspectRatio, quality), exports
HEVC only, and uses the PROJECT's frame rate. New projects start at 60 fps.
**Fix:** set the canvas to 30 fps before cutting, then send the full export row from edits-tools.md.

## Export returns at once but there is no file yet

**Symptom:** `export` answers immediately, and `exportStatus` sits at progress 0 for a minute.
**Why it happens:** exporting runs in the background, and it starts in a `preparing` state before `exporting`.
**Fix:** poll `exportStatus` until `state` is `complete`. A 4-minute 1080p video took about a minute on a quiet Mac; a
35-second one took 8.5 minutes on a busy one.

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

**Symptom:** the profile table is mostly empty, even with 28 past videos.
**Why it happens:** too few past videos, too few views per video, or few beats of each label. A verdict also needs the
typical video AND the total across all videos to be at least 10% off the usual pace, and the biggest videos dominate
the total. So a kind of moment can be faster than usual in most videos and still get no verdict, because one or two
big videos pull the total under 1.1. With 20 to 30 videos of a few hundred views each, an all-empty table is normal.
**Fix:** say so plainly. Add more past videos (10+ is good), and keep the habit of adding each new one in Part 4.

## Payoff, after-the-payoff and call to action never get a verdict

**Symptom:** those three lines show only 1 to 5 videos scored, even though every video has them.
**Why it happens:** they come late, and the script skips a beat when fewer than 10% of the starting viewers (or fewer
than 20 people) are still watching. On most accounts, far fewer than 10% are still there by the payoff.
**Fix:** nothing to fix: that rule keeps a handful of leavers from deciding a verdict. Tell the person, and rely on the
after-the-payoff check for those moments.

## A past Reel has almost no words

**Symptom:** its `words.json` holds one or two words for a long video.
**Why it happens:** the video's speech was not transcribed (music only, or the transcription failed). Its beats would
be one long beat with the wrong label.
**Fix:** skip it in Part 2 (fewer than one word per 2 seconds of video), and say so.

## A sentence runs 30 seconds

**Symptom:** splitting `words.json` at full stops gives a beat of 30 seconds or more.
**Why it happens:** some transcripts lose their punctuation for long stretches.
**Fix:** break long stretches at commas, or before a capital letter that starts a lost sentence, or every 6 seconds.
Beats are usually 1 to 6 seconds.

## Edits updated and a tool changed

**Symptom:** a tool from edits-tools.md is missing or takes different arguments.
**Why it happens:** Edits updates often; this skill was written against version 456.
**Fix:** trust the live tool list and `getCapabilities` over this folder, and tell the person what changed.
