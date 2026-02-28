# Greg's Adventure - Complete Game Improvement Plan

## Critical Fix: Dialogue/Jump Input Conflict

**The Bug**: All triggered dialogues default to `block: false`, meaning the player can move during them. But Space is shared between jumping AND dialogue advancement. In the main loop, `movePlayer(dt)` runs BEFORE `updateDlg(dt)` — so every Space press during non-blocking dialogue BOTH sets the jump buffer AND skips/advances dialogue text. The player can't jump without accidentally blasting through dialogue, and can't read dialogue without accidentally jumping.

**The Fix**:
1. When dialogue is active (even non-blocking), **only Enter advances dialogue** — Space is reserved for gameplay
2. Add a `dlgInputCooldown` (~0.15s) after each dialogue transition so rapid Enter presses don't skip new lines instantly
3. Add a brief `jumpInputCooldown` (~0.1s) when blocking dialogue ends, so the Enter press that dismissed the last blocking dialogue doesn't accidentally register as a jump on the next frame
4. Show "ENTER ▶" instead of just "▶" in the dialogue box so players know which key to use

## Controls & Game Feel

### Movement Polish
- **Add acceleration/deceleration**: Currently movement is binary (0 or PLAYER_SPEED instantly). Add a lerp/acceleration curve — reach full speed over ~0.08s and decelerate over ~0.06s with friction. This makes movement feel responsive but not robotic.
- **Air control reduction**: Reduce horizontal acceleration while airborne to ~70% of ground acceleration. This rewards committed jumps and creates more skillful platforming.
- **Speed boost ramp-up for caffeinated mode**: Instead of an instant 1.5x multiplier, ramp it up over 0.3s with a screen-shake pulse at max.

### Jump Polish
- **Increase coyote time** from 0.1s to 0.12s — slightly more forgiving without feeling floaty.
- **Increase jump buffer** from 0.1s to 0.15s — catch more "early" jump presses, reducing frustration.
- **Add landing squash animation**: When Greg lands, briefly compress his sprite vertically (squash) for 0.1s, then snap back. Pure visual juice.
- **Add apex hang**: When Greg is near the peak of his jump (|vy| < threshold), reduce gravity by 30%. This creates the "floaty apex" feel that makes platforming precise and satisfying.
- **Edge correction ("corner correction")**: If Greg's head bonks a ceiling tile by just 1-3 pixels of overlap horizontally, nudge him sideways so the jump continues. This prevents frustrating "almost made it" moments.

### Camera Improvements
- **Look-ahead**: Offset the camera slightly in the direction Greg is facing/moving, so the player can see more of what's ahead.
- **Vertical camera smoothing**: Use a dead zone for vertical camera — don't track small vertical movements (like walking on uneven ground or small hops), only shift camera when Greg moves significantly vertically. This reduces nausea-inducing jitter.

## Enemy Improvements

### Smarter Patrol Behavior
- **Add brief pause at patrol endpoints**: Enemies pause for 0.3-0.5s when reaching the edge of their patrol range, then turn around. This creates small timing windows for the player to slip past or stomp.
- **Vary enemy speeds**: Currently all enemies use the same speed (60). Vary between 40-80 based on level to create different challenge patterns.
- **Add enemy anticipation animation**: Before an enemy turns around, briefly show a "!" or have it blink, giving the player a visual cue.

### Stomp Feedback
- **Add brief freeze-frame (hitstop)**: On enemy stomp, freeze the game for 2-3 frames (~50ms). This creates impactful, satisfying stomps.
- **Bounce height consistency**: Make the post-stomp bounce always sufficient to clear the enemy's body, preventing accidental damage after a successful stomp.

## Boss Fight Improvements

### Phase Variety
- **Phase 1 (8-5 HP)**: Keep current behavior but add a telegraph — the boss flashes/charges for 0.5s before firing, giving the player time to react.
- **Phase 2 (4-1 HP)**: In addition to dual projectiles, add a new attack pattern — the boss occasionally slams down, creating a shockwave along the ground that the player must jump over. Add office-themed yelling ("MANDATORY OVERTIME!", "SYNERGY REQUIRED!") as text labels on projectiles.
- **Add safe stomp windows**: After being stomped, the boss should be briefly stunned (1s) with a visual "dizzy" effect, during which it doesn't shoot. This rewards aggression.

