---
name: editing-reels-for-retention
description: Use when the person wants Claude to edit a short vertical video (an Instagram Reel, a TikTok, a YouTube Short) in Instagram's Edits app for Mac, for example "edit this Reel in Edits", "add music and captions", "add sound effects", "time it to the beat", "cut the boring parts", "where will people get bored", "use the mind reader", "learn my retention", "update my viewer profile", or "grade my last Reel". Drives Edits' own built-in automation server (an MCP server called metapilot-native) to import the video, cut it, add captions, music and sound effects, line moments up with the beat, and export a 9:16 file. Its "mind reader" learns from the person's own past retention curves (screenshots of Instagram or TikTok retention graphs, or exact data from the retention-teardown skills) which kinds of moments make THEIR viewers leave faster than usual, then flags those moments in a new cut and offers to cut them, one approval at a time. Never posts anything.
---

# Editing Reels for retention, in Instagram's Edits app

Version 1, November 18, 2026. If an older copy of this skill exists, this one replaces it.

Three things this skill does, each one only when the person asks:
1. **Set up** (once): check the Mac, connect Claude Code to Edits, make the data folder. Part 1.
2. **Learn my viewers** (the mind reader): read the person's past retention curves (best: from their own Instagram
   retention teardown, `/retention-teardown-instagram`; otherwise from screenshots) and find which kinds of moments make
   their viewers leave faster than their own average. Part 2. Refresh it every few weeks.
3. **Edit a new video**: flag the stretches the mind reader expects to lose people, cut the ones the person approves,
   then add captions, music, sound effects and beat timing in Edits, and export. Part 3.

Part 4 grades the flags against the real curve once the video has been up 48 hours, so the mind reader gets checked
against reality instead of being trusted.

**At the start of Part 2, 3 or 4 in any new session:** call `appState`. If the editing tools are missing once a project
is open, ask the person to leave that project open in Edits and start a new Claude Code session (the tool list follows
Edits' screen and loads when a session starts). Before they restart in the middle of an edit, tell them the line to
type afterwards: "continue Part 3 of the editing-reels-for-retention skill for <the video's name>". Everything you need
to resume is in `~/EditsRetention/new/<name>/` and in the open Edits project.

Talk to the person in plain words. Many have never used a terminal. Say what you are about to do, do it, show the
result, and wait for their OK before the next step. Read [edits-tools.md](./edits-tools.md) before your first Edits call,
[mind-reader-method.md](./mind-reader-method.md) before Part 2 or Part 3, and [gotchas.md](./gotchas.md) when anything
behaves oddly.

## Ground rules (never break these)

- **Never post or publish.** Edits' automation has no publish tool; keep it that way. You export a file. The person
  posts it.
- **Never open, change or delete the person's own Edits projects.** Every edit happens in a NEW project you create. If
  they want an existing project edited, they duplicate it in Edits first and tell you its name.
- **Ask before every cut.** Show the stretch (its code, the seconds, the exact words) and cut only what they approve.
  Cuts change what the person says; that is their call.
- **Never forecast a percentage.** No "this will keep 62% at 3 seconds". The mind reader flags stretches and gives the
  evidence from the person's own past videos. In the author's own test, the plain average of a creator's past curves
  predicted a new curve better than an AI's guess, so the honest output is "these moments lost your viewers faster
  before", not a number.
- **The Edits access token stays out of the chat.** The person pastes Edits' own setup command into a Terminal window
  themselves, outside Claude Code. Never ask for the token, print it, or put it in a file you write. If they paste a
  command containing `Bearer` into this chat by mistake: do not repeat it and do not run it; tell them to run it in a new
  Terminal window instead, and to switch Edits' Automation Server off and on again if they want a fresh token (the server
  only listens on their own Mac).
- **Read Edits' tool list at the start of every session** (`getCapabilities`, and each tool's schema before its first
  call). Edits updates often and tool names and arguments change.
- **Their analytics stay in `~/EditsRetention` on their Mac.** You read the screenshots and transcripts in this chat to do
  the work, like any file they show you; never upload or copy them anywhere else.

## Part 1. Set up (once)

1. **Check the Mac.** Run `uname -m` (must say `arm64`, Apple silicon) and `sw_vers -productVersion` (must be 15.1 or
   newer). Edits for Mac needs both. If either fails, stop and tell them plainly: this skill cannot run on this Mac.
2. **Edits installed and signed in?** Check `/Applications/Edits.app` exists. If not: they download it free from
   https://edits.instagram.com/desktop , drag it into Applications, open it and sign in with Instagram. Wait for them.
3. **Automation Server.** First run `claude mcp list`. If `metapilot-native` is already there and connected, skip to
   step 5. Otherwise ask them to open Edits, go to Settings, then Automation, turn on "Enable Automation Server",
   and press the copy button for **Claude Code**. They paste that command into a **new terminal tab** (in cmux or
   Terminal; not into this chat: it holds their private access token) and press Enter. It adds a server called
   `metapilot-native` to Claude Code. Edits must stay open whenever they use this skill.
