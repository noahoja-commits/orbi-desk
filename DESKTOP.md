# What a good desktop app looks like

The desktop app is the body you can see. It has two surfaces. They share one state, and they do not look like the same window.

The ball is the product. The window is the board you open when you want to drive it by hand. A good version makes the ball feel like a person who came over to the window he is working on, and makes the board feel like a quiet desk, not a chat client with a mascot glued on.

Noah builds the visual features. This file is the behavior and the shape those features have to land in.

## The two surfaces

```
desktop
  the ball          always there, layer-shell, about 128 px
  the bubble        only while there is something to read, follows the ball
  the desk window   opened from the icon, remembers its size, never at login
```

The ball and the bubble are two layer-shell windows, anchored top-left, placed by margins so they can move. The desk window is an ordinary GTK window. It talks to the body over the 0600 socket `$XDG_RUNTIME_DIR/orbi-desk.sock`. It never calls close, run-script, or type by itself. Those stay behind a spoken or clicked yes on a real confirm.

One instance. A second launch raises the window that is already open.

## The ball

Reference: the same face as `orbi_overlay` and `ORBI-reference.png` in the older checkouts. A glossy blue sphere. Two white capsule eyes. A small specular highlight on the upper left. No mouth, no blush, no cartoon "z". Expression is the eyes.

Poses, same numbers as the overlay:

| Moment | Eyes |
|---|---|
| Waiting, idle, asleep | Open capsules. This is the face people see all day |
| Listening | A little shorter, a little wider apart |
| Thinking, looking | Squint, tilted in |
| Working | Half height, lid down, tilted, looking at the work |
| Speaking | Slightly softer, a small bounce |
| Happy | The eyes arch |
| Angry, error | Lid down, tilt out, dimmer |
| Muted | Short and dim. The sphere goes grey |

Hats are clothes, drawn on top, and they are optional: none, beanie, cap, crown, propeller, bow, headphones. Colors besides blue exist (red, yellow, green, purple, pink, black). Blue is the ORBI sphere, with the overlay's gradient stops. Other colors reuse the same eye grammar on their own gradient. Sizes 96, 128, 176. Default 128, blue, no hat.

Blink is a short squash of the capsules, not a separate eyelid widget. Reduce-motion (KDE instant animations, or the config flag) drops the hop and the blink and keeps the pose.

### How he moves

He has a home. Home is where a drag left him, or the bottom corner the first time he appears.

`wander` in `look.json` means **follow the work**. On: a task moves him. Off: he stays at home even during a task. It never means picking random points on the screen.

| Task | Where he goes |
|---|---|
| Open, focus, close, look at a named app | Onto that window's lower edge. The rect is local. The title is not part of the target |
| A confirm, or he just started listening and no window is in play | Lower-middle, so the bubble and YES stay on screen |
| Speaking while already on a job | He stays with the job |
| The errand's time is up, or the turn is stopped | Back to home |
| Drag | Home becomes the drop point. The errand is cancelled |

Speed is a walk, quicker while an errand is active. He eases. He does not teleport. The bubble follows, and flips below him if he is too near the top.

An app that was just launched may not have a window yet. He looks again for a few seconds and goes when the rect exists.

### Input on the ball

- Click: mute. The mic process stops. Click again: on.
- Drag past a few pixels: move. The click is not also a mute.
- Right-click: color, hat, follow-the-work.
- He never takes the keyboard. Esc on the bubble dismisses the bubble after the bubble has been clicked.

## The bubble

A white card, about 320 px wide, with a tail toward the ball. It pops when there is a line, a card, or a question. It tucks away when it is quiet.

What it shows, and only this:

- His line, as captions, while he is talking and once the sentence is final.
- ORBI's line, growing a sentence at a time.
- A tool card: a label and one summary line. Success and failure look different, and the face matches (arch or the angry pose).
- A confirm: the question in words, YES and NO. The token is not printed.
- A diff card: the title the gateway sent, a count, a few file names. Not a raw patch dumped as a paragraph.
- A push: timer or nudge, one line.
- Dismiss.

The bubble's CSS classes are its own (`.orbi-pop-*`). The desk window's CSS is scoped under the window. Shared unscoped class names restyle one surface with the other's colors. That bug already happened once. New chrome keeps the two stylesheets apart.

YES has to be readable on the white card: dark label, clear hit target. The same yes/no path as the window and as a spoken yes.

## The desk window

Opened from the desktop icon. Not at login. Not a second face.

It is a native GTK3 window, dark, with the ball's blue as the accent and a yellow highlight used sparingly (the trace's hot line, not the whole UI). Type is small and quiet for labels, and the sentence he is reading is the largest text on the screen. Kickers are the section names: THREAD, OPEN, LAUNCH, PLAYING, PET, TRACE, HEARING. They stay uppercase and short.

Layout that already exists and should stay the layout:

**Main column, the turn.**

- THREAD. The in-memory exchange: you, ORBI, a tool, a diff, a note, a push. Newest at the end. A desktop tool result is not a second card on top of the bubble's card.
- HEARING. The live line while he is talking, right-aligned, gone when the sentence lands in the thread.
- The YES card, in the window as well as the bubble, when a confirm is waiting. Same token, same buttons.
- A type-in. Placeholder tells him he can type if he would rather not say it. SAY sends it as a typed utterance. STOP is the hotkey's job in button form.
- A timing label. How long the last turn took. Numbers, not a paragraph.

