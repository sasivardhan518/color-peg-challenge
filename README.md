# Colour Peg Challenge (hidden-colour, multiplayer edition)

A single-file digital wooden peg board game (HTML5 + CSS3 + vanilla JS, no dependencies).

## Run it locally
- Double-click `index.html` (works from `file://`), or
- `python3 -m http.server 8000` in the folder, then open `http://localhost:8000`.

For phones, host the file on any static host or open it over your Wi-Fi via the server above. It is pass-and-play: everyone shares one screen.

## Layouts
- **Desktop / tablet:** the full layout (title, player-count picker, 5 stat tiles including whose turn it is, scoreboard, board, 6 control buttons and a hint line).
- **Phones** (screen narrower than 700px, or a short landscape screen): a compact layout that always fits one screen with no scrolling. Extras are hidden (turn tile, missed-count, hint, duplicate Start/Resume buttons), the board takes all the remaining space, and the player-count picker moves onto the board's Ready screen. Sideways, the board sits on the left and everything else in a column on the right. The switch is one CSS block near the end of the stylesheet (`9. LAYOUT HELPERS + PHONE LAYOUT`); change the breakpoint there.

## Rules (as implemented)
1. All pegs are placed randomly and **their colours are hidden** (plain wood tops).
2. On your turn, tap the dice. It rolls to a colour.
3. Pick pegs blindly. A peg of the rolled colour is yours (+1) and you **keep picking the same colour** until you have found them all or pick a wrong one.
4. **Wrong peg:** its colour is flashed to everyone for about 1.3 seconds (so players can memorise it), it goes back hidden, and **the turn passes** to the next player.
5. **Dice shows a colour that is already finished:** the turn passes to the next player.
6. **You collect the last peg of a colour:** you get **another roll**.
7. **2 to 6 players.** When the board is empty, the player with the most pegs wins (ties are shared).

Choose the player count with the buttons at the top (2 to 6).

**Names and pictures:** before the game starts, tap any player in the scoreboard to open the player editor:
- type a name (up to 14 characters)
- **Take photo** opens a live camera preview right in the editor (front camera by default, with a flip button when the device has more than one camera). Tap **Capture** to use the shot; **Back** returns without it. The camera turns off as soon as you capture, go back, or close the editor. If the browser can't open the in-page camera (blocked permission, an in-app browser, or a non-secure page), the button falls back to the device's own camera app
- **Choose photo** picks a picture from the gallery or files
- Either way the picture is cropped to a square, shrunk to 128px and stored only in your browser (about 2 to 8 KB each), so nothing is uploaded anywhere
- or pick one of 12 animal emojis
- **Remove picture** goes back to the player's initial letter

The avatar shows in the scoreboard, the "Turn" tile, the final ranking, and beside the message under the board, so everyone can see whose turn it is to roll (when a turn passes, the next player's picture appears). Names, pictures and player count are remembered next time (per player slot, so Player 3's picture comes back when you choose 3 or more players). Editing is locked during a game.

## Architecture
All code is in the `<script>` in `index.html`, in numbered sections:

| # | Section | Key functions |
|---|---------|---------------|
| 1 | `CONFIG` | colours, dice faces, board rings, players, rules, timings |
| 2 | `state` | one central state object: `players`, `current`, `pegs`, `status`, `phase`, `selectedColour` ... |
| 3 | Utilities | `shuffle`, `shade`, cancellable `later()` timers, safe `store` |
| 4 | `Sound` | WebAudio tones, no audio files |
| 5 | Board | `generateHolePositions()`, `createBoard()` |
| 6 | Pegs | `generatePegs()`, `renderPegs()` |
| 7 | Dice | `createDice()`, `rollDice()`, `chooseFace()`, `finishRoll()` |
| 8 | Gameplay | `selectPeg()`, `validatePegSelection()`, `removePeg()`, `rejectPeg()`, `nextPlayer()`, `updateScore()`, `checkGameCompletion()` |
| 9 | Flow | `updateTimer()`, `startGame()`, `pauseGame()`, `resumeGame()`, `restartGame()`, `endGame()`, `initializeGame()` |
| 10 | UI | `updateHUD()`, scoreboard, `fillAvatar()`, player editor (`openPlayerEditor()`, `readPhoto()`, `applyProfile()`), `say()`, overlay, end screen |
| 11 | `Confetti` | canvas particles |
| 12 | Events | one delegated click handler for all pegs |

