# Stalingrad Sniper: Game Design Document

| | |
|---|---|
| Current build | 1.7 |
| Platform | Single HTML page (Three.js r128), desktop browser first, touch supported |
| Owner | Petr |
| Last updated | 2026-10-28 |

This document is the source of truth for the game. Every change starts here: update the design on a feature branch, then change the code, then tag the build (see [Change process](#11-change-process)).

---

## 1. Vision

A realistic, slow-paced WW2 sniper game. The player wins with fieldcraft and ballistics, not reflexes. The game has two halves:

1. **Stalk.** A top-down view of no-man's-land. You sneak forward unseen and pick the best firing position.
2. **Shoot.** You go prone in the hide and observe through binoculars. You range the target in mils, read the wind, set the rifle's elevation drum and windage, control your breathing and take the shot. Then you get out before the mortars find you.

Real sniper doctrine drives every system: movement and silhouette betray you, the military crest beats the skyline, a spotter calls corrections, and the rule is shoot and move. The game teaches this through the tutorial and the shooting range.

### Design pillars

- **Authentic, not arcade.** Real rifles, scopes, reticles and ammunition. Ballistics come from simulated drag and wind, not from canned values.
- **Readable.** Every hidden number (visibility, detection, exposure, firing solution) is shown in the HUD or the diagrams at Recruit difficulty.
- **Learn by doing.** The tutorial runs on the real map with in-world markers and in-view visuals.
- **One page, no install.** Everything is in one HTML file that can be shared as a link.

## 2. Setting and story

Red October factory district, Stalingrad, 12 November 1942, 07:40. You are a sniper of the 62nd Army with your spotter, Pvt. Orlov. Every morning a German battalion officer walks from his HQ to an observation post behind the forward trench, where he directs the machine guns pinning your company down.

**Orders:** get into no-man's-land unseen, choose a hide, kill the officer, and return alive to your own trench in the south.

## 3. Game modes

| Mode | Purpose | Notes |
|---|---|---|
| Mission | The main game | Choose a difficulty and a rifle on the briefing sheet |
| Tutorial | Teaches the full drill in 18 steps | Always issues the Mosin 91/30 with the PU scope. Being spotted sends you back to the last checkpoint instead of ending the game |
| Shooting range | Free practice with any rifle | Short lane from 100 to 600 m plus a moving target, and a long lane from 800 to 2,500 m |

### Difficulty (mission)

| Level | Help |
|---|---|
| Recruit | Orlov reads range and wind and shows the full firing solution |
| Marksman | Press E to ask Orlov. His estimates can be a few percent off |
| Veteran | No help. Range with the mil scale and read the flag and smoke yourself |

## 4. The map

- A terrain heightmap grid from x = −450 to 451 and z = −500 to 451 m. North (−z) is the enemy.
- **Our trench** is in the south (z ≈ 348). A **ridge** crosses the middle (crest near z ≈ 152), and no-man's-land has shell craters, tall grass, bushes, rubble, wrecks, a ruined farmhouse and a ruined barn.
- The **enemy line** has a forward trench with a barbed-wire belt (z ≈ −232), an observation post (OP), a battalion HQ and a factory with a sniper in its windows.
- **Props:** a road with telegraph poles, a burning lorry with smoke, a white flag on the HQ roof, oil drums, crates, wrecks and standing water in craters.
- **Ambience:** a low morning sun in the east-north-east (azimuth 72°, elevation 13°), fog, artillery fire, MG bursts and smoke.

## 5. Stalk phase

### Controls

WASD to move (north is up), or click the ground. Shift runs, C crouches, Z crawls, M opens the overview map, Space assesses a hide, H opens the field manual.

### Visibility and detection

Player visibility = stance factor × cover factor × movement factor × skyline factor.

- **Stance:** standing 100%, crouching 60%, crawling 30%.
- **Cover** (while crawling): open mud ×1, tall grass ×0.28, crater ×0.25, rubble ×0.35, bush ×0.18.
- **Movement:** still ×0.35, crawling ×0.6, crouching ×0.85, walking ×1, running ×1.5.
- **Skyline:** ×1.5 when standing or crouching on the crest.

Each watcher (a trench sentry, the listening-post sentry, a patrol member) has a sweeping view cone with its own field of view, range and sweep speed. Inside a cone, with a clear line of sight, detection rises by visibility × (1 − distance / range)^0.7 × 2.4 per second.

Noise adds to detection. Running is heard within 50 m, walking upright within 16 m and crouching within 7 m.

Each enemy's attention meter decays when he cannot see you. At 100% you are spotted and the mission fails, with a lesson explaining why. In the tutorial you are reset to the checkpoint instead.

### Enemy roster

| Unit | Points | Behaviour |
|---|---|---|
| Battalion officer | 500 (+200 bonus) | Walks from HQ to the OP, observes, flees under fire |
| Enemy sniper | 300 | In the factory windows. Shows lens glints and hunts you after your first shot |
| Signaller | 200 | Crouched at the radio |
| MG gunner and loader | 150 each | In the trench |
| Trench sentries (2) | 100 | Watch cones of 44° out to 250 m |
| Riflemen (2) | 100 | In the trench |
| Listening-post sentry | 100 | Wide 70° cone out to 165 m |
| Patrols (2 pairs) | 100 | Walk set routes. They search toward the shot after you fire |
| Runner | 100 | Runs between positions |

Units use jointed, animated rigs with stride-phased gaits: walk, run, crouch-walk, crawl, aim and radio.

## 6. Hide assessment

Prone in a candidate spot, you press Space and Orlov grades it from 0 to 100 on these criteria:

| Criterion | What it rewards |
|---|---|
| Field of fire | Which enemies, and the officer's post, can be seen |
| Concealment | Bush > grass > crater/rubble > open |
| Range | Best at 250–560 m. Closer is heard, further is hard for a 3.5× scope |
| Landmarks | Penalty next to obvious features such as a lone tree or a ruin |
| Skyline | Forward slope below the crest (the military crest) is ideal |
| Withdrawal route | Cover behind you |
| Sun | Penalty when looking into the low sun, which causes glint |
| Known position | −30 if the enemy has already located the area |

A hide that scores 70 or more is good. Press Enter to occupy it.

## 7. Shoot phase

### Views

1 is the naked eye, 2 is 6×30 binoculars with a mil scale (horizontal ±40 mils and a vertical 0–10 mil stadia), and 3 is the rifle scope. Right-click cycles the views.

### Rifle handling

- W/S or the mouse wheel step elevation by one click, and R/F step 10 clicks. Holding a key repeats it.
- A/D turn the windage one click at a time.
- Hold Shift to hold your breath. The reticle steadies, but after 6 seconds you start to shake.
- Pulse adds a heartbeat twitch and settles after running.
- Bolt-action rifles cycle between shots and self-loaders cycle quickly. A reload starts when the magazine is empty.
- E asks Orlov for range and wind. X leaves the hide.

### Ballistics model

- A point-mass bullet with quadratic drag, dv/dt = −KD·|v_rel|·v_rel − g, where v_rel is the velocity relative to the moving air.
- Each rifle sets its muzzle velocity (MV), its drag constant KD (fitted to the issue round's retained velocity near 500 m) and its sight height.
- The drum and turret settings are solved from the same simulation, so the solution, the HUD and the bullet always agree.
- In flight the bullet is integrated in five substeps per collision step. It hits soldiers' body parts (head ×1.5 points), cover, walls, sandbags or the ground.
- Shot dispersion is Gaussian with σ = group size (MOA) ÷ 3, plus a muzzle velocity spread of 0.4%.
- **Moving-post scopes** (the ZF39) lower the post instead of moving the image, so you still aim with the post tip.

### Wind

Wind blows from a base direction and speed, with gusts that vary its direction and speed. Clock values follow the sniper clock method. Only the part of the wind that blows across the bullet's path pushes it sideways, and that part is the wind speed × sin(angle off the line of fire):

| Wind from | Angle off the line | Value | Wind to use |
|---|---|---|---|
| 3 or 9 o'clock | 90° | Full | 100% (10 mph → 10 mph) |
| 2, 4, 8 or 10 o'clock | 60° | Seven-eighths, close to full | ≈ 7/8 (10 mph → 9 mph) |
| 1, 5, 7 or 11 o'clock | 30° | Half | 50% (10 mph → 5 mph) |
| 12 or 6 o'clock | 0° | None | 0 (head or tail wind) |

The physics uses the exact crosswind component, so these values match what the bullet does. The part of the wind along the line of fire (head or tail) changes the drop by only a few centimetres at these ranges and is ignored when dialling. The flag's angle ÷ 4 gives the speed in mph, and smoke leans downwind.

### Firing solution (Recruit, or the range with help on)

The solution gives the elevation setting (the drum or turret value), the remaining hold in cm or m, and the windage in the scope's units. A rifle with no windage turret gets a hold-off in mils and cm instead. The PSO-1 shows chevron hints from 1,100 to 1,300 m. Moving targets get a lead in metres.

### Spotter

Orlov watches each impact and calls the miss in cm, high or low and left or right, with the correction in mils, or in MOA on MOA scopes.

### Exposure and counter-sniping

An exposure meter fills from:

- each shot, scaled by the hide's quality and its cover dust
- lens glint when the scope faces within 30° of the sun
- the enemy sniper hunting you

Exposure decays slowly. Shots fired while our artillery is firing count only ×0.35 (masking). At 100% the mortars start, and you have 10 seconds to leave before they hit.

### End of mission and scoring

- **Win:** the officer is dead and you are back in our trench.
- **Lose:** you are spotted, or caught by the mortars.
- **Score:** kill points, plus 300 for the win, plus 3 × the average hide score, plus 2 × accuracy %.
- **Medals:** Medal "For Courage", Order of the Red Star (above 1,100), Order of the Red Banner (above 1,500) or Order of Lenin (above 1,900).

## 8. Rifles and scopes

All six WW2 rifles can be used in the mission. The five modern rifles are included for comparison and range practice and can also be taken into the mission.

| Rifle · scope | Round · MV | Scope | Elevation · windage | Magazine · group |
|---|---|---|---|---|
| Mosin 91/30 · PU | 7.62×54R light ball 9.6 g · 865 m/s | 3.5×, 4.5° field, post and side bars | Drum 1–13 (×100 m) · 0.5-mil clicks | 5 · 1.3 MOA |
| Mosin 91/30 · PE | same · 865 m/s | 4× (3.87×), 5.5° field, post and fine cross-line | Drum 1–14 · 0.5-mil clicks | 5 · 1.2 MOA |
| SVT-40 · PU | same · 835 m/s | 3.5×, 4.5° | Drum 1–13 · 0.5-mil clicks | 10, self-loading · 2.6 MOA |
| Kar98k · ZF39 (captured) | 7.92×57 s.S. 12.8 g · 760 m/s | 4×, 7°, Absehen 1, moving post | Drum 100–800 m · none, hold off | 5 · 1.3 MOA |
| No.4 Mk I (T) · No.32 | .303 Mk VII 11.3 g · 744 m/s | 3×, 8°, post and cross-line | Drum 100–1,000 yd in 50 yd clicks · 2 MOA clicks | 10 · 1.5 MOA |
| M1903A4 · M73B1 | .30-06 M2 ball 9.7 g · 853 m/s | 2.5×, 8.2°, crosshair | 1 MOA clicks from a 100 yd zero · 1 MOA clicks | 5 · 2.2 MOA |
| SVD · PSO-1 | 7.62×54R 7N1 9.8 g · 830 m/s | 4×, 6°, chevrons, 1-mil scale, rangefinder | Drum 1–10 in 50 m clicks · 0.5-mil clicks | 10, self-loading · 1.2 MOA |
| M24 SWS · Leupold M3A | 7.62×51 M118LR · 790 m/s | 10×, 3.5°, mil-dot | BDC 1–10 (×100 m) · 1 MOA clicks | 5 · 0.75 MOA |
| M110 SASS · Leupold Mk 4 | 7.62×51 M118LR · 783 m/s | 10×, 3.6°, mil-dot | BDC 1–10 · 1 MOA clicks | 20, self-loading · 1.1 MOA |
| M2010 ESR · Leupold Mk 4 ER/T | .300 Win Mag Mk 248 Mod 1 · 869 m/s | 12× (of 6.5–20×), 1.8°, Horus-style grid | 0.1-mil clicks from a 100 m zero · 0.1-mil clicks | 5 · 0.8 MOA |
| M107 · Leupold Mk 4 LR/T | .50 BMG M33 · 853 m/s | 14× (of 4.5–14×), 2.1°, mil-dot | ¼-MOA clicks from a 100 m zero · ¼-MOA clicks | 10, self-loading · 2 MOA |

**Known liberties:**

- Variable scopes are fixed at one power.
- The M107 turret dials out to 2,500 m. A real shooter would also need an angled mount and holdover.
- Bipod rifles sway less: the steadiness factor is 0.55–0.65 for the modern rifles and 0.9 for the SVD.

## 9. Training

### Tutorial (18 steps)

The tutorial runs on the mission map, with gold ground markers, a red tag over the current target, an SVG diagram for each step and quizzes with feedback.

1. Welcome
2. Move
3. Lower your profile
4. Use cover
5. Enemy eyes (open and close the map)
6. Choose a hide
7. Observe first
8. **Range with mils:** a magnified box beside the binocular view brackets the officer against the mil scale ("4.2 mils"), followed by a quiz.
9. Find the wind (the flag)
10. Read the wind (quiz)
11. **Full and half value:** a clock diagram splits a wind arrow into its crosswind part (pushes the bullet) and its head/tail part (ignored), for 3, 2, 1 and 12 o'clock. A table gives the value per clock hour, with worked examples (10 mph from 1 o'clock = 5 mph). A quiz asks for the wind to use from a random oblique clock hour and speed.
12. **Set the elevation drum:** a drum dial walks through ranged → nearest 100 m → drum N, with the presses needed. A trajectory diagram and a red cross in the magnified box show where the round lands with the post on the chest.
13. **Wind in mils:** a wind-to-mils table for the current range (mph → drift in cm → mils at full, 7/8 and half value) with a worked example, followed by a quiz.
14. **Dial the windage:** the direction (into the wind) and the number of 0.5-mil clicks. The cross slides onto the chest and turns green.
15. Breathe and fire
16. Correct and finish
17. Shoot and move
18. Training complete

The tutorial uses a fixed wind of 8 mph from 9 o'clock (full value) on the shot line. The officer stands still at the OP, and the hide's sightline is clear of trees and poles.

### Shooting range

- **Short lane (north):** silhouette boards at 100–600 m with distance signs, a moving target at 250 m sweeping ±18 m, and wind flags every 100 m.
- **Long lane (west along the ridge, past the map edge):** steel boards at 800, 1,000, 1,200, 1,500, 1,800, 2,000, 2,300 and 2,500 m. Boards are 1.5× size out to 1,200 m and 2× beyond. Flags stand every 250 m on a line 7° off the lane. Fog and the camera's draw distance widen on this lane.
- **Controls:** a rifle selector, lane tabs, wind presets (calm, steady, gusty), a toggle for the firing solution and a toggle for the range card.
- **Range card (T key or button):** a table that shows how to set the selected rifle for every target on the current lane. The top explains how this rifle's elevation works (a drum read in metres or yards, or a turret dialled up from the zero in MOA or mils, with keys and click size) and how its windage works (a turret in mil or MOA clicks, or hold-off for scopes without one). Each row gives the distance, the elevation setting and the clicks or presses to get there, any hold-over left, the wind drift for 1 mph of full-value wind, and the windage for the wind blowing now (its clock value applied, in clicks, with the direction to dial into the wind). The moving target row adds the lead. The card updates live as the wind changes and when the rifle or lane changes.
- **Scoring:** a head circle (r 13 cm), torso (50 × 70 cm) and centre-mass ring (r 12 cm). A hit table per target, a target face with the last 6 shots, and spotting discs on the boards.

## 10. Presentation and technology

- **One file:** Three.js r128 from cdnjs, with all assets procedural (canvas textures, SVG diagrams, WebAudio sound effects).
- **Rendering:** shadows that follow the focus point, fog, instanced grass tufts and bushes, and smoke emitters.
- **UI:** a paper and stencil wartime style. Slips show the HUD, and modals hold the briefing, field manual, hide card and after-action report.
- **Testing:** headless Playwright with SwiftShader. The CDN is routed to a local three.min.js. Headless rendering is slow, so ballistics tests step `updateBullets` directly.

## 11. Change process

1. Branch from `main`: `feature/<short-name>`.
2. **Update this GDD first** and commit it, then change the code and commit that.
3. Bump the build number in the briefing meta and in the GDD header.
4. Merge into `main` with `--no-ff`, tag `vMAJOR.MINOR`, and add a line to the version history below.
5. Update the Mac folder `Documents/AI/SNIPER`, which is the repository root and the only home of the sources. The game is `index.html` only: there are no versioned copies, and every past build is retrieved from its tag (`git show v1.6:index.html`). Republish the artifact. No other files (exports or git bundles) are kept in that folder.

## 12. Version history

| Tag | Date | Changes |
|---|---|---|
| v1.0 | 2026-10-25 | First release: stalk and shoot phases, detection, hide assessment, PU ballistics, wind, exposure and mortars, spotter, scoring and medals |
| v1.1 | 2026-10-25 | Graphics detail (textures, shadows, props, smoke) and jointed animated soldier rigs with realistic gaits |
| v1.2 | 2026-10-25 | Interactive tutorial mission with diagrams and markers, plus the shooting range |
| v1.3 | 2026-10-25 | Six selectable WW2 rifles with real scopes, reticles, adjustments and ballistics |
| v1.4 | 2026-10-27 | Modern rifles (SVD, M24, M110, M2010, M107) and the long-range lane from 800 to 2,500 m |
| v1.5 | 2026-10-28 | Clearer tutorial: magnified mil bracket, predicted-impact cross, drum dial, wind-to-mils table and a new "Wind in mils" step |
| v1.6 | 2026-10-28 | New tutorial step "Full and half value" with a crosswind diagram and examples. Wind values corrected to the clock method (2/4/8/10 o'clock = 7/8 value, not half) in the HUD, field manual, wind diagram and wind table |
| v1.7 | 2026-10-28 | Shooting range card: a toggleable (T) per-rifle table of elevation and windage settings for every target on the lane, with live wind |

## 13. Ideas backlog

- Multiple missions and maps (a counter-sniper duel, a night mission).
- Spin drift and Coriolis on the long lane.
- Variable-power scopes with a zoom control.
- Save the range scores between visits.
