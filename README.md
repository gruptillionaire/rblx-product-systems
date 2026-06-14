### Featured Projects
- Infinite Runner -- product analytics, live operations, deterministic validation
- Turn-Based RPG Battle Simulator -- battle simulation, utility AI, story/state systems
- Asymmetrical Horror Shooter -- procedural generation, validation, dynamic mesh rendering




# ROBLOX Product & Systems Work

I have spent several years building and operating Roblox-based live consumer products. These are commercial/live projects, so source code is private; this page focuses on product outcomes, technical systems, and design tradeoffs.
The projects below were built solo, with contractors used only where noted for specific art, UI, or map work.

## Scale / Product Context

- Reached 6M+ visits across shipped products
- 2,000+ peak concurrent users on _two_ solo products
- Owned product design, implementation, analytics, live operations, user feedback, economy / progression tuning and curves

## Project: Infinite Runner - Product Analytics & Live-Ops

<img width="1572" height="646" alt="image" src="https://github.com/user-attachments/assets/8f7fc459-526f-401a-a277-b8a38afea945" />
<img width="1562" height="637" alt="image" src="https://github.com/user-attachments/assets/141e6bd0-af2c-4a40-8a93-4e2c3f269dae" />
<img width="1566" height="643" alt="image" src="https://github.com/user-attachments/assets/0842208d-bb48-4a65-99d3-7ee6a65931a0" />


- Infinite runner where players spend energy to travel distance, earn currency, and upgrade speed, energy, and abilities.
- Reached 2,000+ concurrent users at peak
- Sustained a strong launch period, then saw a second activity spike after a Christmas update was released
<img width="1470" height="573" alt="image" src="https://github.com/user-attachments/assets/19c32c4c-8455-4962-82c6-01f629d187ba" />

Initial metrics showed weak retention and session length. I combined user feedback with progression modelling, then adjusted upgrade curves and reward pacing to make early progress feel more responsive.

After the changes, average session length increased from ~9 minutes to ~15 minutes, and D1 retention improved from ~5% to ~12%. The goal was not just to add content, but to tune the first-session economy so players reached meaningful upgrades sooner.

<img width="1482" height="575" alt="image" src="https://github.com/user-attachments/assets/45a83374-84c9-4159-967c-6526a77be2fb" />

<img width="608" height="222" alt="image" src="https://github.com/user-attachments/assets/1587424c-57e9-4061-a632-d8110daedbf9" />

(The analytics visual can be unpredictable, but this recent 14.29% user retention evaluation would put the game in the top 90th percentile)

### Features:
- Per-life procedurally generated infinite map
- Server-seeded client reconstruction for low-cost map generation.
- Seed-validated rare pickup spawning.
- Mobile-performant render distance and chunk loading.
- Persistent inventory, unlockable abilities, daily challenges, and permanent upgrades.

### Map
The map is generated per-life and per-player, so fully server-authoritative geometry would have been expensive and awkward -- the server would need to track a unique infinite route for every active player.
Instead, each life receives a server-authoritative seed. The client deterministically reconstructs its own route and pickups from that seed, while the server validates claimed rare pickups against the expected chunk state.
This keeps generation cheap and client-local while still preventing arbitrary pickup claims. The remaining exploit risk is minimal: a malicious client could infer pickup contents for a reached chunk, but still needs to
legitimately reach that chunk before any pickup is accepted.

### Summary
This project taught me the value of treating progression curves as product infrastructure. The biggest improvement did not come from adding a new feature, but from making the first-session economy respond better to player behaviour. The result was a measurable lift in session length and D1 retention.

## Project: Turn-Based RPG Battle Simulator

<img width="1566" height="639" alt="image" src="https://github.com/user-attachments/assets/82b57f2b-c266-4e29-8ecc-d7dc6b61714d" />

- A Pokémon-inspired turn-based battling story game
- Contracted map design and UI design
- Current story spans from first join up to the first major battle (unreleased)

My most technically deep game built around catching and battling a team of meme-inspired creatures, mostly revolving around the battle simulation mechanic & bespoke classical AI systems.

### Battle
A turn-based battling engine supporting an N field size format.
- Battlers have unique stats, types, moves, abilities, evolutions, all of which interact with each other due to a holistic design process.
- Levels, field effects, boss effects, statuses and stat changes keep each battle fresh and interesting to the user
- The battle state drives UI, move choice, AI decisions, win/loss flow, XP/cash rewards, catching, move learning and general game progression
- The battle engine is agnostic to the external game, supporting alternative modes: a Roguelike waves mode, unique bosses, trainers & wild battles

