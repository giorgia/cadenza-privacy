---
name: cadenza-routine
description: Turn a workout post (TikTok, Instagram, YouTube, a caption, screenshots) into a Cadenza routine and reply with an https link that opens the Cadenza iOS app with the routine filled in. Use when a user pastes a "Turn this workout post into a routine for the Cadenza app" prompt, or asks to send a workout to Cadenza.
---

# Cadenza routine hand-off

Cadenza is an iOS workout app. It can't read TikTok/Instagram posts from a
link, so the user asked you to read the post and hand the routine back as a
link. Tapping your link opens Cadenza on its Review screen, where the user
checks the routine and saves it. Nothing is saved without them.

Format version: 1. This page:
`https://giorgia.github.io/cadenza-privacy/agent/cadenza-routine.md`

## How to deliver the routine (read this first)

Always deliver the routine as this **https** link:

```
https://giorgia.github.io/cadenza-privacy/routine.html?data=<URL-encoded routine JSON>
```

- **Never send a bare `cadenza://` link.** Chat apps only open http(s)
  links: a `cadenza://` link in a message, a Markdown link, an HTML button or
  a card is not tappable, or the tap is silently dropped. The https page
  above shows the routine's title and forwards to Cadenza itself.
- Below the link, always include the **same JSON, unencoded, in a code
  block**. If the link can't open (no Cadenza installed, a blocked redirect),
  the user copies the JSON and pastes it into Cadenza's Import screen, which
  reads it the same way.
- Then add a short plain list of what you extracted and where each part came
  from (caption, video text, screenshots), so the user can check it at a
  glance.

## Opening Cadenza

How the link opens the app, so you can set it up and explain it:

1. The user taps the https link in your reply. It opens in their browser
   (or your app's in-app browser).
2. The page shows the routine's title and hands off to Cadenza. iOS may ask
   "Open in “Cadenza”?"; the user taps **Open**.
3. Cadenza opens on its Review screen with the routine filled in. The user
   checks it and taps Save. Nothing is saved before that.

What to do in your interface:

- **Plain chat reply:** put the https link on its own line, as a Markdown
  link if supported: `[Open in Cadenza](https://giorgia.github.io/cadenza-privacy/routine.html?data=...)`.
- **Buttons, cards, widgets or embedded HTML:** use the same https URL as
  the `href`/action. Never `cadenza://`: these views drop custom-scheme taps
  silently, and some reject non-https URLs outright.
- **You can open URLs yourself** (a browser or computer-use tool on the
  user's iPhone): open the https link, not `cadenza://`. On any other device
  (your own sandbox, a desktop browser) don't try to open it; Cadenza is an
  iPhone app, so just give the user the link.
- **Tell the user what to expect** in one short line, e.g. "Tap the link,
  then tap Open if your phone asks."

If it doesn't open, tell the user:

- If the page stays open, tap its **Open in Cadenza** button (some in-app
  browsers block the automatic hand-off).
- If the page says the routine can't be read, the link got cut off: send it
  again, ideally shorter (leave out `video_url` first).
- If nothing happens on the button either, Cadenza probably isn't installed
  (it's on the App Store), or the browser blocks app links: copy the JSON
  from your code block, open Cadenza, go to **Import**, paste it and tap
  **Extract**.

## Steps

1. **Get the workout yourself.** The user should have to do almost nothing.
   Work down this list and stop as soon as you have the complete workout:
   1. **Fetch the post page** and read the caption, description, on-page
      text and pinned comment. Many captions list the full routine.
   2. **Download the video** if you have a tool for it (yt-dlp, a code
      sandbox, a video-capable fetch) and read its on-screen text, and its
      audio transcript if you can. Then add its direct file link as
      `video_url` (see "Sending the video"), so Cadenza can show move
      previews.
   3. **If you can't download it, use your browser tool.** Open the post
      (also try the embed view, e.g. `https://www.instagram.com/reel/<id>/embed/`
      or `https://www.tiktok.com/embed/v2/<id>`, which often load without a
      login). Play the video and pause or scrub through it, taking a
      screenshot each time the on-screen text changes (usually once per
      exercise). Step through every slide of a carousel. Then read the
      screenshots together with the caption.
   4. **Only as a last resort, ask the user once** for the caption text or
      screenshots, and say exactly what you couldn't get ("the reel needs a
      login, so I couldn't see the exercises").

   Combine everything: the caption often has the rounds and rest, while the
   video text has the exercises and times. Don't guess from the URL, the
   thumbnail or the creator's name.
2. **Extract only what the source states.** Every exercise, in order, with
   its reps or its time as given.
   - **Never invent sets, reps, durations or rest values.** If the post
     doesn't state a number, leave that field out. Don't fill in "typical"
     values (3 sets, 10 reps, 30 s rest) and don't average or round ranges up.
     Cadenza flags missing values for the user to fill in; a made-up number
     looks real and won't be checked.
   - Never invent exercises either. If a move is unreadable, leave it out and
     say so in your summary.
3. **Build the JSON** (format below). Keep exercise names as the post writes
   them, minus emoji, counts and times ("Squats 40s" → name `"Squats"`,
   `duration_seconds: 40`).
4. **Encode it with code, not by hand**, and build the link:
   - JavaScript: `"https://giorgia.github.io/cadenza-privacy/routine.html?data=" + encodeURIComponent(JSON.stringify(routine))`
   - Python: `"https://giorgia.github.io/cadenza-privacy/routine.html?data=" + urllib.parse.quote(json.dumps(routine, separators=(",", ":")), safe="")`

   Base64url-encoded JSON is also accepted as the `data` value. Keep the
   JSON under 32 KB.
5. **Reply** as described in "How to deliver the routine": the https link
   (as a Markdown link if your interface supports it, e.g.
   `[Open in Cadenza](https://giorgia.github.io/cadenza-privacy/routine.html?data=...)`),
   then the JSON code block, then the short summary.

## Sending the video

When you could get the video, add `video_url`: a direct **https** link to
the video file itself (an `.mp4`/`.mov` response, not the post page).
Cadenza downloads it when the user opens your link, keeps a copy with the
routine and cuts a short preview clip for each move.

- Get it with `yt-dlp -g "<post url>"` (pick the single combined
  video+audio format, e.g. `-f "b[ext=mp4]/b"`), or from the `<video>`
  element's `src` in your browser tool if it's an https file URL (not
  `blob:`).
- It must download on the user's phone **without cookies, a login or special
  headers**. Don't send `sandbox:`/`file:` paths or links to your own
  workspace; the phone can't reach them.
- These CDN links usually expire within hours. Tell the user to open the
  link soon.
- Max 300 MB. If you can't produce a reachable URL, leave `video_url` out:
  the routine still opens, just without previews.

## JSON format

```json
{
  "title": "Leg Day Burner",
  "sets": 3,
  "rest_between_exercises_seconds": 20,
  "work_seconds_per_exercise": 40,
  "source_url": "https://www.instagram.com/reel/ABC123/",
  "creator": "@coachname",
  "video_url": "https://scontent.cdninstagram.com/v/t50.2886-16/123_n.mp4?...",
  "exercises": [
    { "name": "Jumping Jacks", "duration_seconds": 60, "section": "warmup" },
    { "name": "Squats", "reps": 15, "section": "main" },
    { "name": "Reverse Lunges", "reps": 10, "side": "per_side", "section": "main" },
    { "name": "Plank", "duration_seconds": 45, "rest_seconds": 30, "section": "main" },
    { "name": "Hamstring Stretch", "duration_seconds": 30, "side": "per_side", "section": "cooldown" }
  ]
}
```

| Field | Required | Meaning |
| --- | --- | --- |
| `title` | yes | Short routine name. Use the post's, or describe it ("Full Body HIIT"). |
| `exercises` | yes | 1 to 40 moves, in order. |
| `exercises[].name` | yes | The move's name, max 80 characters. |
| `exercises[].reps` | if stated | Rep count (1 to 1000). |
| `exercises[].duration_seconds` | if stated | Time for the move (1 to 3600). Use this, not reps, for timed moves. |
| `exercises[].rest_seconds` | if stated | Rest stated after this specific move (0 to 3600). |
| `exercises[].side` | no | `"per_side"` when done on each side (lunges, side planks). |
| `exercises[].section` | no | `"warmup"`, `"main"` (default) or `"cooldown"`. |
| `sets` | if stated | Rounds of the main section ("3 rounds", "x3"), 1 to 99. For a range ("2-3 sets") use the lower number. |
| `rest_between_exercises_seconds` | if stated | One rest stated for the whole routine ("rest 15s between moves"). |
| `work_seconds_per_exercise` | if stated | One work interval stated for every move ("40s on / 20s off" → 40, with rest 20). |
| `source_url` | no | The post link (http/https). |
| `creator` | no | The creator's handle, for credit. |
| `video_url` | no | Direct https link to the video file, reachable without login. Enables move previews. |
| `language` | no | Two-letter language code of the post (default `"en"`). |

Rules of thumb:

- A move with neither reps nor time is fine: leave both out.
- Don't add rests as exercises. Use `rest_seconds` or
  `rest_between_exercises_seconds`.
- "Repeat the circuit 3 times" is `sets: 3`. Don't list the moves three
  times.
- Warm-up and cool-down moves are not repeated by `sets`.
- Numbers are plain integers, never strings with units.
- Values outside the ranges above are dropped by Cadenza, so the user sees
  them as missing.

## Example reply

> Here's the routine from the post: 2 moves, 3 rounds, 20s rest between moves.
>
> [Open in Cadenza](https://giorgia.github.io/cadenza-privacy/routine.html?data=%7B%22title%22%3A%22Leg%20Day%22%2C%22sets%22%3A3%2C%22rest_between_exercises_seconds%22%3A20%2C%22exercises%22%3A%5B%7B%22name%22%3A%22Squats%22%2C%22reps%22%3A15%7D%2C%7B%22name%22%3A%22Plank%22%2C%22duration_seconds%22%3A45%7D%5D%7D)
>
> If the link doesn't open Cadenza, copy this into Cadenza's Import screen:
>
> ```json
> {"title":"Leg Day","sets":3,"rest_between_exercises_seconds":20,"exercises":[{"name":"Squats","reps":15},{"name":"Plank","duration_seconds":45}]}
> ```
>
> From the caption: 3 rounds, 20s rest. From the video text: Squats ×15, Plank 45s.
