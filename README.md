# Zhess X

## Render settings
- Runtime: Node
- Build Command: `npm install`
- Start Command: `npm start`
- Root Directory: leave blank

## Online gameplay included
- Create/join a 6-character room code.
- Both players must mark Ready.
- Each player independently selects their own roster (no duplicate unit type within that roster; population cap 15).
- When both rosters are submitted, the server starts the match and randomly chooses the first turn.
- Both players see the same board state. Player 2 sees the board rotated 180 degrees so their own side is at the bottom.
- Moves, skill effects, cooldowns, HP/status changes, and turn changes are sent to the other player through Socket.IO.
- The previous bot is disabled during online matches; offline bot logic remains in the original game code.

## Art and effects
- All pictures are SVG files in `public/img/` (`char/<unit>.svg` for the 9 characters, `icon/<name>.svg` for skill/status/UI icons). To change a picture, replace the file with the same name (keep a 64x64 viewBox for characters, 24x24 for icons). The frame, team glow and skill effects are still drawn by script in `public/index.html`. Adding a new unit = add `img/char/<key>.svg` and its entry in the `R` table.
- Every skill has its own SVG effect (about 1-2 seconds). Effects are sent to the opponent together with the game state, drawn on the rotated board for Player 2, and effects of an invisible (Wind-Walk) unit's dash/teleport are not shown to the opponent.

## v3 additions
- New units: Red Scales (Epic, pop 6, always flying, lands after any skill until its next turn) and Juggerknight (Epic, pop 4, 50% parry vs physical, Crushing Leap).
- Tag system: skills carry tags (phys, magic, melee, ranged, ground, fire, water, sword ...) and units carry tags (dragon, human ...). Ranged = range > 1 or the ranged tag without melee. Ground-tagged skills cannot hit flying units.
- Arena: every match randomly gets one of 4 themes (ancient stone, night castle, lava, glowing forest); the server picks it so both players see the same one.
- Turns: a banner announces your turn, and each turn lasts 60 seconds (shown next to MP/AP). At zero the turn is ended automatically; if that player's device is stuck, the opponent's device ends it 6 seconds later.
- Tags: a skill counts as ranged only if it carries the `ranged` tag (reach above 1 cell does not count), so grounded Red Scales can still be hit by long-reach melee skills like Crushing Leap.
- Characters are faceless silhouettes on element-coloured backgrounds, in a metallic frame (shape by rarity, colour by element) with a blue/red team glow.

## Important limitations
- This is an early playable PvP implementation using client-generated state snapshots relayed by the server. It is suitable for private testing with trusted players, not cheat-resistant competitive play.
- Deployment: after both rosters are confirmed, each player secretly places their units on their own two back rows (20 cells) within 60 seconds (countdown shown). Both layouts are revealed together when both confirm; anyone who runs out of time gets the remaining units placed at random. Offline games use the same screen, and the bot places its army at random.
- Rooms and matches are stored in memory and are lost if the Render service restarts.
- Rematch: after a match ends, both players can press "เล่นอีกครั้ง" to return to roster selection in the same room. If someone leaves or loses connection, the other player goes back to the lobby; a dropped connection closes the old room (reconnecting into a running match is not implemented yet).