### NPC AI
A classical use of Utility AI provides an interesting experience that includes more than just the battle engine. Each potential outcome is scored via heuristic evaluation and some noise depending on difficulty.
- NPCs will attack the user supporting an N field size format based off of the difficulty parameter
- Higher difficulties allow for a high level play of AI predictions on future turns rather than what information is immediately available
- AI is utilised in other areas than battle; for instance upon a loss, the user is sent to a character who offers them shady deals. The character will offer deals and respond to the user with messages based off of what happened in battle, aiming to provide the most attractive deal and the most realistic message line, supporting an immersive experience.

### Area Rendering
As mobile and low-end device performance is a looming threat, the rendering and replication of the entire map would become an issue in development. To counteract this before it became an issue, I immediately implemented an area rendering system.
- Upon join, the user's last position is based off of their current area and their last known point (based off of a named checkpoint ID rather than a mathematical position, as editing the map would cause critical error on players with stale saved data).
- Entering a new area will unload the last, resetting some local states and loading the new one in from storage.
This also prevents unnecessary humanoid setup, physics states and more, as they will not be simulated while not in the direct workspace.

### Conversation / Story Flow
The original conversation flow quickly began to get out of hand. I rewrote the file to allow for a more data-driven conversation handler & action registry to allow for easier development. The conversation system was tricky, mostly because the many of the user's different fields had to be kept in-line and mutated in relation to the current story *stage*, not directly what was going on visually. For instance, if a player were to move to a new area during a story, not advance a story stage, then rejoin, they would be in the wrong area, and at worst a *restricted area*. Overcoming this issue was a larger task than I had initially expected, and may even rival the battle engine for complexity.
- Server validation still posed a large problem. Multiple different files of metadata was required to ensure that the local state was allowed, and each small change increases the amount of data storage by a few hundred bytes -- for instance, the quest system required modes that would or would not reset your quest on a loss, or upon a quest completion the user would be granted an item.
- Keeping in-line with the rejoining edge cases while also validating that the client has not 'spoofed' the success is a problem that can only be solved by staying vigilant.
- Branching dialogue / responses are supported.

### Local Rendering / World Presentation
To make the world feel alive I implemented a wide array of local-only visual systems:
- Scrolling billboards
- Trees shaking in the wind
- Destructibles
- Kickable rocks and traffic cones
I also implemented spatial UI, which involves a render level of physical geometry on top of the client's visual render level. This can be seen in the main menus and some inner menus. The process is some simple matrix multiplication around the camera's object space, as well as continual rotational / screen-based % movements based off of viewport size.

### FTUE / Retention-First Design
Upon response from testers, the user's FTUE _(First-Time User Experience)_ was far too slow to get invested into. This game style requires a certain amount of investment from a player, and for a platform with a generally younger audience this was tricky to get down. The importance of each playtime milestone, from 30s to 15 minutes, was thoroughly planned out and redesigned. Each 'win' at the start of the game was carefully planned, for instance:
- The first 60 seconds must include getting your first battler
- The first 2 minutes must include the first battle, with no chance for a loss
- The first 3 minutes must include catching a new battler
- The first 4 minutes must lightly introduce the plot of the story, and quietly introduce the main points of interest a user would interact with (healing stations, shop stations, et cetera)
The game being free-to-play means that a user needs to be snared from the beginning before churn. Telemetry was implemented (funnels & custom events) for release as to ensure this FTUE had its churn reduced as much as possible.

### Roguelike mode
As mentioned earlier. A battle simulation for early playtesters, featuring waves rather than the standard story experience. Each wave would provide the user with cash, levels and healing. An alternative mode beside the main game allows for a higher retention as the user can effectively play two different games in one, despite playing the same game. Real-world examples show _Pokémon Showdown!_ just as popular as mainline Pokémon games, even though there is great overlap.

### Additional Systems
Personalities affecting battle animations and send-out lines, bosses, 2v1 battles, dynamic battle field size, in-battle familiars, giants / wild events, rigged battler sprites rather than static decals, shops, storage, shinies, daily catches to improve retention... the list goes on.