### Boss Arena
- Spawn additional one-way platforms dynamically during the fight (floating TPS reports) to give the player more aerial options as the fight intensifies.

## Visual Juice & Polish

### Player Animation
- **Squash/stretch on jump and land**: Compress sprite horizontally on jump launch, vertically on landing.
- **Trail effect when caffeinated**: Draw 3-4 afterimage shadows of Greg trailing behind him during caffeine mode.
- **Death animation**: Instead of just respawning, have Greg ragdoll briefly or do a dramatic spin before the level resets.
- **Idle animation**: After 3 seconds of no input, Greg checks his watch or sighs. Reinforces character.

### Screen Effects
- **Hit feedback**: Flash the screen border red briefly on damage (not the whole screen — just a vignette).
- **Level transition**: Add a brief "wipe" transition instead of just a fade, matching the game's retro energy.
- **Coin collect satisfaction**: Scale up the coin briefly before it disappears (pop effect), add a "+1" floating text.

### Enemy Death Animation
- Currently enemies just disappear. Add a brief "poof" — shrink and spin the enemy sprite over 0.3s, then remove it.

## Level Design Tweaks

### Level 0 (Tutorial)
- The level is good as a tutorial but the trigger zones are too wide (2-tile radius). Tighten to 1.5 tiles so dialogue triggers when the player is actually AT the relevant thing, not running past it.

### Level 1 (Office)
- Add more vertical variety — some sections where the player needs to climb UP using platforms, not just traverse left-to-right on the ground.
- Add a "water cooler" decoration entity that spawns idle office chatter particles.

### Level 2 (Narrator Snaps)
- The glitch effects timer (`glitchT += 0.016`) is hardcoded instead of using `dt`. Fix this to prevent timing issues on different frame rates.
- The fake error popup timer also uses hardcoded 0.016 instead of dt. Fix this too.
- Make the enemy spawn wave from `snap_glitch` more dramatic — stagger them over 2 seconds instead of all at once, with each one accompanied by a glitch flash.

### Level 3 (Boss)
- The arena is too flat. Add 2-3 more one-way platforms at varying heights to give the player more options for dodging and reaching the boss.

## Audio Improvements
- **Add a simple background beat**: A minimal procedural chiptune bass line using Web Audio API oscillators — just a 4-bar loop that changes per level. Not a full soundtrack, just ambient rhythm.
- **Pitch variation on coin pickup**: Slightly increase the pitch of each successive coin pickup in a combo (resets after 2 seconds of no coins). Creates a satisfying ascending arpeggio.
- **Boss hit sound escalation**: Each stomp on the boss makes a slightly more dramatic sound (higher pitch, longer duration).

## Quality of Life

### Checkpoint System
- Add a mid-level checkpoint on levels 1 and 2 (the longer levels). When the player dies, respawn at the checkpoint instead of the level start. Mark checkpoints with a funny sign ("SAVE POINT — Management Not Responsible For Lost Progress").

### Controls Display
- Show a small controls reminder in the corner during the first 10 seconds of Level 0: "← → Move | SPACE Jump | ENTER Talk".
- Show controls on the title screen too.

### Pause
- Add pause functionality on Escape key. Show a simple overlay: "PAUSED" with Greg saying "Can I stay here forever?" and the narrator responding "NO."

## Implementation Order

1. **Dialogue/Jump conflict fix** — highest priority, directly requested
2. **Movement acceleration/friction + jump polish** — core game feel
3. **Hitstop + stomp feedback** — satisfying combat
4. **Boss fight improvements** — end-game polish
5. **Camera improvements** — comfort
6. **Visual juice (squash/stretch, death anim, coin pop)** — polish
7. **Enemy improvements** — gameplay variety
8. **Glitch timing fixes** — bug fix
9. **Audio improvements** — atmosphere
10. **Pause + controls display** — QoL
11. **Checkpoint system** — reduce frustration
12. **Level design tweaks** — content refinement
