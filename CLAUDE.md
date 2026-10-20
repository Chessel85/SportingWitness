# Sporting Witness player

A simple web page for working through the BBC World Service *Sporting Witness* podcast back catalogue, oldest first, while running on a treadmill. The original brief is in `requirements.txt`.

- Live site: https://chessel85.github.io/SportingWitness/ (GitHub Pages, deployed from `main`, root folder)
- Repo: https://github.com/Chessel85/SportingWitness (public)
- Feed: https://podcasts.files.bbci.co.uk/p00j2pn5.rss (about 685 episodes, 30 Jul 2011 to now; **2013 is missing**; the feed allows cross-origin fetches)

## The user

The user is blind and uses **VoiceOver on an iPhone 13 mini** (iOS 27) in Safari. Build VoiceOver-first:
- Real `<button>`s with spoken text labels, never bare symbols. Use headings to structure the page.
- One polite live region (`#status`) for announcements. Describe results in words, not visuals.
- Getting a new URL onto the phone is tedious, so **link every new page from the main page**.
- After each change, push, wait for the Pages deploy to finish, then tell the user exactly what to test.

## VoiceOver silence bug: what we learned (October 2026)

When a new audio file started playing in Safari, VoiceOver speech went completely silent. Only turning VoiceOver off and on brought it back. This happened with no headphones connected. Findings:

| Tried | Result |
|---|---|
| `<audio controls>` in the page | Silenced VoiceOver |
| `<audio>` element in the page (hidden) | Silenced VoiceOver |
| Media Session API (metadata, action handlers) | Silenced VoiceOver |
| `navigator.audioSession.type` = auto / playback / ambient | No effect |
| Delaying the "Playing" announcement by 3 seconds | No effect |
| 1-second beep file before the episode | No effect |
| Add to Home Screen web app | Worse (VoiceOver would not recover) |
| **Inaudible Web Audio oscillator kept running from the first press, under a detached `new Audio()`** | **Works.** VoiceOver keeps talking for new episodes, Next, pause and resume, and autoplay. |

So the rules for the player are:
- Play through a detached `new Audio()`. Never put an `<audio>` element in the page.
- Don't use the Media Session API, so there are no lock-screen or Magic Tap handlers for now. Possibly retest it later with the keep-alive running.
- Start the keep-alive `AudioContext` (oscillator, gain 0.0001) on the first user press and keep it running (`startKeepAlive()` in `index.html`).

`test/index.html` is the diagnostic page (tests 1 to 9) that isolated this. Keep it, and keep it linked from the main page.

## Current state (full player, `index.html`, October 2026)

- The 0.2 second "Playing <title>" announcement delay is confirmed to keep VoiceOver talking.
- Progress is saved in `localStorage` under `sportingWitnessProgress` as `{ marks: { guid: "played" | "skipped" }, current: guid, position: seconds }`. All access is wrapped in try/catch.
- On load, the page picks the saved episode and position (or the oldest unheard episode) and says "Ready to resume ... Press Play".
- Next episode, and an episode ending, mark it `played` and start the oldest unheard episode (user decision: Next = listened).
- The progress count reads like "142 of 684 heard, 3 not interested". The feed currently has 684 items.
- Now playing shows the title, date, length and the feed's `itunes:subtitle`.
- Up next lists the 10 oldest unheard episodes, each with an `h3` plus Play and "Not interested" buttons. After "Not interested", focus moves to the next item's Play button. An Undo button reverses the last mark.
- Below Up next, a link "Episodes marked not interested (N)" goes to `not-interested/index.html`. That page fetches the feed, lists the `skipped` episodes oldest first (an `h2` plus a "Restore to Up next" button each), and restoring deletes the mark from the same `localStorage` key. The main page re-reads progress on `pageshow` from Safari's back cache.
- Backup: "Save backup file" downloads the progress JSON, and a file input restores it (this replaces the current progress).
- "Test: skip to last 10 seconds" and the log are kept under a Testing heading.

## Next steps

1. Get the user to test the full player on the phone (resume after reload, Not interested, Undo, the Not interested page, backup and restore).
2. Ideas not built yet: skip back and forward buttons, and retesting the Media Session API with the keep-alive running.
3. Optional: find the missing 2013 episodes (possibly in the "Witness History: Archive 2013" feed, `p02p8z14.rss`) and merge them in.