### Screenshots

<img width="1559" height="632" alt="image" src="https://github.com/user-attachments/assets/cd4d9ce8-c4ce-4bed-9d64-135b714f2da8" />
<img width="1571" height="639" alt="image" src="https://github.com/user-attachments/assets/29150f2c-486c-4002-b2cd-c75ee390bdbf" />
<img width="1917" height="1024" alt="1" src="https://github.com/user-attachments/assets/8fa1973a-acbf-4133-837a-47e4431547b4" />
<img width="1917" height="1026" alt="6" src="https://github.com/user-attachments/assets/47b865f5-8170-4ce2-b837-c96730b0dee8" />
<img width="1916" height="1027" alt="8" src="https://github.com/user-attachments/assets/9eea6d9a-65f0-40c6-8709-0108fc2a0efb" />

## Project: Asymmetrical Horror Shooter

<img width="1572" height="597" alt="image" src="https://github.com/user-attachments/assets/fc115a3e-2712-495f-b483-1d235267ade1" />

- An SCP-genre shooter inspired by games such as SCP:SL and GMod: Breach
- Focuses on advanced client/server architectural splits, procedural generation, and visual feedback systems.
- My current/most recent project (in-dev)

This is an active prototype, so visuals and animation polish are still a work-in-progress.

### Life State Architecture
The round life state is mostly standard, with a few unique points.
- Round states: waiting, loading, active, post-round
- Post-round: The round is over and the game has been won. The game continues as normal, so the aftermath of the gameplay is visible.
- Per-life setup: Player role, inventory, physical character setup, UI, controls
- Death transition: Server-owned corpse marker + client-side ragdoll creation, death view, spectator pipeline (See: Gunplay)
- Distinct separation between round state and life state. Defeated players can respawn as different roles to supply the game with 'backup' forces.

### Custom Character Controllers
Each role is given its own controller based off of its metadata. Human roles are given the human controller, but monster roles are given unique monster controllers to provide the user with unique UI, abilities, and passive effects.
- All roles are given a per-user display, health, movement/jump speed, bespoke camera setup.
- Human roles are given an inventory
- Monster roles are given a shield
- Some human roles can escape, converting them to a higher authority of whatever alignment their faction is in

The following is an example of the movement of 'SCP-173', the custom monster controlled that can only move when not being looked at, or the user is blinking.
https://github.com/user-attachments/assets/c8638c51-fc43-4025-827b-4cbcc26bc699

### Dynamic Blood Mesh
The most technically impressive visual system, utilising the EditableMesh API (tight limitations due to low-end devices being necessary to keep in mind)
- The server creates a lightweight blood marker attachment saving the state of the gunshot
- Each time the blood is rendered, the client deterministically re-creates that blood spatter based off of its given seed as well as the geometric matrix of the attachment
- The client will raycast outwards from the origin point to find each projected sample point
- Via normal discontinuity detection if an edge is detected, connection points are defined at that edge
- Each point then becomes a vertex for a triangle via the EditableMesh API, converting the points into physical world space geometry
- Z-layer fighting is avoided via 32 possible layers of blood spatter (after which it would return back to the original, which would underlap, effectively giving 63 possible layers before actual overlap)
- To avoid hitting API limits, unfocused/irrelevant blood spatters are culled to free up current space
- Supports multiple blood type profiles

https://github.com/user-attachments/assets/0db751c9-21d1-4a1d-a271-5aa1e42c7144

### Door Systems
Doors are rendered completely locally due to the issue with animation lag which would create unfair visuals. Interaction is done via basic raycast + server validation. Clients construct door state via shared time-based attributes (AnimationStartTime, AnimationEndTime), wherein elapsed/total = % open.
- Feedback states designed in Figma for a clean visual on the keypad