## v4 additions
- Beast Hunter (Epic, HP 90, move 1, pop 5, jade-green silhouette, uses the existing `wind` frame colour). Jade Katana (melee, 6; 12 + Bleeding 2 turns vs dragons/monsters), Qi Tian Slash (3-cell line in any of 8 directions, 5 per cell, 9 vs dragons/monsters, hits every enemy in the line, never allies), Awakening (3 turns: move 2, +1 damage / +3 vs dragons/monsters, takes only 10% from dragons/monsters; CD 4 counted after it ends). Both attacks can hit flying dragons/monsters even though they carry the ground tag (skill flag `aa`).
- Beast Hunter passives: takes +50% from ranged skills (not from dragons/monsters), and Curse of Master: no shields/helpful statuses from teammates; healing still works but is halved (rounded, min 1), curing Bleeding/Poison still works. Unit flags: `rngIn`, `healMul`. "Beast" = unit tag `dragon` or `monster`.
- Hidden units (Assassin Wind-Walk) can now be stepped on by enemies with any kind of movement (walk, Legging-Dash, Teleport, Asteroid). The hidden unit is revealed and pushed to the free neighbouring cell farthest from the mover (or swapped onto the mover's old cell if none is free); the mover takes 5 damage, Bleeding 2 turns and is stunned, and the Assassin is stunned too. A hidden unit's own team still cannot step on it. Because hidden cells are now valid destinations, the blue move highlights no longer reveal where a hidden unit is.
- New art: `img/char/hunter.svg`, `img/icon/katana.svg`, `qislash.svg`, `awaken.svg`.
- Lobby has a "เล่นกับบอท (ออฟไลน์)" button (works without the server); the roster screen has a "กลับหน้าแรก" button to return.
- Ocean Leviathan (Epic, HP 160, move 1, pop 5; unit tags legend/monster/water, frost frame). Ocean Spear (range 2, single target, 5, 10% stun), Tide Shield (shield 10 on an ally within 2 or itself; the caster is silenced for 2 turns including the cast turn, and the 2-turn cooldown starts when the silence ends or is cured), Whirlpool (pick any cell: 5x5 vortex that pulls enemies 1 step toward the centre on cast and at the start of each of the caster's turns, 5 pulls in total; 3 damage the first time a unit is hit, 2 afterwards, 5% stun; ground+ranged tags so flying units and grounded dragons are unaffected; cooldown 4 starts when it ends). Passive Iron Body: stun chance -10 points, ranged -20%, melee +20%, water -30%, earth/wind +50% (the Archer's skills now carry the `wind` tag). The vortex lives in the shared `VX` list that is sent with every game state. I assumed Tide Shield may target any ally within 2 (or itself), and that the pull hits hidden units too (revealing them); Beast Hunter cannot receive the shield (Curse of Master).

## v5 additions
- Nimbora (Epic, HP 100, move 1, pop 5, water/wind, new `storm` frame colour). Files: `img/char/nimbora.svg`, icons `nimbus`, `lfield`, `zonecloud`, `cdash`, `overload`; `nimbora: 5` added to `POPULATION` in `server.js`.
- Cloud Domain (passive): the 3x3 around Nimbora (5x5 in Super Zone) is her team's cloud and moves with her. Zone Created adds permanent cloud cells that stay on the board. Clouds belong to a team, may overlap, and never take over each other; a cell shows the viewer's own cloud if both exist. Mine = blue with a hint of green, the opponent's = blue with a hint of yellow. Two faint wisps drift between random cloud cells of the whole board. If Nimbora dies, permanent clouds stay but rain and skills stop.
- Nimbocumulus: pick an own cloud cell within 3, 3x3 rain zone, rains once at the start of her next turn, CD 1. Super Zone replaces it with Lightning Field: bolt on an own cloud cell, a second bolt on a random enemy within 3 of it, 30% for one extra bolt on a random cell within 3 (it needs no target); CD 1.
- Zone Created: pick a cell within 2 to become permanent cloud. Stage 1 free, stage 2 costs 2 AP, stage 3 free, then locked until the next turn; any unfinished stage also resets to 1 at the next turn start. During Super Zone this slot becomes Cloud Dash (move to an own cloud cell within 3, no MP/AP, CD 2).
- Overload: Super Zone for 4 turns not counting the cast turn (personal cloud 5x5, every cloud cell rains at the start of each of her turns; CD 4 starts when it ends).
- Rain: allies in the rain heal 1-2 (Beast Hunter halved), enemies take 1-2 unblockable water damage (not hidden units); 20% for one random enemy in the rain to be struck by lightning. Lightning Strike: 8 magic (ground tag), 40% chain 6, then 20% chain 4; chains hit any non-hidden enemy (flying too) and never the same unit twice. Bolts never hurt allies.
- State: `CZ = {pc, rz}` is sent with every game state like `VX`. The bot can use Nimbora.
