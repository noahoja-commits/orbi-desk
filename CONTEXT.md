# Context

ORBI is Noah's voice companion for the suite. One gateway, several bodies. The iPhone app is a native shell over the gateway's page. The laptop page is another. The GTK overlay (`orbi_overlay`) is a third face for that same page. **ORBI desk is a different kind of body:** a thin client that owns the microphone, the wake word, the voice in the room, and the laptop's hands, and speaks the device protocol to the gateway.

The GitHub About line on `noahoja-commits/orbi` is one sentence: "ORBI — voice-companion gateway for the suite; the render is the face." The gateway README's section **What ORBI does without a model** (2026-10-13) is the command list. This file is the rest of the picture, written for the desk.

## Brain and body

| | Gateway | Desk |
|---|---|---|
| Machine | The Dell, always on | The Framework 13, the laptop on the desk |
| Code | `~/orbi` on the Dell, repo `noahoja-commits/orbi` | `desk/` in that same repo, checkout on the Framework |
| Process | `orbi.service`, loopback only | user unit `orbi-desk.service` |
| Job | Understand the words, call the tools, decide what to say | Hear, speak, show the face, do the thing on this computer |
| What crosses the wire | Text frames on one WebSocket | The same frames. No audio. No screenshots. No window titles |

The desk keeps an SSH forward to the gateway and connects to `ws://127.0.0.1:18125/device`. The public site in front of the gateway is for browsers. A body is not a browser.

An Android phone later is the same protocol with a different `hello` and different hands. The protocol doc in the gateway repo is `docs/DEVICE-PROTOCOL.md`.

## What ORBI does before a model is asked

Every turn is read by a rule table first (`orbi/intents.py`). A plain command runs immediately through the same tool registry the model uses. Only what the rules do not recognise goes to a model.

| Say | What happens |
|---|---|
| "Set a timer for 10 minutes" · "Set a pasta timer" · "How long is left" · "Cancel the pasta timer" | Timers. One with no length asks "For how long?" |
| "Play Khalid" · "Pause" · "Skip" · "Louder" · "Volume 40" · "What's playing" | The player for that surface. On the desk, these speakers, not the Dell's player, unless he says "on the laptop" |
| "What's on my calendar today" · "And tomorrow?" · "What's next" · "Do I work on Friday" | ALMANAC, read |
| "Add dentist to my calendar tomorrow at 3pm" | Read back as a question. A plain yes adds it |
| "Add milk and eggs to the shopping list" · "We're out of coffee" · "What's on my shopping list" | UPKEEP |
| "Add call the bank to my to-do list" · "What's my one thing" | CHROME. Adds ask first |
| "Note to self…" · "Capture a thought" | STASH, category `orbi` |
| "What's the weather" · "Will it rain tomorrow" | Open-Meteo |
| "What time is it" · "How many days until Halloween" · "15 percent of 80" · "How many cups in a gallon" | Clock and arithmetic |
| "Is everything running" · "How much memory is free" | The machine the gateway is on |
| "Undo that" · "Say that again" · "More" · "Stop" · "What can I say" | The conversation itself |

Answers are one short sentence. A list is cut to three, and "more" reads the rest. A follow-up within about a minute keeps its subject ("and tomorrow?", "make it fifteen", "and eggs"). Anything that changed something can be taken back with "undo" for two minutes. Spoken text is rewritten for the ear ("3:05 PM" is read "3 oh 5 PM"). The thread shows it as written.

`GET /api/help` is that table as data. `GET /api/diagnostics` shows which path answered the last twenty turns and how each provider is doing, with no word of what was said.

When no model can answer, ORBI says so once and names what still works: timers, music, calendar, lists, weather, quick sums.

## What else the brain can do

These are real, past the 2026-10-13 table. They matter on the desk because the bubble and the thread have to show them.

