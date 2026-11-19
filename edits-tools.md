# Edits tools: which tool for which job

Edits for Mac has an automation server built in (it names itself `metapilot-native`). It speaks MCP over HTTP on your
own Mac only (`http://127.0.0.1:19501/mcp` on the Macs we have seen), with a private access token. Its tool list follows
the screen: 32 tools on the home screen, 274 once a project is open in the editor. Read the live list and
`getCapabilities` at the start of every session: this page was checked call by call against Edits 456 (November 18,
2026, tool schema version "2026.10.19-1"), and Edits updates often.

Many schemas mark every argument as required, even when the tool only needs one of them. The analysis tools
(`findPauses`, `quietPoint`, `audioLevels`, `audioOnsets`) take a clip `id` OR a file `path`: send the `id` and
`"path": ""`. Calls can be slow: on a busy Mac the first `getCapabilities` took 34 seconds and a plain `seek` took over
a minute. Never treat a slow answer as a failure, never send the same edit twice because the first one is slow, and
give your MCP client a timeout of several minutes.

What has been run end to end on a real Mac (Edits 456, November 18, 2026): a new 9:16 project at 30 fps, importing a
Reel, words from `transcribeClip`, pauses, a cut with `quietPoint`, `splitAt` and `delete`, captions with a style and
a corrected word, a royalty-free song added by id, beat markers and a slip that put a downbeat on the cut, two sound
effects, and a 1080x1920 30 fps export whose length matched the project and kept every word outside the approved cut.
Text overlays, transitions and effects have NOT been run. Check each result as you go (read `clips` and `elements`
after every change).

Conventions from `getCapabilities`: time is in seconds on the project timeline unless an argument name says otherwise
(`...SourceSeconds` = seconds in the source file, `...Ms` = milliseconds). Canvas positions run 0 to 1 from the top left.

## Getting around

| Job | Tools | Notes |
|---|---|---|
| Where am I? | `appState` | `home`, `editor` or `loading` |
| List projects | `projects` | read only; never open one of theirs unless they asked |
| New project | `newProject`, then `setTitle` | always work in a new project |
| Close | `closeProject` | back to the home screen |
| Rules and catalogs | `getCapabilities` | conventions, catalog counts, recipes |
| What is on the timeline | `clips` (video), `elements` (everything) | read after every change |
| Select | `selectById {id}` | most editing tools act on the selection |
| Playhead | `seek {seconds}` | the argument is `seconds`, NOT `time` |
| Undo | `undo` | one step |

## Import and canvas

| Job | Tools | Notes |
|---|---|---|
| Bring in a video | `addMedia {path}` | copy the file into `~/Movies/Edits/Imports/` first: other folders failed with "Sandbox bookmark unavailable". The answer gives the new clip's `id` |
| 9:16 at 30 fps | `setCanvas {"aspect": "9:16", "frameRate": 30}` | new projects start at 16:9, 60 fps; set this before importing and cutting, so cuts snap to the 30 fps grid. `getCanvas` then reads 2160x3840 |

## Words, pauses and clean cut points

| Job | Tools | Notes |
|---|---|---|
| Words with times | `transcribeClip {"id", "language": "en", "mergeSplitWords": true, "background": false}` | all four are required. Returns `words` as `{text, start, end}` in timeline seconds, without adding captions. The skill's `words.json` uses `{w, s, e}`: convert each word. About 17 seconds for a 46-second clip. Its description mentions a `transcriptJumpCut` tool that Edits 456 does not have |
| Loudness of the voice and the background | `audioLevels {"id", "path": "", "start", "duration", "hop": 0.5}` | gives `noiseFloorDb` and `speechDb`. Read it before `findPauses` |
| Silences | `findPauses {"id", "path": "", "start", "duration", "minDuration": 0.6, "thresholdDb"}` | all six are required. Set `thresholdDb` about 3 dB above `audioLevels`' `noiseFloorDb`: with music under the voice the floor can sit near -20 dB, and -40 finds nothing. Quiet words also read as "pauses": keep only the part of each pause that no word from `transcribeClip` overlaps |
| Clean cut point | `quietPoint {"id", "path": "", "seconds", "window"}` | the quietest 10 ms within `seconds` plus or minus `window` (window is HALF the search width). It can land inside a word: search the middle of the gap between the last word you keep and the first word you cut, with `window` = half that gap |
| What is in a shot | `describeClip`, `detectCuts`, `detectShots` | optional |

## Cutting

| Job | Tools | Notes |
|---|---|---|
| Split at exact times | `splitAt {"id", "times": [a, b], "snap": "projectFrame"}` | both edges of a cut in one undo step. The answer lists the `pieces` with their times. EVERY piece gets a new id, the first one too, so the old clip id stops working. `projectFrame` moves each time onto the 1/30 s grid (5.78 became 5.767): check it is still between the two words. Do NOT use `splitClip`: it only splits at the clip's midpoint |
| Remove a piece | `selectById` the middle piece, then `delete` | with ripple on (the default) the gap closes by itself. Read `clips`: the next clip's `startTime` must equal the previous clip's `endTime` |
| Trim an end | `selectById`, `seek {seconds}`, then `trimStart` / `trimEnd` | a negative `trimClip` delta EXTENDED a clip once; avoid it |
| Move | `move`, `reorderClip` | to close a gap if one is left |
| Ripple settings | `rippleState` | read it before cutting. `tier1RippleEnabled: true` means a delete closes its gap |

## Captions

