# ORBI desk — agent guide

This repo is the **context and the desktop-app spec** for the Framework body of ORBI. It is not the running code.

- Implementation: [`noahoja-commits/orbi`](https://github.com/noahoja-commits/orbi), folder `desk/`. On this machine the checkout is `~/orbi/orbi-src` (branch `desk`).
- Brain: the same repo's gateway, running on the Dell. Read that repo's `AGENTS.md` and `README.md` before changing behavior.
- Protocol: `docs/DEVICE-PROTOCOL.md` in the gateway repo.
- This repo: `README.md` (map), `CONTEXT.md` (what it does), `DESKTOP.md` (what the desktop app should look like).

## How to work

- Noah is building the desktop features (motion, materials, the window's craft). Read `DESKTOP.md` before changing the ball, the bubble, or the window, and keep the "Do not" list.
- Functionality changes (turns, tools, privacy, the protocol) land in `noahoja-commits/orbi`, not as a second copy of the Python here.
- Never commit `.env`, keys, the device token, `elevenlabs.key`, `data/`, or WirePlumber config. This repo should stay prose.
- Do not print tokens or API keys into a doc, a commit, or a log.
- Do not push the gateway repo unless he asks. This repo is the one these docs belong in.
- The desk tests are `desk/test_desk.py` and `desk/test_petlook.py`, run from the checkout. A visual change to the ball is not done until those still pass and the body has been restarted with `systemctl --user restart orbi-desk.service`.
- Do not reboot, do not run `install.sh`, and do not train the wake word unless he asks.

## Words that already mean something

- **Ball** — the face. Glossy blue sphere, two white capsule eyes.
- **Bubble** — the white card that pops beside the ball.
- **Desk** — the window opened from the desktop icon.
- **Follow the work** — `look.json` `wander`. He walks to the task, then home. He does not roam.
- **Home** — where a drag left him.
- **Body** — this laptop client. **Brain** — the gateway.