- **Money** (EUROBANK). Reads are free: bills, spending, subscriptions. Anything that writes asks first, alone, and the yes in the next turn is the only yes that counts.
- **Find.** "Find that note about …" searches the suite's own apps and answers with a few hits. No model.
- **Quiz.** "Quiz me" runs SCRY's study cards by voice. While a quiz is running, the next utterance is the answer.
- **Briefing and nudges.** "Brief me", "how was my week with money", "quiet today", "nudges off". The gateway also speaks first: a timer, a bill, an agent that finished. Quiet hours 22:00–08:00, one nudge per 30 minutes, six a day. They arrive on the desk as a push while he is here.
- **Coding desk.** "What's Kimi doing", "have Kimi fix the login bug in SCRY", "tell Kimi to add tests", "stop Kimi", "show me what Kimi changed". Starting, typing into, and ending a session ask first. The diff is a card, not a dump in the sentence.
- **Phone.** The same words on the phone surface queue actions for the iPhone app (its player, its timers). The desk does not become the phone.

A model turn is shown only the tools of the apps that sentence is about. Small talk can stay on a small local model with no tools. Writes still ask. The model is never allowed to approve itself.

## What the desk adds

Four powers, all on this laptop.

1. **Run the computer.** Open an app, focus it, list what's open, media, volume, play from the library through the tunnel onto these speakers, lock the screen. Close an app, run a script, and type text ask first. Scripts run only if they are named in the desk's config. The gateway never sends a shell command.
2. **See the screen.** "What's on my screen", "read this error", "look at Firefox". One capture, into RAM, shown to a local vision model, file deleted, answer spoken here. A named app is found by class and app id. The window title is screen text and stays on the machine.
3. **Be present.** The ball and the bubble. Pushes and timers show and, when the mic is open and he is not already in a sentence, are said aloud.
4. **Code beside you.** The coding tools above, plus the diff card on the bubble and in the thread.

Socket commands the desk window and the CLI already use: listen, stop, approve, decline, open, focus, volume, media, lock, look, copy, cancel-timer, pet, say, speak, show, hide, status.

## One turn

1. He says "ORBI", or presses Super+Space. Super+Space skips the wake word. Saying ORBI while it is talking stops the voice and drops the turn.
2. Audio never left the machine. The wake-word model saw short frames in memory. A short ring kept the word after "ORBI" from being clipped.
3. He talks. Silence ends the sentence. Whisper runs here. The gateway's noise rules run here too, so a clip that came out as "Thank you." is dropped and nothing is sent.
4. The words go up as an `utterance`. The ball is listening, then thinking.
5. The gateway answers with `reply.start`, then one `reply.delta` per sentence, then `reply.end`. Each sentence is spoken as it arrives. The bubble shows his line and ORBI's line.
6. A tool that belongs on the desk comes down as a `tool_call`. The desk does it and sends `tool_result`. Open, focus, and close walk the ball to that window. A question (`confirm`) brings him forward and puts YES / NO on the bubble. "Yes", "no", and the buttons are the same path. A token that is not the one waiting is ignored.
7. "Never mind" and the bubble's dismiss drop the turn, decline a question still waiting, and send no new utterance. "Stop" stops the voice.
8. The bubble tucks away after a quiet moment. The ball goes back to where he was put.

The thread of the conversation is kept in memory on the desk (a short deque) and on the 0600 control socket. It is not written to the journal. A restart forgets it on purpose. His words are not a log file.

## Protocol, short

Text frames are JSON with a `type`. Binary audio exists only for a body that asked the gateway to speak, or a body with no ears. This desk has ears and a local voice, so it does not use those binary frames.

Body to gateway: `hello`, `utterance`, `stop`, `cancel_timer`, `approve` / `decline`, `tool_result`, `presence`, `listening`, `caps`, `ping`.

Gateway to body: `welcome`, `reply.start` / `reply.delta` / `reply.end`, `state`, `tool`, `confirm`, `tool_call`, `push`, `card`, `error`, `mood`, `pong`.

`hello.caps` lists the desktop ops this body can do. The gateway does not call an op the body did not offer. Risky ops (`close_app`, `run_script`, `type_text`) arrive with `confirmed: true` or the body refuses them again.

