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

## Current state (proof of concept, `index.html`)

- Loads the feed, sorts oldest first, and lists the first 5 episodes as "Play <title>, <date>" buttons.
- Play/Pause, Stop and Next episode controls. Autoplay to the next episode works.
- "Test: skip to last 10 seconds" button, used to test autoplay. Remove it in the full build, or keep it if the user wants it.
- Announces "Playing <title>" 3 seconds after playback starts.
- No progress is saved yet.

## Next steps

1. **Cut the post-start announcement delay from 3 seconds to about 0.2 seconds** (the `setTimeout` in the `playing` handler). Then ask the user to confirm VoiceOver still keeps talking.
2. Build the full player:
   - Save progress in `localStorage`, keyed by episode guid: `played` or `skipped` status, plus the playback position within the current episode.
   - **Next episode marks the current episode as listened** (user decision).
   - The "Up next" list shows the next ~10 unheard episodes, each with Play and "Not interested" buttons.
   - Show a progress count, e.g. "142 of 685 heard".
   - Resume the last episode at its saved position.
   - Export and import progress as JSON (a backup, since cleared Safari data would lose it).
   - Wrap all `localStorage` access in try/catch.
3. Optional: find the missing 2013 episodes (possibly in the "Witness History: Archive 2013" feed, `p02p8z14.rss`) and merge them in.