### Gunplay / Local State
The server tracks state history to ensure gunplay is fair and smooth, validating client-claimed gunshots against it. Upon a valid shot the target will be hit.
- Gun state (magazine capacity, one-in-the-chamber, et cetera) is saved upon dropping and picking up the weapon
- Server gun authorisation is required to be based off of local state -- doors specifically posed an issue, as it may be in the 'open' state while the server geometry remains static. A custom raycast handler allows *all* raycasts (including SCP-173's view raycasts) to respect doors based off of % open as well as door panel size. If a raycast hits a door part, and that part should be open, another raycast with the remaining distance is immediately fired in the same direction. This allows the server to respect client door states.
- Client-side viewmodel is constructed separately to the third-person viewmodel. Animations are reused via locally simulated inverse kinematic control, which allows for procedural limb animation towards any item's hold points
- The client's head pitch and yaw is replicated with server-validation, allowing other players to see specifically where you are looking. A well-selected head mesh allows the neck to seamlessly interpolate between these points, providing a smooth transition.
- Hitmarkers, dynamic crosshair, friendly fire custom server flags, fall damage & more is all included

<img width="924" height="547" alt="image" src="https://github.com/user-attachments/assets/cfcd8621-eff6-4341-ac24-c0e74d6602f0" />

### Map Generation
A prefab-based, socket-driven procedural generation with constraint-based room placement and graph connectivity validation on a discrete grid. Each area chooses its generation type to create an interesting facility layout.
- Main zone generation is hierarchical: some zones have dependencies that must be satisfied before generation.
- Two current styles of generation: 'Labyrinth' and 'Hub'

**Labyrinth**:
- Creates an abstract graph on a rectangular grid
- Carves nodes & edges via randomised 'branch' growth
- Trims 'leaves' and counts loops & validates shape
- Assigns *required* terminals and through rooms to the graph nodes, then converts node degrees into authored room types:
- Degree 4 - Four Way Intersection
- Degree 3 - T-Intersection
- Degree 2 - Straight Corridor
- Degree 2 Corner - Corner
- Degree 1 - Dead End / Terminal Room
Using graph algorithms:
- Breadth First Search (BFS) / Flood Fill style traversal for connected components, branch size, empty regions, reachability and distance
- Loop / cycle counting via graph edge / node counts + component checks
- Validates to avoid a rectangular fill, boring corridors, 'fake' junctions, unused junctions and more.

**Hub**:
More procedural/skeleton-driven than the Labyrinth system.
- Place an initial 4-way intersection
- Attach a connection to the previous zone (each zone must filter in to each other, so the Labyrinth zone(s) has connection points to the Hub zone)
- Builds a main spine of hubs into straights into hubs, et cetera.
- Adds side-hubs and extra hubs
- Places required through-rooms
- Places required terminal points, or dead ends if no terminal points remain (terminal points are unique areas)
Then runs post-layout repairs:
- Collapse useless/filler 4-way intersections (too many dead ends or no feasible 'through')
- Repairs pairs of filler caps into better connecting paths, so two dead ends near each other can instead become a path to each other

This is all done in data by tables and numbers. After the map is confirmed it will be physically made in the workspace with geometry.
Upon critical error of a map generation, the failed zone will retry. If a zone retries >64 times, the entire map generation will retry. This retry amount should be kept as low as possible.

### Extras

### Power Systems
The facility power, dictated by the 'Guards' spawn role, affects the entire facility via transit, security, or network. The asymmetric consequences of a player's actions leads to a more interesting and unique gameplay. This makes the map feel systemic and 'alive' rather than one static hallway network.

### Ragdolls, Monster Models/Bones
As stated before, client ragdolls are made via server markers, cleaned up on round end. This allows for almost unlimited persistent corpses to be rendered very cheaply. This differs from more complex systems wherein a ragdolled player may need to get up, as the ragdoll will not need to be interacted with again. Network cost is negligible.
Monsters use meshes and bones (made in Blender) for animations, allowing for unique spring-related control systems to control their bodies. Utilised currently is:
- a droplet system
- a gross monster stomach system
both of which interact with the velocity of its ancestors on a spring-based PID control / transfer functions.

### Audio Systems
Wiring up the AudioPlayer/AudioEmitter pipeline allows users to project their voice at multiple places at once, useful for intercom / proximity voice chat / radio systems, as well as hear other people, useful for spectator chats and more. Audio is thus forced to be rendered by the client.

### Screenshots

<img width="665" height="536" alt="image" src="https://github.com/user-attachments/assets/7c35eb1d-6a1a-419e-aec9-f068d2c8a029" />
<img width="679" height="266" alt="image" src="https://github.com/user-attachments/assets/5d33994e-b84c-4dcf-905e-8cf72c83d996" />
<img width="1558" height="636" alt="image" src="https://github.com/user-attachments/assets/1c1c52f7-ac03-4e8e-8722-375c6fb2e73e" />