**Side column, the hands.**

- Meters. Wake score and mic level, so a dead mic is obvious. The node name can be the short hardware name.
- OPEN. App names of the windows on this desktop. Click focuses. Titles are not in the list.
- LAUNCH. His `[apps]` names, or the stock few when he has none.
- Volume, as a percent, and mute for the speakers. Separate from the mic.
- PLAYING. Prev, play, next. The label is the player name, ellipsized.
- LISTEN, LOOK, MIC, LOCK, HUD. HUD shows the bubble. LOCK locks the screen. LOOK asks the local model what is on screen.
- PET. Color, hat, size, FOLLOWS WORK / STAYS PUT. The selected chip is the only one that looks pressed.
- TRACE. A short scrolling log of phases. Collapsed in spirit: one small pane at the bottom of the side, not the product. His sentences are not the point of this pane. It is how you see "gateway away" and "didn't catch that."

The window remembers width and height only.

When the body is away the action buttons disable. The window says the body is away. It does not pretend a click worked.

### What "good" adds on top of that board

The board is already complete enough to use. A good desktop app makes it feel like one object.

- **One accent, one sphere.** Blue is the identity. The window does not invent a second mascot, a banner, or a reactor. If the ball is drawn again inside the window, it is the same cairo face at a smaller size, in the same pose the real ball is in. Two different faces is a bug.
- **The thread reads like the bubble.** Same words, same confirm, same tool line. The window can hold history the bubble has already tucked. It does not reword it.
- **Hands are one row of verbs.** Open, look, listen, lock, and the player should feel like the same family of buttons. Labels stay verbs. A feature that needs a paragraph belongs in the thread, not in the side column.
- **PET is a dressing room.** It is allowed to be delightful. It is not the top of the window. Follow-the-work is a switch with those two labels, not a "wander" toggle. Wander, as a word on the UI, is the wrong idea.
- **Motion belongs to the ball, not the window.** The window does not slide around the screen. The ball does, and only for a task. Flourishes (a hop on success, the propeller, the bubble tail) are welcome. Random roaming is not.
- **Empty states are one line.** No windows: "Nothing is open." No thread yet: the column is empty, not a tutorial. Gateway away: that sentence, and the buttons dead.
- **Confirm is the loudest card and the only red-adjacent one.** Everything else stays quiet. A yes that is hard to see is a broken feature.
- **Trace stays a tool.** It can be prettier. It cannot become a transcript archive.

### Room to invent

These are open, and they are the point of building desktop features:

- How the ball arrives at a window: the path, the hop onto the frame, what he does with his eyes while the app is opening.
- Hats and colors beyond the current set, as long as blue with no hat still matches the reference.
- The bubble's tail, shadow, and the way it pops and tucks.
- The window's materials: type, spacing, the meters, how OPEN rows feel. Dark, blue accent, white cards for things he must read.
- A smaller way to see the thread without opening the whole board, if the bubble is not enough. The ball stays the face.
- Sound design of the tiny listen chime. The voice stays Roger unless he names another.

### Do not

- Do not make the window the thing that auto-starts. The ball is what is there when he sits down.
- Do not put window titles, screenshots, the device token, or the voice key in the UI's logs, the journal, or a repo.
- Do not grim the whole desktop to test the ball. Capture the ORBI surfaces, or one named app, when a capture is actually needed.
- Do not let the bubble and the window share CSS.
- Do not send close, script, or type from a window button that skips the confirm.
- Do not roam.
- Do not draw a mouth onto the reference face. The arch is the smile.

## States the UI has to show

`asleep` and `idle` look like the reference ball at home. `listening` and `hearing` bring him forward if he is not already on a window. `thinking` and `working` use the squint, and working walks. `looking` is the local screen question: squint, on that app's window if one was named. `speaking` is the bounce and the bubble. `muted` is the grey short-eyed ball, and the mic button reads as off. Gateway down is a state of the link, said in the window and on the bubble's status, with the ball dimmer while he is idle.

Lock and mute interact. Locked means the mic is closed. Unlocked restores the previous mute. The UI should show mic off whenever the process is closed, whether he clicked or the screen locked.

## A turn, as the pixels see it

1. Idle ball at home. No bubble.
2. Wake or Super+Space. Eyes go to the listening pose. He comes forward if he was not on a job. A small chime.
3. Captions appear in the bubble as he talks. The window's HEARING line matches, if the window is open.
4. Thinking pose. Then sentences, spoken, in the bubble, hopping a little.
5. If the act is a window, he is already walking there, working pose, and the bubble can say "Opening Firefox."
6. If the act needs a yes, he is forward, the bubble has YES / NO, the window has the same card. Either one answers.
7. Success arches the eyes for a moment. Failure uses the angry pose and a short line.
8. Quiet. Bubble tucks. He returns home. The window keeps the thread until the process restarts.

That is the whole app. Everything else is a feature on top of this loop.
