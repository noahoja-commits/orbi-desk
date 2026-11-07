# Wiring the new UI to the body we have

The new desktop app is the look and the motion. The body already running on the Framework keeps everything else: the mic, the wake word, Whisper, the voice, the gateway, the tools, the yes/no token, and privacy.

When the new UI exists, it gets attached in `desk/orbi_desk/hud.py` (or a module the body imports in that place). `app.py` should not grow a second protocol. It already talks to one object, `self.hud`. The new UI becomes that object.

## What the UI build owns

Drawing and animation only.

- The ball, the eyes, the hats, the colors, the blink, the hop.
- Walking from home to a task and back. The body says where and for how long. The UI plays it.
- The bubble appearing, following the ball, and tucking away.
- The desk window's materials, type, and layout, if that window is part of the same build. The sections in [DESKTOP.md](DESKTOP.md) stay.

No microphone. No wake word. No WebSocket. No tools. No screenshot. No key, no token.

## What the body keeps calling

These are the methods on the current HUD. A new view implements this set. Names can sit on a thin adapter if the new code wants its own internals. `app.py` keeps calling these.

| Call | When |
|---|---|
| `set_state(state)` | `asleep` `idle` `listening` `hearing` `thinking` `working` `looking` `speaking` |
| `set_link(up, why)` | Gateway connected or not. `why` is a short reason, never his words |
| `set_muted(on)` | Mic process is closed |
| `attend(box, where, seconds)` | Walk. `box` is `{x, y, w, h}` or absent. `where` is `work` or `forward`. `seconds` is how long the errand lasts |
| `clear_errand()` | Stop walking and return home |
| `express(mood, seconds)` | `happy` or `angry`, laid over the state |
| `apply_look(color=\|hat=\|wander=\|size=)` | Save and wear it. `wander` means follow the work |
| `new_turn()` | A new thing was said. Clear the live caption |
| `caption(text, final)` | His line, while he talks and when the sentence lands |
| `add_answer(sentence)` | One spoken sentence from ORBI |
| `note(text)` | A short line that is not a full answer ("Didn't catch that.") |
| `card_tool(label, summary, ok)` | A tool finished |
| `card_confirm(token, ask)` | Show the question and YES / NO. Keep the token. Do not print it in a log |
| `settle_confirm()` | The question is over. Hide YES / NO |
| `card_diff(card)` | A diff card from the gateway |
| `card_info(title, text)` | A plain card |
| `card_push(kind, text)` | A timer or a nudge |
| `show()` / `hide()` | Bubble up or tucked. The ball stays |

The body also reads `hud.look` after a pet change. Return the same `{color, hat, wander, size}` dict `apply_look` saves.

Clicks come back the other way, the way they do now:

- Click the ball: mute toggle.
- Drag: that point becomes home, and any errand cancels.
- YES / NO: call the body's yes/no with the token from `card_confirm`.
- Dismiss: the body's "never mind" path.
- Pet controls: `apply_look(...)`.

The desk window, if the new build replaces it, keeps using the socket the body already serves (`orbi-desk` commands: listen, stop, look, open, focus, volume, media, lock, pet, say, copy, cancel-timer, approve, decline). It does not open its own gateway connection.

## How the attach will go

1. The new UI lands as its own view, with no body code copied into it.
2. An adapter implements the table above and draws with the new animation.
3. `Desk` is given that adapter instead of the cairo HUD.
4. The cairo ball stays in the tree until the new one actually shows state, walks to a window, pops the bubble, and takes a yes.
5. Restart is `systemctl --user restart orbi-desk.service`. The window unit stays disabled at login.

Nothing in that swap moves audio, the token, or the screen off the machine.
