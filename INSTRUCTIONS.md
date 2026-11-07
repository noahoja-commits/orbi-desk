# Instructions

This is the full instruction set for the desk ORBI. The other files are the detail it points at. The running code is not in this repo. It is `desk/` in [`noahoja-commits/orbi`](https://github.com/noahoja-commits/orbi). On the Framework the checkout is `~/orbi/orbi-src`, branch `desk`.

Two jobs, in order:

1. **Build the desktop UI.** Animations and looks only. No ears, no voice, no gateway, no tools.
2. **Wire that UI to the body that is already running.** That happens after the UI exists. The body keeps the mic, the wake word, Whisper, the voice, the gateway, the tools, and the yes/no token.

Read [DESKTOP.md](DESKTOP.md) for what it should look like, [WIRING.md](WIRING.md) for the method list the body already calls, and [CONTEXT.md](CONTEXT.md) for what ORBI does.

## 1. Build the UI

Own the drawing and the motion.

- The ball: glossy blue sphere, two white capsule eyes, the overlay highlight. Poses, hats, colors, and sizes are in [DESKTOP.md](DESKTOP.md). Waiting looks like the reference, full eyes, not a slit.
- The walk: he has a home. Home is where a drag left him. A task walks him onto that window, a question brings him forward so YES is on screen, then he returns. Follow-the-work can be switched off, and then he stays home. He does not pick random points.
- The bubble: white card, pops when there is a line or a question, follows the ball, tucks when it is quiet. YES and NO are readable. The token is kept for the click and never written into a log.
- The desk window, if this build includes it: the sections in [DESKTOP.md](DESKTOP.md). It does not start at login. It is not the face.

Implement the calls in [WIRING.md](WIRING.md), or leave a view that an adapter can wrap with those exact methods. `app.py` already calls them. A second protocol is the wrong shape.

Leave these alone:

- The microphone, PipeWire, and the wake-word model.
- Whisper, the hearing rules, and the voice (ElevenLabs, Kokoro, Piper).
- The WebSocket, the SSH forward, and the device token.
- Desktop tools: open, focus, close, media, volume, scripts, typing, lock, screen look.
- Config files under `~/.config/orbi-desk/`. The voice key and the device token are 0600 files. Do not read them into a prompt, a log, or a commit.

The bubble and the desk window do not share CSS class names. The bug from doing that paints the dark window with the bubble's light text. Scope window rules under the window. Give the bubble its own class names.

Click on the ball mutes by closing the mic process. A drag past a few pixels moves him and is not also a mute. Right-click is color, hat, and follow-the-work. Esc dismisses the bubble. The ball never takes the keyboard.

Do not screenshot the whole desktop to judge the ball. The private desktop stays private. Window titles are screen text and stay on the machine.

## 2. Wire it up after

When the UI exists:

1. It lands as a view. It does not contain a copy of the body.
2. An adapter implements the [WIRING.md](WIRING.md) table and draws with the new animation.
3. `Desk` is given that adapter instead of the cairo HUD in `desk/orbi_desk/hud.py`.
4. The cairo ball stays until the new one shows the state, walks to a window, pops the bubble, and takes a yes.
5. Restart the body with `systemctl --user restart orbi-desk.service`. That is deaf for a few seconds while Whisper loads. The window unit stays disabled and does not start at login.

Do not start a second copy of the body. Do not `pkill` a pattern that matches the tool wrapper around `python -m orbi_desk`. Stop or restart the user unit.

After the restart, `orbi-desk status` should show a phase, `hud` true, the gateway up, and the mic on the internal Mic2 source. Do not print the thread, his lines, or a confirm token from that status.

Tests from the checkout, before calling it done:

```
cd ~/orbi/orbi-src
desk/.venv/bin/python desk/test_petlook.py
desk/.venv/bin/python desk/test_desk.py
```

`test_desk.py` warns on `listen.wake` and `end_silence_ms`. Those warnings are expected.

## 3. Run the body we have

The face and the ears are `orbi-desk.service` (`python -m orbi_desk run`). The board is `orbi-desk app`, from the desktop icon. Its unit is disabled.

```
systemctl --user status orbi-desk
journalctl --user -u orbi-desk -n 50 --no-pager
orbi-desk status
orbi-desk say what's the weather
orbi-desk mute | unmute | toggle
orbi-desk speak Hello.
```

`speak` is said in the room and sent nowhere. `say` is an utterance to the gateway.

Config is `~/.config/orbi-desk/config.toml`. Look is `look.json` (`color`, `hat`, `wander`, `size`). `wander: true` means follow the work. The voice key is `elevenlabs.key`, mode 0600, ORBI's own key, not a key copied from another app. The device token is `token`, mode 0600. Edit config, then restart the user unit. Do not commit those files.

Launcher is `~/.local/bin/orbi-desk`. It cds into the checkout's `desk/` directory.

Session on this machine: Plasma on Wayland, `WAYLAND_DISPLAY=wayland-1`, `DISPLAY=:1`, `XDG_RUNTIME_DIR=/run/user/1000`. The control socket is `$XDG_RUNTIME_DIR/orbi-desk.sock`, mode 0600. One body. A second `orbi-desk app` raises the window that is already open.

### Commands

`run`, `pair`, `app`, `toggle`, `listen`, `stop`, `mute`, `unmute`, `mute-toggle`, `show`, `hide`, `status`, `debug`, `board`, `say`, `speak`, `focus`, `volume`, `media`, `open`, `look`, `copy`, `cancel-timer`, `pet`.

`pet` is `pet color blue`, `pet hat crown`, `pet wander on|off`, `pet size 128`.

### Pairing

`orbi-desk pair` writes the token (0600, never printed) and registers its SHA-256 on the Dell over `ssh dell`. The body forwards `127.0.0.1:18125` to the gateway's `127.0.0.1:8125` and speaks `ws://127.0.0.1:18125/device`. Revoke on the Dell with `python -m orbi.devices remove desk` from `~/orbi`. Do not print the token. The public site in front of the gateway is for browsers. A body is not a browser.

### Voice

ElevenLabs, Roger, model `eleven_flash_v2_5`, 24 kHz PCM. If that fails, Kokoro `bm_george`, then Piper. Do not change the voice unless he names one. Do not put the key in `config.toml`.

### Microphone

The usable input is the Framework's internal stereo mic, Mic2 (`alsa_input.pci-0000_c1_00.6.HiFi__Mic2__source`). Mic1 is a constant full-scale pin and stays disabled in `~/.config/wireplumber/wireplumber.conf.d/51-framework-internal-mic.conf`. That WirePlumber file is machine-local. Do not commit it. `status` fields `mic`, `mic_level`, and `mic_dead_s` say whether the ear is alive. Mute closes `pw-record`. Locking the screen closes it too, and unlocking restores the mute from before the lock.

### Wake word

Model `desk/wakeword/orbi.onnx`, threshold 0.73, trained on synthetic voices. It does not yet know Noah's voice, and it does not yet hold him out of music or ignore other people. Super+Space is the sure way in. Do not run `record_my_voice.py` or the trainer unless he is at the mic and asks.

Hotkey name in KDE: "ORBI - Talk to ORBI" (System Settings, Shortcuts). Saying ORBI while he is talking stops the voice and drops the turn.

### Screen look

Only when he asks. One capture, in RAM under `$XDG_RUNTIME_DIR`, deleted immediately, shown to the local model on `127.0.0.1:8085`. Any other URL is refused. The answer is spoken here and is not sent to the gateway. A named app is matched by class and app id. The title is not the match and is not logged.

### Risky acts

Close an app, run a script, and type text ask first. The gateway sends them with `confirmed: true`. The body refuses them without that flag. Scripts run only by name from the allowlist in `config.toml`. The window does not call those three on its own.

### Typing

Not working yet. ydotool needs `/dev/uinput`. The running kernel on 2026-11-06 did not have that module. The package that has it is the newer CachyOS kernel. A reboot onto that kernel, a fresh login so group `input` is in the session, then restart `ydotool.service` and `orbi-desk.service`. Do not reboot unless he asks. Do not claim typing works before that.

### When something is wrong

- Never hears: hardware mic switch, or `mic_dead_s` climbing. Mic2 is the source.
- "Gateway away": `ssh dell` works without a password prompt, and `orbi.service` on the Dell answers `curl 127.0.0.1:8125/healthz`.
- No ball: the body waits up to a minute for Wayland. The journal names the failure. A layer-shell surface may not show up in a KWin window list. `status` with `hud: true` is the check.
- Right after a restart the socket refuses connections for a few seconds. Wait until `status` returns JSON with a `phase`.

## 4. Rules for anyone editing either repo

- Do not commit `.env`, keys, `elevenlabs.key`, the device token, `data/`, or WirePlumber config.
- Do not print a key, a token, a window title, or his sentences into a doc, a commit, or the journal.
- Do not push `noahoja-commits/orbi` unless he asks. This repo is the one these instructions belong in.
- Do not rewrite `~/briefing/projects-index.md` by hand. It is generated.
- Do not touch `orbi-ios-certs`.
- Do not run `desk/install.sh` again unless he asks.
- Do not train the wake word unless he asks.
- Plasma containment for the desktop icons stays `url=desktop:/`, `filterMode=1`, `filterPattern=*.livingicons-hidden`. Do not point it at an empty folder and do not clear the filter.
- The thread on the body is RAM only. It dies on restart on purpose.
- Tests that import GTK import Gtk 3.0 before anything else. Importing Gtk 4 first breaks `hud.py`.

## 5. Words

| Word | Means |
|---|---|
| Ball | The face. Blue sphere, two white capsule eyes |
| Bubble | The white card beside the ball |
| Desk | The window from the desktop icon |
| Body | This laptop client |
| Brain | The gateway on the Dell |
| Home | Where a drag left him |
| Follow the work | `wander` in `look.json`. Walk to the task, then home |
| Errand | One walk, with a rect or "forward", and a time limit |