The game is a small state machine: `status` (idle / playing / paused / won) and `phase` (awaitRoll / rolling / awaitPick / resolving). Buttons and clicks are enabled purely from these two values, which is what stops a player rolling mid-turn.

Each rule lives in one function: rules 3 and 6 in `removePeg()`, rule 4 in `rejectPeg()`, rule 5 in `finishRoll()`, turn order in `nextPlayer()`. Hidden colours are a CSS class: a peg's colour layer only becomes visible with `.revealed`.

## Customising (all in `CONFIG`)
- **Peg colours:** `CONFIG.colours` (e.g. add `purple: { name:'Purple', hex:'#8E24AA' }`). Put it on the dice by editing `CONFIG.dice.faces`. Pegs are split evenly between colours, so a number of pegs divisible by the colour count is fairest.
- **Board positions:** `CONFIG.board.rings` (hole count, radius, rotation per ring; board edge = radius 100). `jitter` and `seed` change how organic it looks. For a hand-drawn layout use `customHoles: [{x:0,y:-80}, ...]`. `holeSize` is the hole diameter as % of board width. Keep holes about 25 units from the centre so the dice has room.
- **Number of pegs:** `CONFIG.pegs.count` (null fills every hole).
- **Players:** `CONFIG.players`: `min`, `max`, `default`, the ring colour `palette` per slot, `nameMaxLength`, `photoSize` (px) and the quick-pick `emojis` list. From code, `ColourPeg.applyProfile(i, {name, emoji, photo})` sets a profile programmatically.
- **Rules:** `CONFIG.rules`:
  - `pointsPerPeg` (default 1: score = pegs collected)
  - `hidePegColours` (false = easy mode with visible colours)
  - `revealWrongMs` (how long a wrong peg shows its colour)
  - `rollAgainWhenColourCleared`
  - `finishedColourPassesTurn` (false = just roll again)
  - `rollOnlyAvailableColours` (true = the dice never shows a finished colour)
- **Speeds:** `CONFIG.timing`.

## Assumptions (the rules left these open; all configurable)
- "If all colours are picked while he is playing, he gets another chance to roll" is read as: when you collect **every peg of the rolled colour**, you roll again.
- A wrong peg is **revealed to everyone briefly, then hidden again** (that is what makes it a memory game). It stays on the board.
- The cube has 6 faces but there are 5 colours, so one colour appears twice. The roll picks a **colour** uniformly first, so the repeated face does not make that colour more likely.
- 25 holes and 25 pegs (5 of each colour).
- Score is **1 per correct peg**, so a player's score is the number of pegs they collected; the winner is whoever has the most when the board is empty. Change `pointsPerPeg` for other values.
- The earlier Challenge mode (timer and streak bonus) was removed because it conflicts with these turn rules. The clock still counts up for the end-screen stats.
- Pause is disabled during the roll and peg animations (about 1.5 seconds). A hidden browser tab queues an auto-pause.

## Testing done
Headless tests (jsdom plus real Chrome, 56 checks) covered:
- 25 non-overlapping holes, 5 pegs per colour, colours not leaked in labels or attributes, pegs re-randomised on restart
- 2 to 6 players, the name/picture editor (emoji, photo, cancel, blank and over-long names), profiles saved in localStorage and kept across restarts
- a real photo upload in Chrome (non-square image cropped and saved at about 3 KB), a non-image file rejected with a message, and the photo and name restored after a page reload
- the camera, with a fake webcam in Chrome: live preview, capture, saved to the player, camera tracks stopped on capture / Back / closing the editor; permission denied shows a message and the next tap falls back to the camera-app input; no camera API at all goes straight to that input
- roll lock while picking, no picking before rolling
- wrong pick: reveal, re-hide, turn passes, no points
- correct picks: same player and same colour continue
- clearing a colour gives another roll
- rolling a finished colour passes the turn, and the turn wraps round to the first player
- pause and resume with a frozen clock
- full games to the end screen with winner and ranking, Play Again, Restart
- a randomised game that finishes, sound toggle
- on phone sizes (375x667, 360x640, 390x844, 844x390 landscape), with 6 players and long names: no scrolling and every element inside the screen. No sideways scroll on any size
- real mouse and touch clicks, and zero console errors