4. **Restart Claude Code.** Tools load when a session starts, so a session that was open before step 3 cannot see Edits.
   Tell them to type `/exit`, run `claude` again, and say: "continue Part 1 of the editing-reels-for-retention skill".
   In the new session, confirm `metapilot-native` is connected (`claude mcp list`, or they type `/mcp`), then call
   `appState`. It must answer `home`, `editor` or `loading`.
5. **Prove the editing tools load.** Call `newProject`, title it "Claude connection test (safe to delete)", and check
   that editor tools such as `splitAt`, `transcribeClip` and `generateCaptions` are now listed. If they are not, ask the
   person to leave that project open in Edits and start one more new Claude Code session, then check again (the tool list
   follows Edits' screen). Then `closeProject`. Tell them they can delete the test project in Edits.
6. **Node.** The mind reader's two small scripts need Node.js 18 or newer: `node --version`. If it is missing, explain
   what it is and offer the installer from https://nodejs.org (or `brew install node` if they use Homebrew). Ask first.
7. **Make the data folder** `~/EditsRetention/` with `past/` (one folder per past video), `new/` (videos to edit) and
   `profile/`. Copy [mind-reader-method.md](./mind-reader-method.md)'s "What to collect" list into
   `~/EditsRetention/README.txt` so they can see what goes where.
8. Tell them, in two sentences each, how to use Part 2 and Part 3.

Test: `appState` answers, the editor tools appeared in the test project, `node --version` prints 18 or newer, and
`~/EditsRetention/` exists with its three folders.

## Part 2. Learn my viewers (the mind reader)

Follow [mind-reader-method.md](./mind-reader-method.md) step by step. There are two ways in. Use the first whenever it
is set up.

**Route A, exact data (recommended): the Instagram retention teardown.** The person's own `/retention-teardown-instagram`
(from github.com/christianjohnston-ai/claude-cockpit, set up with its SETUP.md) already holds, for every Reel it tracks,
the per-second curve, the views, the words with times and the video file, in
`~/.claude/skills/instagram-retention-teardown/accounts/<alias>/library/<shortcode>/`. If that folder does not exist yet,
offer to set the teardown up first (its README's one-line install, Instagram steps only, inside cmux), or use Route B.
1. **Refresh.** Ask the person to run `/retention-teardown-instagram` once in a cmux Claude Code session (it pulls new
   numbers, up to 20 Reels a run), or use what is there if it ran in the last day.
2. **Convert.** For each Reel in the library posted at least 48 hours before its newest capture: the posted time is
   `created_ts` in the Reel's `meta.json` (a Unix timestamp; if missing, `media_created_at` in the capture, which can be
   null), the capture time is `captured_at` (a date string). Read the newest line of `captures.jsonl` that has
   `retention.points` and write `~/EditsRetention/past/<shortcode>/curve.json` (`"platform": "instagram"`,
   `"source": "teardown"`, `views` from the same line, `points` as they are), and copy `words.json` beside it. Skip a
   Reel with no curve, no posted time or no `words.json` (its video download failed), and say how many you skipped and
   why.

**Route B, screenshots (no cmux, or TikTok without the teardown).**
1. **Collect.** For each past video (aim for 10 or more, each posted at least 48 hours ago), the person makes a folder in
   `~/EditsRetention/past/` with: the video file they posted, a screenshot of its retention graph, and a screenshot showing
   its views (about a minute per video). Instagram graphs come from the Reel's insights in the Instagram app, which needs
   a Professional (Creator or Business) account. TikTok graphs come from the video's analytics.
2. **Read each curve** into `curve.json` (the method file says exactly how to read a graph from a screenshot, and how
   rough that is).
3. **Get the words** with times: import the video into a scratch Edits project and call `transcribeClip`. Save
   `words.json`. Close the scratch project afterwards.

**Then, either route:**
4. **Split into beats and label them from the words alone, before you look at the curve.** Use the fixed label set in the
   method file. Save `beats.json`.
5. **Build the profile:** `node scripts/viewer_profile.mjs ~/EditsRetention/past` (from this skill's folder). It writes
   `~/EditsRetention/profile/profile.json` and prints a table.
6. **Explain it** in plain words: which kinds of moments make their viewers leave faster than their own average, which
   hold better, and which have too little data to say. Name the number of videos behind each line. If most lines say
   "not enough data", say so honestly and explain what more data would change.

Test: `profile.json` exists, and the person has heard which moments lose their viewers, with the video counts.

## Part 3. Edit a new video

Work in this order. After each step, read the result back from Edits (`clips`, `elements`) and tell the person what
changed.

1. **Import.** Copy the video into `~/Movies/Edits/Imports/` (Edits can only read some folders; this one works). Call
   `newProject`, give it a clear title, set the canvas to 9:16 at 30 fps (`setCanvas`, read its schema), then `addMedia`
   with the copied file's path.
2. **Words.** `transcribeClip` on the main clip. Save `~/EditsRetention/new/<name>/words.json`.
3. **Beats and flags.** Split into beats and label them the same way as Part 2 (words only). Then make every pause over
   0.6 seconds from `findPauses` its own `pause` beat: end the beat before it where the pause starts and start the next
   one where it ends, so no two beats overlap. Save `beats.json`. Run
   `node scripts/flag_cut.mjs ~/EditsRetention/profile/profile.json ~/EditsRetention/new/<name>/beats.json` (pass `none`
   instead of the profile if Part 2 has not been done yet; then only the after-the-payoff and pause checks run, and say
   so). It prints flags with codes (F1, F2...): the seconds, the words, and why. It never flags the hook, the main
   point, the payoff or the call to action; if one of those leaks, it prints a note to tighten it instead.
4. **The person picks.** Show every flag. They answer by code ("cut F1 and F3, keep F2"). No answer means no cut.
5. **Cut.** For each approved flag, from the LAST one to the first (so earlier times don't move): find clean cut points
   with `quietPoint` near each edge, `splitAt` the main clip at both points, select the middle piece with `selectById`
   and `delete` it. Then read `clips` and confirm there is no gap. If there is one, close it before the next cut. Tell
   them the new length. Then run `transcribeClip` on each remaining main-track clip in order (their words come back in
   timeline seconds) and save them together as `words_after.json`: every time from here on (captions, music, beats,
   effects, and Part 4) uses the NEW timeline, not the old one.
6. **Captions.** `generateCaptions`, then list the caption styles and apply the one they pick (`setCaptionStyle`).
   Check a few captions against `words.json` and fix misheard names with `editCaption`.
7. **Music.** Ask the mood. Search with plain mood words (`searchMusic`), play their top picks (`previewMusicTrack`),
   add the chosen one (`addSongToComposition`), set it well under the voice (`setVolume`, start around 0.2 and let them
   judge), fade it out at the end (`setFade`). If they will post the same file on TikTok or YouTube, pick from the
   royalty-free tab and tell them Edits' music is licensed for Instagram's own apps.
8. **Beat timing.** Get the beat times (`beatMarkers` on the music element after `setBeatDensity`; if it has none,
   `audioOnsets` gives the tempo). Then: slip the music (`slipClip`) so a strong beat lands on the payoff or the first
   cut, read `beatMarkers` AGAIN (slipping moves every beat), and place every sound effect and text pop-in on the nearest
   beat. Do not move cuts inside speech to chase a beat:
   the words come first.
9. **Sound effects.** Few and purposeful: a soft whoosh on a cut, a pop when a key word or number appears. Search with a
   word (`searchSoundEffects`, never an empty query), `seek` to the moment, `addSoundEffect`, then set its volume under
   the voice. Read `elements` to confirm each landed where you meant.
10. **Export.** `setExportSettings` with the full row (see edits-tools.md: 1080x1920, HD, nineBySixteen, high, 30 fps,
    SDR), `export` to `~/Movies/Edits/Exports/<name>.mp4`, and poll `exportStatus` until it says complete. Give them the
    file's path and open its folder (`open -R <path>`). Edits exports HEVC only.
11. **Record** in `~/EditsRetention/new/<name>/cuts.json`: each cut (code, old seconds, words), and each flag the person
    kept, with its time moved onto the new timeline (its old start minus the length of every cut before it). Part 4
    needs those new times.

Before you call it done: the export exists, its length matches the project, captions match the words, and the voice is
clearly louder than the music. Tell them where the file is and what you did, in five lines or fewer. Never post it.

## Part 4. Grade the flags (48 hours after posting)

Get the new video's curve: with the teardown, ask the person to run `/retention-teardown-instagram <alias> <reel link>`
48 hours after posting (the link adds the new Reel to what it tracks) and read it as in Part 2, Route A; without it, read
their retention screenshot as in Route B. Add the video to `past/` (with its beats from `words_after.json`), and check
each flag they kept, at its NEW time from `cuts.json`: did that stretch lose viewers faster than their average? Tell them what
the mind reader got right and wrong, then rebuild the profile with `viewer_profile.mjs`. This is how the profile earns
trust or loses it.

## Where everything lives

| What | Where |
|---|---|
| This skill | the folder this file is in (`scripts/` beside it) |
| The person's data | `~/EditsRetention/` (`past/`, `new/`, `profile/`) |
| The teardown's Instagram data (Route A) | `~/.claude/skills/instagram-retention-teardown/accounts/<alias>/library/` (read only: never change or delete it) |
| Videos Edits can import | `~/Movies/Edits/Imports/` |
| Exports | `~/Movies/Edits/Exports/` |
| The Edits connection | Claude Code's own config (added by Edits' copy-button command) |

## Additional resources

- [Edits tools: which tool for which job, with the arguments that bit us](./edits-tools.md)
- [The mind reader: what to collect, how to read a graph, labels, the math](./mind-reader-method.md)
- [Gotchas and footguns](./gotchas.md)