Auth is a per-device token in a 0600 file on the laptop. The gateway stores only the hash. Pairing is `orbi-desk pair` over SSH. Revoking is `python -m orbi.devices remove desk` on the Dell. Nothing in this repo is a token, and nothing here should become one.

## Privacy

- Before the wake word or the hotkey, audio goes nowhere. Nothing is written to disk.
- After it, only the transcribed words leave.
- The screen never leaves. Any vision URL that is not the local model is refused in code.
- Window titles never leave. "What windows are open" sends app names.
- Mute closes the microphone process. It does not keep recording and ignore the samples. Locking the screen closes it the same way. Unlocking restores the mute he had before the lock. A mute he sets while the screen is locked stays off.
- Copy stays on this machine.
- The trace on the socket can hold the live exchange for the window. The journal gets phase, timings, and errors. It does not get his sentences.

## Voice

The desk speaks with ElevenLabs, voice **Roger** (laid-back, casual, resonant), model `eleven_flash_v2_5`, 24 kHz PCM into the existing player. The key is ORBI's own key, in `~/.config/orbi-desk/elevenlabs.key`, mode 0600. It is a different key from any other app on the machine. It is not in config and it is not in git.

If ElevenLabs fails, the desk falls back to local Kokoro (`bm_george`), then Piper. The voice is not changed unless he names another one.

The gateway's own voice is still Piper. That is the phone and the page. The desk's voice is the one in the room.

## Ears

Wake word: `desk/wakeword/orbi.onnx`, threshold 0.73, trained on synthetic voices. It does not yet know Noah's voice, and it does not yet hold him out of music or ignore other people. That training is a session with him at the mic. Until then Super+Space is the sure way in. Do not start a recording run unless he asks.

The Framework's usable microphone is the internal stereo pair, Mic2. Mic1 emits a constant full-scale sample and is disabled in WirePlumber on this machine. The desk config pins Mic2. `orbi-desk status` shows `mic`, `mic_level`, and `mic_dead_s` (seconds of a flat signal).

## Where the code lives

On the Framework the checkout is `~/orbi/orbi-src`, branch `desk`. Launcher `~/.local/bin/orbi-desk`. Config `~/.config/orbi-desk/config.toml`. Look `~/.config/orbi-desk/look.json` (color, hat, wander, size). `wander` means follow the work. It does not mean roam.

| Module | Job |
|---|---|
| `app.py` | Joins it. The turn, the thread, the socket |
| `listen.py` | Wake, end of sentence, local Whisper, hearing rules |
| `wake.py` | The wake-word model |
| `tts.py` | ElevenLabs, then Kokoro, then Piper. Barge-in |
| `audio.py` | Mic and player. Mute closes the mic process |
| `hud.py` | The ball and the bubble. GTK3, layer-shell, cairo |
| `petlook.py` | Colors, hats, eye poses, where a task puts him |
| `appwindow.py` | The desk window |
| `desktop.py` | KWin or Hyprland, media, volume, launch, typing |
| `screen.py` | Local look. Titles stay here |
| `link.py` | The tunnel and the WebSocket |
| `hotkey.py` | Super+Space on KDE |
| `config.py` | `config.toml` |
| `__main__.py` | `orbi-desk` CLI |

Tests: `desk/test_desk.py` and `desk/test_petlook.py` from the checkout root. The gateway side of the protocol is `test_devices.py`.

`orbi-desk-app.service` is the window. It is disabled and does not start at login. Opening the desktop icon starts it. A second launch raises the one already open.

## Still open

- The wake word has not been trained on his voice.
- Typing needs a kernel with `uinput` and a login whose session is in group `input`, then `ydotool`. Do not claim typing works until that session exists.
- The big window is a working board. [DESKTOP.md](DESKTOP.md) is what it should grow into. The ball is the face. The window is not.
- Screen look needs the local vision model on 127.0.0.1:8085. If it is down, the desk says so and does not send the question anywhere else.