| Job | Tools | Notes |
|---|---|---|
| Make captions | `generateCaptions {"source": "video", "language": "en", "curseWords": true}` | from the project audio; run it AFTER the cuts. About 40 seconds for 35 seconds of speech; it answers with the phrase count |
| Read them | `captions` | `{captions: [...]}`, each with `id`, `text`, `startTime`, `endTime` and its words in `tokens` |
| Pick a look | `listCaptionStyles`, then `setCaptionStyle {"style": "<name>"}` | 73 styles in Edits 456 (`getCapabilities` still says 39). With no `id` it styles every caption at once |
| Fix a word | `editCaption {"id", "text"}`, `splitCaption`, `mergeCaptions` | `editCaption` replaces the whole phrase's text and keeps its timing. Names are often misheard |
| Highlight words | `setCaptionHighlights`, `setCaptionHighlightColor` | |

## Music

| Job | Tools | Notes |
|---|---|---|
| Find (royalty-free) | `listMusicTracks {"tab": "royalty-free"}` | 30 tracks, each with `id`, `title`, `artistName`, `duration` and `license.type`. Tracks have no mood or tempo field, and a mood word with `"license": "royaltyFree"` found almost nothing ("chill", "calm" and "ambient" 0 tracks, "lofi" 1), so show the person titles and lengths from the tab and let them listen |
| Find (any license) | `listMusicTracks {"query": "<mood>"}` | `searchMusic` returns titles only, with no id and no license: don't use it to choose |
| Listen | `previewMusicTrack {"track": "<id>"}` | plays it in Edits' mini player for the person |
| Add | `seek {"seconds": 0}`, then `addSongToComposition {"audioAssetId": "<id from listMusicTracks>"}` | ALWAYS pass `audioAssetId`. `track` is a title search that adds the first hit: an id passed as `track` added a different, licensed song. Check the answer's `title` and `license.type` before going on. The song lands at the playhead and is trimmed to the project's length |
| Fit | `slipClip`, `trimEnd`, `setVolume {"id", "volume"}`, `setFade {"id", "fadeOut"}` | volume is a gain from 0 to 10, where 1 = unchanged; under a voice, start near 0.2 |
| Window ready? | `licensedMusicStatus` | before export: `windowComplete` must be true. Royalty-free tracks also arrive as a window of the song (`importPath: "obfuscatedWindow"`), not the full file |

Edits' music comes from Meta's library. Treat it as licensed for Instagram and Meta's apps. For a file that will also go
to TikTok or YouTube, use the royalty-free tab and tell the person to check the terms themselves.

## Beat timing

| Job | Tools | Notes |
|---|---|---|
| Beat times | `setBeatDensity {"id", "density": "medium"}`, then `beatMarkers {"id"}` | on the music element. Library songs arrive with beats already found (149 on a 60-second track). `medium` keeps the downbeats (one every 1.6 s on that track). Each marker has `timelineSeconds`, `sourceSeconds`, `isDownbeat` and `isStrong`. `estimatedBpm` counts the VISIBLE markers, not the song's tempo |
| Line a beat up | `slipClip {"id", "deltaSeconds"}` | a positive delta starts the song later, so every beat moves EARLIER on the timeline by that much (snapped to 1/30 s). To put a beat at time t, slip by (beat's timeline time minus t). Read `beatMarkers` again afterwards |
| No markers? | `audioOnsets {"id", "path": "", "start", "duration"}` | hits plus the tempo that fits them; make a beat grid from it |
| Add or remove a marker | `addBeatMarker`, `removeBeatMarker` | |

There is no one-step "sync to the beat" tool. Use the beat times yourself: slip the music so a strong beat lands on the
first cut or the payoff, and put a sound effect on a beat only when one falls within about 0.15 seconds of its moment.
Never move a cut inside speech to chase a beat, and never move an effect off the word it belongs to.

## Sound effects

| Job | Tools | Notes |
|---|---|---|
| Find | `searchSoundEffects {"query": "whoosh"}` | always search with a word (whoosh, pop, click, ding); an empty query returns nothing. `category` is optional (one of a fixed list such as `transitions` or `comedy`): leave it out to search everything. Each result has `id`, `title` and `durationSeconds` |
| Add | `seek {"seconds"}`, then `addSoundEffect {"effect": "<id>"}` | it lands at the playhead. The answer gives the new element's `id` and `title`. Its length is rounded up to whole frames (a 0.24 s whoosh became 0.33 s). It shares the music's audio track and leaves the music as it was; read `elements` to check |
| Level | `setVolume {"id", "volume": 0.5}` on the effect | under the voice |

## Text

`addText {atSeconds, duration, text}`, `applyTextPreset`, `setTextAnimation`: for a title or a number on screen.

## Export

| Job | Tools | Notes |
|---|---|---|
| Settings | `setExportSettings {"resolution": "1080x1920", "resolutionType": "HD", "aspectRatio": "nineBySixteen", "quality": "high", "frameRate": 30, "dynamicRange": "SDR"}` | the FULL row: a missing field is refused. `resolution` is width x height. Leave `codec` out: Edits exports HEVC only and refuses h264. The answer echoes the row back |
| Go | `export {"path": "<absolute path in ~/Movies/Edits/Exports/>"}` | returns at once |
| Wait | `exportStatus` | poll every 10 seconds or so: `state` goes `preparing`, `exporting` (with `progress` 0 to 1), then `complete`. A 4-minute 1080p video took about a minute on a quiet Mac; a 35-second one took 8.5 minutes on a busy one |
| Keep it | `saveProject` | stores the project in Edits' library; answers `saved: true` and the element count |

The 9:16 export came out HEVC 1080x1920 at 30 fps, SDR (bt709), AAC stereo at 44.1 kHz, exactly the project's length.

## Tools to avoid

- `dragSpeedPanelCurve` and `selectSpeedPanelPreset`: marked deprecated, and they crashed Edits. Use `setSpeedCurve`.
- Panel plumbing (open or dismiss a panel, `editValueField`): use the direct tools above instead.
- `deleteProject`: never.
