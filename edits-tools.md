# Edits tools: which tool for which job

Edits for Mac has an automation server built in (it names itself `metapilot-native`). It speaks MCP over HTTP on your
own Mac only (`http://127.0.0.1:19501/mcp` on the Macs we have seen), with a private access token. Its tool list follows
the screen: about 30 tools on the home screen, about 270 once a project is open in the editor. Read the live list and
`getCapabilities` at the start of every session: this page comes from the tool list of Edits 454 (November 8, 2026, tool
schema version "2026.10.19-1"; version 456 was out ten days later) and Edits updates often.

What has actually been run end to end on a real Mac so far: making a project, importing a video, finding and placing a
library song at exact points and volumes, and exporting (at 16:9). Cutting with `splitAt` and `delete`, captions, sound
effects, beat markers, text and a 9:16 export are in Edits' own tool list but have not been run end to end, so check
each result as you go (read `clips` and `elements` after every change).

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
| Bring in a video | `addMedia {path, ...}` | copy the file into `~/Movies/Edits/Imports/` first: other folders failed with "Sandbox bookmark unavailable" |
| 9:16 at 30 fps | `setCanvas {aspect, frameRate}` (or `setAspectRatio`) | new projects start at 60 fps; set 30 before cutting, so cuts snap to the 30 fps grid |

## Words, pauses and clean cut points

| Job | Tools | Notes |
|---|---|---|
| Words with times | `transcribeClip {id, language, ...}` | returns each word's start and end without adding captions |
| Silences | `findPauses {id, minDuration, thresholdDb, ...}` | for pause flags |
| Clean cut point | `quietPoint {id, seconds, window}` | the quietest 10 ms near a time: cut there, between words |
| What is in a shot | `describeClip`, `detectCuts`, `detectShots` | optional |

## Cutting

| Job | Tools | Notes |
|---|---|---|
| Split at exact times | `splitAt {id, times, snap}` | many times in one undo step. Do NOT use `splitClip`: it only splits at the clip's midpoint |
| Remove a piece | `selectById` the piece, then `delete` | then read `clips` and check for a gap |
| Trim an end | `selectById`, `seek {seconds}`, then `trimStart` / `trimEnd` | a negative `trimClip` delta EXTENDED a clip once; avoid it |
| Move | `move`, `reorderClip` | to close a gap if one is left |
| Ripple settings | `rippleState` | read it before cutting so you know whether deletes close gaps |

## Captions

| Job | Tools | Notes |
|---|---|---|
| Make captions | `generateCaptions {source, language, curseWords}` | from the project audio; run it AFTER the cuts |
| Pick a look | list the caption styles in the catalog, then `setCaptionStyle {style}` | about 40 styles |
| Fix a word | `editCaption`, `splitCaption`, `mergeCaptions` | names are often misheard |
| Highlight words | `setCaptionHighlights`, `setCaptionHighlightColor` | |

## Music

| Job | Tools | Notes |
|---|---|---|
| Find | `searchMusic {query}`, `browseMusicCategories`, `listMusicTracks`, `getMusicTrackDetails` | tracks have no mood or tempo field: search with mood words, then listen |
| Listen | `previewMusicTrack {track}` | let the person hear the top picks |
| Add | `addSongToComposition {track, audioAssetId}` | royalty-free tracks arrive as the full file; licensed songs as a window of the song |
| Fit | `slipClip`, `trimEnd`, `setVolume {id, volume}`, `setFade {id, fadeIn, fadeOut}` | volume is a gain from 0 to 10, where 1 = unchanged; under a voice, start near 0.2 |
| Licensed window ready? | `licensedMusicStatus` | before export |

Edits' music comes from Meta's library. Treat it as licensed for Instagram and Meta's apps. For a file that will also go
to TikTok or YouTube, use the royalty-free tab and tell the person to check the terms themselves.

## Beat timing

| Job | Tools | Notes |
|---|---|---|
| Beat times | `setBeatDensity {id, density}`, then `beatMarkers {id}` | on the music element; returns timeline and source times |
| No markers? | `audioOnsets {id, ...}` | hits plus the tempo that fits them; make a beat grid from it |
| Add or remove a marker | `addBeatMarker`, `removeBeatMarker` | |

There is no one-step "sync to the beat" tool. Use the beat times yourself: slip the music so a strong beat lands on the
payoff, and place sound effects and text pop-ins on beats. Never move a cut inside speech to chase a beat.

## Sound effects

| Job | Tools | Notes |
|---|---|---|
| Find | `searchSoundEffects {query, category}` | always search with a word (whoosh, pop, click, ding); an empty query returns nothing; send an empty string for category if you mean any |
| Add | `seek {seconds}`, then `addSoundEffect {effect}` | `effect` = an id or title from the search; read `elements` to check where it landed |
| Level | `setVolume` on the effect | under the voice |

## Text

`addText {atSeconds, duration, text}`, `applyTextPreset`, `setTextAnimation`: for a title or a number on screen.

## Export

| Job | Tools | Notes |
|---|---|---|
| Settings | `setExportSettings` with the FULL row | `resolution` "1080x1920" (width x height), `resolutionType` "HD", `aspectRatio` "nineBySixteen", `quality` "high", `frameRate` 30, `dynamicRange` "SDR", codec HEVC (Edits refuses h264). A missing field is refused |
| Go | `export {path}` | returns at once |
| Wait | `exportStatus` | poll until it says complete; a 4-minute 1080p video took about a minute |

## Tools to avoid

- `dragSpeedPanelCurve` and `selectSpeedPanelPreset`: marked deprecated, and they crashed Edits. Use `setSpeedCurve`.
- Panel plumbing (open or dismiss a panel, `editValueField`): use the direct tools above instead.
- `deleteProject`: never.
