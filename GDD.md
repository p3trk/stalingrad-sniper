# Stalingrad Sniper: Game Design Document

| | |
|---|---|
| Current build | 1.8 |
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
| Campaign | 18 missions based on real WW2 sniper stories, 1939–1945 | See [section 14](#14-campaign-18-missions). Each mission issues the historical rifle and has its own briefing, history, map and sources |
| Mission | The original Red October mission (campaign mission 7) | Choose a difficulty and any rifle on the briefing sheet |
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
5. Update the Mac folder `Documents/AI/SNIPER`, which is the repository root and the only home of the sources. The game is `index.html` only: there are no versioned copies, and every past build is retrieved from its tag (`git show v1.6:index.html`). Republish the artifact. The only other files kept there are the briefing maps in `maps/` with their credits (`maps/SOURCES.md`). No exports or git bundles are kept in that folder.

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
| v1.8 | 2026-10-28 | Campaign of 18 missions from real WW2 sniper stories (1939–1945), each with its own terrain, season, sun, enemy, objective, historical rifle, history, sources and briefing map. New Finnish M/28-30 with iron sights, decoy (G) and sniper duels |

## 13. Ideas backlog

- A night mission.
- Hand-built maps per mission (the campaign reuses one battlefield layout with per-mission terrain, season, buildings and enemies).
- Spin drift and Coriolis on the long lane.
- Variable-power scopes with a zoom control.
- Save the range scores between visits.

## 14. Campaign: 18 missions

The campaign follows real WW2 snipers from the Winter War in December 1939 to Manchuria in August 1945. Every mission is built on a documented person, unit, place and date. Where the record is thin or disputed (the Zaitsev–König duel, Allerberger's memoir, kill tallies), the briefing says so. The player is never told they *are* the historical sniper: they serve in that sniper's unit, on that day, doing the job the record describes.

### How the campaign works

- **Campaign sheet.** The title screen opens the campaign: 18 mission cards in date order, each with its number, title, place, date, army, a tick when won and the best score. All missions can be played in any order; the sheet suggests the next unfinished one.
- **Mission briefing.** Each briefing has: the order (situation and objective), the date, time, place and unit, the issued rifle, "What really happened" (two to four sentences of history, with any doubts stated), a briefing map (a period map or aerial photo of the real place, when one with a free licence exists, with its caption, author and licence), and the sources.
- **One battlefield, many places.** The battlefield keeps its proven layout (our trench in the south, a ridge, no-man's-land, the enemy trench, wire, an observation post, an HQ and a tall building for the enemy sniper). Each mission changes everything a player sees and plays against: season and ground colours, snow, sky and fog, sun position (computed for the real date, time and place, then turned so the enemy is at the top of the map), wind, the terrain type (steppe, forest, urban ruins, bocage, polder, mountains, jungle), the buildings and their names, the enemy army and uniforms, the objective, the issued rifle, the spotter's name and the medal table. The mission is chosen with `?m=N` in the page address, so loading a mission rebuilds the whole world.
- **Rifle.** Each mission issues the historical rifle (or the closest one in the game, stated in the briefing). The original Red October mission keeps the free rifle choice.
- **Progress** (wins, best score, best medal) is kept in the browser's local storage. Losing it only clears the ticks.
- **Medals** follow the player's army: Soviet (Medal "For Courage", Order of Glory 3rd class, Order of the Red Star, Order of the Red Banner, Hero of the Soviet Union), Finnish (Medal of Liberty 2nd and 1st class, Cross of Liberty 4th and 3rd class), German (Iron Cross 2nd class, Sniper's Badge in bronze, silver and gold, Iron Cross 1st class), US (Bronze Star, Silver Star, Distinguished Service Cross) and Canadian (Mentioned in Dispatches, Military Medal, Distinguished Conduct Medal).

### Objective types

| Type | Win condition | Used in |
|---|---|---|
| Officer | Kill the named officer (the observation-post routine from v1.0), then withdraw to our trench | 1, 7, 12, 14, 15 |
| Count | Kill a set number of named targets (observers, water carriers, gun crew, tree snipers, MG crew), then withdraw | 2, 3, 5, 6, 9, 10, 11, 13, 16, 17, 18 |
| Duel | Kill the enemy sniper, who knows you are out there, then withdraw | 4, 8 |

### New mechanics

- **Decoy (G).** In the hide, the spotter raises a helmet on a stick a few metres to the side. An enemy sniper with a line of sight fires at it after a few seconds, showing a muzzle flash and a puff of dust at his position. Two uses per mission. Real basis: Zaitsev's partner Kulikov raised a helmet to draw "König's" shot, and Nomokonov worked helmets on strings to make the enemy fire. Available in missions 4, 8 and 18, and in any mission with an enemy sniper.
- **Duel sniper.** In a duel the enemy sniper changes position between a few hides chosen for the mission (a ruin, a wreck, a building window). He glints more often when the sun shines into his lens, and he fires at you if your exposure is high and he can see you.
- **Iron sights.** The Finnish M/28-30 has no scope: view 3 shows the rear notch and front blade at 1× with a narrow field, there is no windage turret (hold off), and there is no lens glint.
- **Snow.** Snow missions whiten the ground, grass and bushes. A player in white camouflage (Finnish) gets the bush value on snow-covered open ground while prone.
- **Count targets** move on their own routines: water carriers run side-on to a well at dawn, observers sweep with binoculars, tree snipers sit high in palm trees and glint.

### The missions

| # | Title | Place and date | Player | Objective | Rifle |
|---|---|---|---|---|---|
| 1 | White Death | Kollaa front, Ladoga Karelia, Finland · 21 Dec 1939 | Finnish JR 34, 6th Company | Officer | M/28-30, iron sights |
| 2 | First Blood at Biliaivka | Biliaivka, west of Odessa · 8 Aug 1941 | Soviet 25th "Chapaev" Rifle Division | Count: 2 observers | Mosin PE |
| 3 | Neva Bank | Nevskaya Dubrovka, Leningrad · 8 Sep 1941 | Soviet 11th Separate Rifle Brigade | Count: 2 advancing riflemen | Mosin PE |
| 4 | Mackenzie Heights | Sevastopol · Jan 1942 | Soviet 25th Rifle Division | Duel | SVT-40 · PU |
| 5 | Mamayev Kurgan | Stalingrad · 25 Oct 1942 | Soviet 1047th Rifle Regiment, 284th Rifle Division | Count: artillery observer and signaller | Mosin PE |
| 6 | Water at Dawn | Ninth January Square, Stalingrad · Oct 1942 | Soviet 13th Guards Rifle Division | Count: 3 water carriers | Mosin PU |
| 7 | Red October | Red October factory, Stalingrad · 12 Nov 1942 | Soviet 62nd Army | Officer | Free choice |
| 8 | The Duel | Metiz works, Stalingrad · Nov 1942 | Soviet 284th Rifle Division | Duel ("Major König") | Mosin PE |
| 9 | Hill 27 | Guadalcanal · 13 Jan 1943 | US 132nd Infantry, Americal Division | Count: 3 tree snipers | M1903A4 |
| 10 | Velizh Sector | Kalinin Front · Mar 1943 | Soviet 1122nd Rifle Regiment, 334th Rifle Division | Count: 3 of the gun crew and drivers | Mosin PU |
| 11 | Captured Rifle | Mius front, Donbas · Jul 1943 | German 144th Gebirgsjäger Regiment, 3rd Mountain Division | Count: 2 Soviet machine-gunners | Mosin PU (captured) |
| 12 | Hedgerows | Hill 192, St-Lô, Normandy · 10 Jul 1944 | US 38th Infantry, 2nd Infantry Division | Officer (artillery observer) | M1903A4 |
| 13 | Kapellen | Kapellen, north of Antwerp · 6 Oct 1944 | Canadian Calgary Highlanders, scout and sniper platoon | Count: MG gunner and loader | No.4 Mk I (T) |
| 14 | Border of East Prussia | East Prussian border · Oct 1944 | German 196th Grenadier Regiment, 68th Infantry Division | Officer | Kar98k · ZF39 |
| 15 | Across the Valley | Eastern Carpathians · Oct 1944 | German 144th Gebirgsjäger Regiment, 3rd Mountain Division | Officer at long range | Kar98k · ZF39 |
| 16 | Last Winter | Near Ilmsdorf, East Prussia · 26 Jan 1945 | Soviet 184th Rifle Division | Count: 3 (MG crew and an officer) | Mosin PU |
| 17 | Moltke Bridge | Berlin · 29 Apr 1945 | Soviet 150th Rifle Division, 3rd Shock Army | Count: 3 defenders covering the bridge | Mosin PU |
| 18 | Taiga Shaman | Foothills of the Greater Khingan, Manchuria · 12 Aug 1945 | Soviet Transbaikal Front | Count: 2 Japanese officers | Mosin PU |

### Mission details

Sun positions were computed for the real place, date and local time, then turned so the enemy is at the top of the map ("game azimuth"). Spotter names are invented unless stated.

**1. White Death.** Kollaa, 21 December 1939, 11:30, −30 °C. Snow forest, a frozen clearing, sun barely 4° above the southern horizon (game azimuth 85°). Enemy: Red Army, 56th Rifle Division. Objective: the Soviet company officer at a forward observation post at the forest edge.
- *History:* Simo Häyhä, a corporal in the 6th Company of Infantry Regiment 34 under Lt. Aarne Juutilainen, held the Kollaa line through the Winter War. He used a Civil Guard M/28-30 with iron sights, which kept his head low, could not fog in the cold and did not glint. He packed snow in front of his position and kept snow in his mouth so his breath would not show. His best day was 21 December 1939 with 25 kills. Counts range from 219 rifle kills (division commander, February 1940) and 259 (chaplain Rantamaa's diary) to about 500 in his own notes; historian Risto Marjomaa credits "more than two hundred". He was badly wounded on 6 March 1940.
- *Game:* iron sights, white camouflage, no lens glint, enemy counter-sniper in the trees after the first shot. Spotter: Cpl. Korpela.
- *Sources:* [Simo Häyhä (Wikipedia)](https://en.wikipedia.org/wiki/Simo_H%C3%A4yh%C3%A4); Tapio Saarelainen, *The White Sniper* (Casemate, 2016).

**2. First Blood at Biliaivka.** 8 August 1941, 07:00. Summer steppe and vineyards, dry grass, morning sun behind you (game azimuth 168°). Enemy: Romanian 4th Army. Objective: two observers on a hill about 400 m away.
- *History:* Lyudmila Pavlichenko volunteered in Odessa and joined the 25th Rifle Division as a sniper. She dug trenches with only a grenade until a wounded comrade gave her his rifle. Her first two kills were at Biliaivka on 8 August 1941 at about 400 m. In two and a half months at Odessa she was credited with 187 kills; her final tally was 309, including 36 enemy snipers.
- *Game:* Mosin with the pre-war PE scope. Spotter: Pvt. Bondarenko.
- *Sources:* [Lyudmila Pavlichenko (Wikipedia)](https://en.wikipedia.org/wiki/Lyudmila_Pavlichenko); [National WWII Museum: "Lady Death" of the Red Army](https://www.nationalww2museum.org/war/articles/lady-death-red-army-lyudmila-pavlichenko); Pavlichenko, *Lady Death* (Greenhill, 2018).

**3. Neva Bank.** Nevskaya Dubrovka, 8 September 1941, 08:00. Late summer birch and pine edge, river meadow, sun to your right (game azimuth 211°). Enemy: German infantry. Objective: the two riflemen of a patrol advancing toward the river.
- *History:* Vladimir Pchelintsev, a reconnaissance platoon commander in the 11th Separate Rifle Brigade, opened his tally on 8 September 1941 near Nevskaya Dubrovka on the Neva by shooting two advancing German soldiers. He had 102 kills by 6 February 1942, when he was made Hero of the Soviet Union, and he trained snipers before touring the United States with Pavlichenko in 1942. Reliable sources put his total at about 150; his memoir's 456 is considered overstated.
- *Game:* targets are the patrol pair, who move. Spotter: Pvt. Gusev.
- *Sources:* [Vladimir Pchelintsev (Wikipedia)](https://en.wikipedia.org/wiki/Vladimir_Pchelintsev).

**4. Mackenzie Heights.** Sevastopol, January 1942, 09:30. Wooded limestone hills, oak scrub, bare winter trees, light frost (game azimuth 102°). Enemy: German 11th Army. Objective: duel with a German sniper who has been killing our runners.
- *History:* Pavlichenko's division was shipped from Odessa to Sevastopol in October 1941. During the siege she trained snipers and was cited for 257 kills by May 1942. Thirty-six of her 309 kills were enemy snipers. Her sniper partner and husband, Jr. Lt. Alexei Kitsenko, was mortally wounded by a mortar shell in early 1942. Photographs from Sevastopol show her with a scoped SVT-40.
- *Game:* duel, decoy (G). Spotter: Jr. Lt. Kitsenko.
- *Sources:* [Lyudmila Pavlichenko (Wikipedia)](https://en.wikipedia.org/wiki/Lyudmila_Pavlichenko); [National Park Service: "Lady Death" and the First Lady](https://www.nps.gov/elro/blogs/lady-death-and-the-first-lady.htm).

**5. Mamayev Kurgan.** Stalingrad, 25 October 1942, 08:00. Shell-torn hill slopes, water towers, ruins of the Metiz works (game azimuth 214°). Enemy: German 6th Army. Objective: the artillery observer and his signaller.
- *History:* Vasily Zaitsev, a former Pacific Fleet clerk, served in the 1047th Rifle Regiment of the 284th Rifle Division, which crossed into Stalingrad in September 1942. Soviet sources credit him with 225 kills between October and 17 December 1942. With Nikolai Kulikov he developed the "sixes": three pairs of sniper and observer covering a sector. He was wounded in the eyes by a mortar round in January 1943.
- *Game:* urban hill, many ruins. Spotter: Nikolai Kulikov (historical partner).
- *Sources:* [Vasily Zaitsev (Wikipedia)](https://en.wikipedia.org/wiki/Vasily_Zaitsev_(sniper)); Vasily Grossman, *A Writer at War* (2005), Stalingrad chapters.

**6. Water at Dawn.** Ninth January Square, Stalingrad, October 1942, 06:40. Blocks of burnt-out flats, Pavlov's House, a well in the square, dawn at your back (game azimuth 194°, elevation 3.5°). Enemy: German infantry. Objective: three men running for water.
- *History:* Anatoly Chekhov, 19, of the 13th Guards Rifle Division, often fired from the roof of Pavlov's House. Vasily Grossman went out with him and wrote that he shot Germans going out at dawn for water, often running side-on, and never fired with the sun on him so his shadow would not show. He knocked down nine on his first day, 17 in two days and reported 40 in eight days.
- *Game:* fast side-on targets (lead them). Spotter: Pvt. Ivanov.
- *Sources:* Grossman, *A Writer at War* ([chapter on Stalingrad snipers](https://erenow.org/ww/awriteratwar/18.php)); [Pavlov's House defenders (stalingrad.net)](https://stalingrad.net/russian-hq/pavlovs-house/defenders.htm).

**7. Red October.** The original mission (sections 2 to 7), now placed in its campaign context: the German attack on the Red October works (Operation Hubertus, 11 November 1942) left both sides dug in among the factory ruins.

**8. The Duel.** Stalingrad, November 1942, 14:00. No-man's-land between a wrecked tank and a pillbox, with a sheet of iron and a pile of bricks. The afternoon sun shines into the enemy's lens (game azimuth 165°). Enemy: a German sniper instructor. Objective: duel.
- *History:* In Zaitsev's memoir, Moscow sent the head of a Berlin sniper school, "Major König", to kill him. After days of searching, Zaitsev and Kulikov found him under a sheet of iron. Kulikov raised his helmet, the German fired, and Zaitsev shot him when he looked up. No German record of any König or "Heinz Thorvald" exists, and historians treat the duel as a propaganda story built on a smaller real event. The briefing says this.
- *Game:* duel with decoy (G). The sniper glints when the sun is on his lens. Spotter: Nikolai Kulikov.
- *Sources:* [Erwin König (Wikipedia)](https://en.wikipedia.org/wiki/Erwin_K%C3%B6nig); [National Interest: "The Greatest Sniper Duel in History Is a Myth"](https://nationalinterest.org/node/33161); Zaitsev, *Notes of a Russian Sniper* (2003).

**9. Hill 27.** Guadalcanal, 13 January 1943, 08:00. Jungle ridge with kunai grass and tall palms, humid haze (game azimuth 230°). Enemy: Imperial Japanese Army. Objective: three snipers tied into palm trees.
- *History:* John B. George, a rifleman with the 132nd Infantry of the Americal Division, landed on Guadalcanal on 8 December 1942. His battalion flanked the Gifu strongpoint near Mount Austen and took Hill 27 in January 1943. He used a scoped rifle with a spotter on field binoculars, and wrote it all down in *Shots Fired in Anger* (1947). He actually carried a privately bought scoped Winchester; the game issues the M1903A4.
- *Game:* targets high in palms (aim above the treeline). Spotter: PFC Mazur.
- *Sources:* [Pacific Wrecks: review of *Shots Fired in Anger*](https://pacificwrecks.com/reviews/shots-fired-in-anger.html); John B. George, *Shots Fired in Anger* (1947, NRA reprint 1981); John Miller Jr., [*Guadalcanal: The First Offensive*](https://history.army.mil/html/books/005/5-3/index.html) (US Army CMH, 1949).

**10. Velizh Sector.** Kalinin Front, March 1943, 09:00. Snow-covered fields and birch woods with a German gun position (game azimuth 227°). Enemy: German infantry and gunners. Objective: three of the gun crew and tractor drivers.
- *History:* Ivan Sidorenko taught himself sniping as a mortar officer in the Battle of Moscow and became assistant chief of staff of the 1122nd Rifle Regiment, 334th Rifle Division. His Red Star citation of 31 December 1942 credited 129 kills; by 1 May 1943 he had 349. On one sortie he and a trainee destroyed a tank and three tractors with incendiary rounds. He ended the war with about 500 kills and over 250 trained snipers.
- *Game:* snow, gun position with tractors. Spotter: a trainee, Pvt. Sergeyev.
- *Sources:* [Ivan Sidorenko (Wikipedia)](https://en.wikipedia.org/wiki/Ivan_Sidorenko).

**11. Captured Rifle.** Mius front, Donbas, July 1943, 07:00. Sunflower fields, balkas (dry ravines), heat haze (game azimuth 351° after turning, so the sun is ahead: glint risk). Player: German Gebirgsjäger. Enemy: Red Army. Objective: two Soviet machine-gunners.
- *History:* Josef "Sepp" Allerberger came to the Eastern Front with the 144th Gebirgsjäger Regiment in 1943 as a machine-gunner. While recovering from a light wound he tried a captured Mosin 91/30 with a PU scope and made his first sniper kills with it, before formal sniper training. He was credited with 257 kills. His memoir was written by Albrecht Wacker from interviews, and some of its claims (such as a Knight's Cross) are not documented; the briefing says so.
- *Game:* captured Mosin PU, player side German. Spotter: Gefr. Huber.
- *Sources:* [Josef Allerberger (Military Wiki)](https://military-history.fandom.com/wiki/Josef_Allerberger); Albrecht Wacker, *Sniper on the Eastern Front* (Pen & Sword, 2005).

**12. Hedgerows.** Hill 192, east of St-Lô, 10 July 1944, 07:30. Bocage: small fields boxed in by earth banks and hedges, sunken lanes, an orchard, a farm and a church tower (game azimuth 248°). Enemy: German 3rd Parachute Division. Objective: the artillery observer on the hill.
- *History:* Hill 192 gave the Germans a view over all the approaches to St-Lô. The 3rd Parachute Division held it with strongpoints in the hedgerows, which were 1–1.5 m thick and 2–3 m high. The US 2nd Infantry Division took it on 11 July 1944 after a month of stalemate. The player is a sniper of the 38th Infantry the day before the attack; no single named sniper is portrayed.
- *Game:* hedgerow lines give cover and block sightlines. Spotter: Pvt. Novak.
- *Sources:* [*St-Lô (7–19 July 1944)*, American Forces in Action (US Army, 1946)](https://ibiblio.org/hyperwar/USA/USA-A-StLo/USA-A-StLo-3.html).

**13. Kapellen.** North of Antwerp, 6 October 1944, 09:00. Flat polder with drainage dykes, poplar rows, brick farms, wet grey sky (game azimuth 111°). Enemy: German infantry. Objective: the MG gunner and loader.
- *History:* Sgt. Harold Marshall was the sniper of the Calgary Highlanders' scout and sniper platoon. Army photographer Ken Bell photographed him near Fort Brasschaat in September 1944 and in Kapellen on 6 October 1944 with his partner, Cpl. Steven Kormendy, who showed how to use foliage as cover. Marshall's No. 4 Mk I (T) had a scope and a wooden cheek piece. He fought through Normandy and the Scheldt and was wounded on 15 December 1944.
- *Game:* No.4(T), flat open ground, dykes for cover. Spotter: Cpl. Kormendy (historical partner).
- *Sources:* [Harold Marshall (Wikipedia)](https://en.wikipedia.org/wiki/Harold_Marshall_(sniper)); [Calgary Highlanders: Scouts & Snipers Platoon](https://calgaryhighlanders.com/about-the-regiment/detailed-history/calgary-highlanders-1939-45/scouts-snipers/).

**14. Border of East Prussia.** October 1944, 09:00. Autumn farmland with a village, beet fields and a wood (game azimuth 41°). Player: German infantry. Enemy: Red Army. Objective: a Soviet battalion officer.
- *History:* Bruno Sutkus, from a Königsberg suburb, trained at the sniper school in Vilnius in 1943 and served with the 196th Grenadier Regiment, 68th Infantry Division. He was credited with 209 kills, each confirmed by an observer and the battalion commander in his sniper's book. He received the Sniper's Badge in gold on 21 November 1944. After the war he spent over 20 years in Siberia.
- *Game:* Kar98k with ZF39. Spotter: Gefr. Lange.
- *Sources:* [Bruno Sutkus (Wikipedia)](https://en.wikipedia.org/wiki/Bruno_Sutkus); Sutkus, *Im Fadenkreuz* (Munin, 2003).

**15. Across the Valley.** Eastern Carpathians, October 1944, 10:00. Mountain valley with spruce forest, rock and early snow on the tops; the officer stands across the valley at 700–1,000 m (game azimuth 53°). Player: German Gebirgsjäger. Enemy: Red Army. Objective: the officer.
- *History:* Matthäus Hetzenauer of the 144th Gebirgsjäger Regiment, 3rd Mountain Division, trained at the Seetaler Alpe sniper school in 1944 and fought in the Carpathians, Hungary and Slovakia with a Kar98k with a 6× scope and a G43 with a ZF4. He was credited with 345 kills, the most of any German sniper, and his longest reported kill was at about 1,100 m. He was wounded on 6 November 1944 and awarded the Knight's Cross in April 1945.
- *Game:* the longest shot of the campaign with the ZF39's 800 m drum (hold over beyond it). Spotter: Gefr. Steiner.
- *Sources:* [Matthäus Hetzenauer (Wikipedia)](https://en.wikipedia.org/wiki/Matth%C3%A4us_Hetzenauer).

**16. Last Winter.** Near Ilmsdorf, East Prussia, 26 January 1945, 10:00. Deep snow, estate buildings, a frozen pond, our self-propelled guns firing often (game azimuth 230°). Enemy: German infantry. Objective: an MG crew and the officer directing them.
- *History:* Roza Shanina joined the 184th Rifle Division in April 1944 and made her first kill near Vitebsk three days later. She fought at Vilnius and was among the first Soviet troops into East Prussia. She kept a diary from October 1944, was wounded by a sniper in December, and talked her way back to the front in January 1945. She was mortally wounded by a shell fragment on 27 January 1945 near the Reichau estate and died the next day, aged 20, with 59 confirmed kills.
- *Game:* frequent artillery masking during the assault. Spotter: Sgt. Kalugina (named in tribute; Klavdiya Kalugina served in the same division's sniper platoon).
- *Sources:* [Roza Shanina (Wikipedia)](https://en.wikipedia.org/wiki/Roza_Shanina).

**17. Moltke Bridge.** Berlin, 29 April 1945, 07:00. Ruined city blocks on the Spree, the half-wrecked bridge, the Ministry of the Interior beyond (game azimuth 270°). Enemy: SS and Volkssturm. Objective: three defenders covering the bridge.
- *History:* On 28 April 1945 the 3rd Shock Army reached the Moltke Bridge, about 600 m from the Reichstag. The demolition charges only partly destroyed it, and by midnight the 150th and 171st Rifle Divisions held a bridgehead. The player is a sniper covering the assault; no single named sniper is portrayed.
- *Game:* dense urban cover, short ranges, many targets. Spotter: Sgt. Zagitov (named in tribute to Gazi Zagitov of the 150th Rifle Division, a Reichstag flag-raiser).
- *Sources:* [Moltke Bridge (Wikipedia)](https://en.wikipedia.org/wiki/Moltke_Bridge); Antony Beevor, *Berlin: The Downfall 1945* (2002).

**18. Taiga Shaman.** Foothills of the Greater Khingan, Manchuria, 12 August 1945, 08:00. Grassy hills, larch woods, summer haze (game azimuth 337°: the morning sun is ahead of you, so watch the glint). Enemy: Japanese Kwantung Army. Objective: two officers.
- *History:* Semyon Nomokonov, an Evenk hunter from Transbaikalia, served from 1941 to 1945 on five fronts. He marked kills on his pipe, moved in horsehair boots, used mirror fragments and helmets on strings as decoys, and was credited with 367 kills. In August 1945, on the Transbaikal Front in the Khingan foothills, he killed 8 soldiers and officers of the Kwantung Army (some records say 15) and was given a personal rifle, binoculars and a horse.
- *Game:* decoy (G). Spotter: Pvt. Tsyrenov.
- *Sources:* [Semyon Nomokonov (Wikipedia)](https://en.wikipedia.org/wiki/Semyon_Nomokonov); [Номоконов, Семён Данилович (Russian Wikipedia)](https://ru.wikipedia.org/wiki/%D0%9D%D0%BE%D0%BC%D0%BE%D0%BA%D0%BE%D0%BD%D0%BE%D0%B2,_%D0%A1%D0%B5%D0%BC%D1%91%D0%BD_%D0%94%D0%B0%D0%BD%D0%B8%D0%BB%D0%BE%D0%B2%D0%B8%D1%87).

### New rifle: Finnish M/28-30

| Rifle · sight | Round · MV | Sight | Elevation · windage | Magazine · group |
|---|---|---|---|---|
| M/28-30 "Pystykorva" · iron sights | 7.62×53R D166 heavy ball, 13 g · 730 m/s | Open notch and blade, 1×, 14° view | Rear sight 150–2,000 m (the game uses 100 m steps from 100 m) · none, hold off | 5 · 2 MOA |

## 15. Briefing maps

Maps live in a `maps/` folder next to `index.html` (`maps/m01.jpg` … `maps/m18.jpg`, at most 1,600 px on the long side) with their credits in `maps/SOURCES.md`. Only public-domain or freely licensed images are used (public domain, CC0, CC BY, CC BY-SA), each with its author, licence and source page. A briefing without a map file simply leaves the map out. Because Wikimedia cannot be reached from the cloud build machine, the maps are downloaded on the Mac.

| # | Map wanted |
|---|---|
| 1 | Battle of Kollaa or the Ladoga Karelia front, 1939–40 |
| 2 | Siege of Odessa, 1941 |
| 3 | Nevsky Pyatachok / Neva crossings, autumn 1941 |
| 4 | Siege of Sevastopol, 1941–42 |
| 5, 6, 7, 8 | Stalingrad city fighting, Oct–Nov 1942 (map or Luftwaffe aerial photo: Mamayev Kurgan, central city, Red October, Metiz) |
| 9 | Guadalcanal, Mount Austen, the Gifu and Hill 27, Jan 1943 (US Army CMH) |
| 10 | Kalinin Front / Velizh, 1942–43 |
| 11 | Mius front, July 1943 |
| 12 | Hill 192, July 1944 (US Army CMH *St-Lô*) |
| 13 | Battle of the Scheldt, Oct 1944 |
| 14 | Gumbinnen–Goldap operation, Oct 1944 |
| 15 | East Carpathian operation, Sep–Oct 1944 |
| 16 | East Prussian offensive, Jan 1945 |
| 17 | Battle of Berlin, Reichstag and Moltke Bridge, Apr 1945 |
| 18 | Soviet invasion of Manchuria, Aug 1945 |
