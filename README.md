# ORBI desk

The extra ORBI. The gateway on the Dell is the brain. This is the body that lives on the Framework: it hears "ORBI", speaks in the room, and uses the laptop. The face on the desktop is a small glossy blue ball. A white speech bubble pops beside it when someone is talking, and tucks away when it is quiet.

The running code stays in [`noahoja-commits/orbi`](https://github.com/noahoja-commits/orbi), folder `desk/`. This repo is the context that repo's README does not carry, and the spec for what the desktop app should be.

| Read | For |
|---|---|
| [CONTEXT.md](CONTEXT.md) | What ORBI does, what the desk adds, a turn from wake word to answer, the device protocol, privacy, voice, and where the code lives |
| [DESKTOP.md](DESKTOP.md) | What a good desktop app looks like: the ball, the bubble, the window, and what is free to design |
| [WIRING.md](WIRING.md) | The seam. The new UI is look and motion. The body we have keeps the ears, the voice, and the hands, and calls the methods listed there |
| [AGENTS.md](AGENTS.md) | How to work in this repo without breaking the body |

## The picture

Two surfaces, one companion.

**The ball** is always on the desktop. It is the face. Click mutes. Drag sets where he waits. A task walks him to the window that task is about, then he comes back. He does not roam.

**The desk** is the app you open from the desktop icon when you want the board: the thread, the YES card, volume, what's playing, the open apps, launch, the pet's clothes, and a short trace. It does not start at login. It is not the face.

Say **ORBI**, or press **Super+Space**. Audio stays on this machine. Only the words go to the gateway.

## What he can already do from here

- **Run the computer.** Open an app, switch to it, list what's open (app names), pause, next, volume, play from the library on these speakers, lock the screen. Close, run a named script, and type all ask first.
- **See the screen.** "What's on my screen", "look at Firefox". A local model. The picture and the answer stay here.
- **Talk first.** Timers, nudges, and "Kimi finished" show on the bubble and are said aloud while you are at the desk.
- **Code with you.** Ask what an agent is doing, start or stop one (asks first), and show the diff as a card.

The commands that never need a model (timers, music, calendar, lists, weather, sums, undo) are in [CONTEXT.md](CONTEXT.md). The gateway README has the same table under "What ORBI does without a model".
