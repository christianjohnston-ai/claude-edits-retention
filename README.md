# Claude edits your Reels in Instagram's Edits app, and cuts where your viewers get bored

**Install it by pasting this into Claude Code:**

```
Install the editing-reels-for-retention skill from https://github.com/christianjohnston-ai/claude-edits-retention : put it in ~/.claude/skills/editing-reels-for-retention (replace an older copy if there is one). Then read its SKILL.md and do Part 1 with me, one step at a time.
```

Instagram's free desktop editor, Edits, now lets an AI agent work inside it. This skill teaches Claude Code how: import
your video, add captions, music and sound effects, line moments up with the beat of the music, and export a 9:16 file
ready to post.

And one part works like a mind reader. Before you post, it looks at your own past retention from Instagram or TikTok,
learns which kinds of moments make YOUR viewers leave (a slow set-up? step-by-step parts? anything after the answer?),
finds those moments in your new video, and offers to cut them. You approve every cut.

The mind reader's best data comes from the **Instagram retention teardown** (`/retention-teardown-instagram`), a free
companion that pulls each of your Reels' per-second retention curve straight from Instagram, lines it up with your
words, and builds its own page of where your viewers leave, lessons and hook rewrites. The full setup below installs
both. Without the teardown, the mind reader can learn from screenshots of your retention graphs instead.

## What you need

- A Mac with Apple silicon (M1 or later) and macOS 15.1 or newer. Edits for Mac needs both.
- **Edits for Mac**, free: https://edits.instagram.com/desktop , signed in with your Instagram account.
- **Claude Code**, which comes with a paid Claude plan (Pro or higher): https://code.claude.com/docs/en/setup
- **cmux**, the Mac terminal with a built-in browser: https://cmux.com . The teardown runs inside it, so run Claude Code
  in cmux for everything.
- An Instagram **Professional** (Creator or Business) account: Instagram shows Reel insights only to those.
- Node.js 18+ for the mind reader's scripts, and for the teardown: Homebrew, ffmpeg, Python 3.10+ and about 1.6 GB of
  disk for its transcription model. Claude checks each one and installs what is missing, asking first.
- Edits open whenever you edit.

## Set it up

**Part A, the editing skill (about 15 minutes if you already have Claude Code):**
1. Install Edits for Mac and sign in. Install cmux and open it.
2. In Edits: **Settings**, then **Automation**, turn on **Enable Automation Server**, and press the copy button for
   **Claude Code**. Open a new terminal tab in cmux, paste that command there and press Enter. (It holds a private access
   token: never paste it into Claude's chat.)
3. Now start `claude` in cmux, copy the install prompt above and paste it. It walks you through the rest, including a
   restart of Claude Code so it can see Edits, and a quick test project to prove the editing tools load.

**Part B, the Instagram retention teardown (longer: it installs a transcription model, then pulls your Reels one by
one, about 75 seconds each, up to 20 per run).** In Claude Code inside cmux, paste:

```
Clone https://github.com/christianjohnston-ai/claude-cockpit into ~/claude-cockpit (git pull if it is already there), then read ~/claude-cockpit/retention-teardown/SETUP.md and follow it one step at a time for Instagram only (skip its TikTok steps 11 and 12), stopping at every step marked YOU DO THIS until I say done.
```

It stops at the steps only you can do: your Mac password for Homebrew, signing in to Instagram inside cmux's browser,
and sending the links of the Reels to track. Then run `/retention-teardown-instagram` for your first teardown.

## Use it

Say it the way you'd say it to an editor:
- "Learn my viewers." Claude reads your teardown data (or your retention screenshots) and tells you which moments lose
  your audience, and how sure it is (it says "not enough data yet" when that's the truth).
- "Edit this video in Edits: [path to the file]." It flags the boring stretches with codes (F1, F2...). You say
  which to cut. Then captions, music, sound effects and beat timing, and an export to your Movies folder.
- "Grade my last Reel." Two days after you post, run `/retention-teardown-instagram` with the new Reel's link (or give
  it a retention screenshot), and it checks its own flags.
- `/retention-teardown-instagram` on its own, any time: the teardown page, where every Reel loses viewers, with lessons
  and hook rewrites built from your own lines.

## How the mind reader decides

It compares each moment with your own usual pace: how fast your viewers usually leave at that point in a video,
averaged over your other videos. It labels every sentence of your past videos from the words alone (hook, set-up,
steps, payoff and so on) before looking at the graph, then counts who actually left against who usually would. A kind of
moment only counts as "your viewers leave here" when it shows up in 5 or more videos and so many of them agree that
chance alone would rarely do it. It never predicts a percentage, and it never offers to cut your hook, your main point,
your payoff or your call to action. The details are in [mind-reader-method.md](./mind-reader-method.md).

## Good to know

- It never posts. You export the file and post it yourself.
- It never touches your existing Edits projects. Every edit happens in a new project.
- Your videos, screenshots and retention data stay in `~/EditsRetention` on your Mac. Claude reads them in your chat to
  do the work, like any file you show it, and never uploads them anywhere else.
- What has run end to end on a real Mac (Edits 456, November 18, 2026): the mind reader learned from a creator's own
  teardown data, flagged a slow set-up in a new Reel, and Claude cut it in Edits, added captions, a royalty-free song
  with a downbeat placed on the cut, and two sound effects, then exported a 1080x1920, 30 fps file with every word
  outside the approved cut intact. That run corrected more than a dozen steps of this skill (see gotchas.md). Text overlays,
  transitions and video effects have not been run yet, so Claude checks each result as it goes.
- What to expect from the mind reader at first: on the author's own account, 28 Reels gave no verdict yet. A kind of
  moment needs nearly all of your videos to agree before it gets one, so the table can read "not enough data yet" for a
  long time. Until it fills in, the flags come from the general checks: the main point landing later than your usual,
  anything after the payoff, and long pauses.
- Edits' music library is Meta's. If you'll post the same file on TikTok or YouTube, pick from the royalty-free tab and
  check the terms yourself.
- Edits updates often. The skill reads Edits' own list of tools every session instead of trusting a fixed list.
- Small accounts get rough answers: with a few hundred views per video, most patterns read "not enough data yet".

## Files

| File | What it is |
|---|---|
| [SKILL.md](./SKILL.md) | What Claude follows: setup, learning your viewers, editing a video, grading the flags |
| [edits-tools.md](./edits-tools.md) | Which Edits tool does which job, with the argument names that matter |
| [mind-reader-method.md](./mind-reader-method.md) | What to collect, how a graph is read, the labels, the math |
| [gotchas.md](./gotchas.md) | Things that went wrong in real runs, and the fix for each |
| [scripts/viewer_profile.mjs](./scripts/viewer_profile.mjs) | Builds your viewer profile from past videos |
| [scripts/flag_cut.mjs](./scripts/flag_cut.mjs) | Flags the stretches in a new video, with codes |

Made by Christian Johnston (@christianjohnston.ai)

MIT License
