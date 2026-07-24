# Live Product & Systems Engineering

Selected engineering work from live interactive products built and operated primarily as a solo developer.

## At a glance

- **6M+ visits** across shipped products
- **2,000+ peak concurrent users** on two solo-built products
- End-to-end ownership across architecture, implementation, analytics, performance, live operations, and product iteration
- Experience with deterministic simulation, client/server validation, procedural generation, runtime rendering, persistence, and classical AI

These are commercial projects, so the complete source remains private. Selected code, analytics, algorithms, and implementation walkthroughs can be shared during a hiring process.

| Project | Primary evidence |
| --- | --- |
| [Infinite Runner](#infinite-runner) | 2,000+ concurrent users, D1 retention from ~5% to ~12%, deterministic generation |
| [Turn-Based RPG](#turn-based-rpg) | Variable-size battle simulation, utility AI, rejoin-safe story state |
| [Asymmetrical Horror Shooter](#asymmetrical-horror-shooter) | Historical validation, runtime mesh projection, graph-based map generation |

## Infinite Runner

- **Focus:** product analytics, live operations, deterministic client/server architecture
- **Scale:** 2,000+ peak concurrent users
- **Ownership:** solo product design, engineering, analytics, economy tuning, and updates

<img width="1562" height="637" alt="Infinite runner gameplay" src="https://github.com/user-attachments/assets/141e6bd0-af2c-4a40-8a93-4e2c3f269dae" />

An infinite runner in which each attempt generates a new route. Players convert distance into currency, then invest in speed, energy, abilities, and permanent progression.

### Measurable product iteration

Initial analytics showed weak first-session retention and short sessions. I combined player feedback with progression modelling, then revised reward pacing and upgrade curves so useful progress arrived earlier.

- Average session length increased from **approximately 9 minutes to 15 minutes**
- D1 retention increased from **approximately 5% to 12%**
- A later seasonal update produced a second activity spike after the initial launch period

The highest-impact change was an economy revision rather than another content update. The result shaped how I approach live products: identify the weak stage in the user journey, model the relevant system, release a focused change, then measure the outcome.

<img width="1482" height="575" alt="Infinite runner session analytics" src="https://github.com/user-attachments/assets/45a83374-84c9-4159-967c-6526a77be2fb" />

### Deterministic world generation

Every player receives a unique, effectively infinite route for each attempt. Replicating and storing separate server-owned geometry for every active player would have created unnecessary state and bandwidth.

Instead:

1. The server assigns an authoritative seed for the attempt.
2. The client deterministically reconstructs chunks and pickup placement.
3. The server validates valuable pickup claims against the seed and reached chunk.
4. Distant chunks unload to keep rendering viable on lower-end mobile hardware.

The server retains authority over progression while high-volume geometry remains local. The trust boundary is explicit: cosmetic construction is delegated; valuable outcomes are checked.

## Turn-Based RPG

- **Focus:** deterministic simulation, utility AI, data-driven state, persistence
- **Ownership:** solo engineering and product design, with contracted map and UI art
- **Status:** unreleased development project

<img width="1566" height="639" alt="Turn-based RPG battle" src="https://github.com/user-attachments/assets/82b57f2b-c266-4e29-8ecc-d7dc6b61714d" />

A creature-collection RPG built around a reusable battle simulator rather than scripted encounters.

### Simulation engine

The battle engine supports variable field sizes, including singles, doubles, triples, bosses, wild encounters, trainers, and a separate wave-based mode.

- Data-driven battlers, types, moves, abilities, evolutions, status effects, field effects, and stat stages
- Phase-based event resolution with battle state driving UI, rewards, progression, catching, and move learning
- Rules kept independent from presentation so alternate modes can reuse the same simulator
- Deterministic state transitions that are easier to inspect, reproduce, and test than visual-scripted battle flow

### Utility AI

Each legal action is scored through heuristic state evaluation. The model considers damage, survivability, type coverage, field advantage, status, switching value, and future-turn risk. Difficulty controls search depth and injected noise rather than granting hidden statistical advantages.

The same utility approach also supports contextual dialogue. After a loss, a dealer evaluates battle context and saved progression state to choose an offer and response intended to be both relevant and persuasive.

### Persistent story state

Story progress, visual location, quests, rewards, and conversation actions can change independently. A data-driven action registry replaced tightly coupled dialogue functions and made those transitions explicit.

The important edge case was rejoining: saved story stage must resolve to a valid area and checkpoint even after the map changes, while the server still verifies that rewards and progression were legitimately reached. Named checkpoints and authoritative stage metadata avoid restoring players to stale world coordinates or accepting client-spoofed progress.

### Performance and onboarding

- Areas load locally on demand and unload with their associated characters, physics, and transient state
- Spatial interfaces are positioned as 3D geometry relative to camera space
- A structured first-time-user funnel introduces the first character, battle, capture, plot hook, and core services within the opening minutes
- Telemetry points were designed around each onboarding milestone for later release analysis

<img width="1917" height="1024" alt="RPG spatial interface" src="https://github.com/user-attachments/assets/8fa1973a-acbf-4133-837a-47e4431547b4" />

## Asymmetrical Horror Shooter

- **Focus:** client/server architecture, runtime validation, procedural generation, dynamic geometry
- **Ownership:** solo engineering and systems design
- **Status:** active prototype

<img width="1572" height="597" alt="Asymmetrical horror shooter" src="https://github.com/user-attachments/assets/fc115a3e-2712-495f-b483-1d235267ade1" />

A round-based multiplayer shooter used to explore stronger trust boundaries and richer runtime systems under replication, streaming, and device constraints.

### Lag-aware server validation

The server stores timestamped player-root history and validates client actions against bounded historical state. This is used for gunfire, close-range attacks, and world interaction without accepting arbitrary client positions.

Doors introduced a less obvious problem: their animation is rendered locally for responsiveness, while server geometry remains static. Door state is represented by shared start and end timestamps. A custom raycast handler calculates how open each panel should be at the queried time and continues through the remaining ray distance when the panel would not block that path.

This preserves responsive presentation without allowing local animation state to become authority over hits or access.

### Dynamic surface projection

The server replicates lightweight impact markers containing a transform, seed, and profile. Each client deterministically reconstructs the same persistent surface geometry.

The renderer:

1. Generates irregular 2D sample geometry from the marker seed.
2. Projects samples into world space with raycasts.
3. Detects surface-normal discontinuities using dot products.
4. Inserts connector vertices where geometry crosses corners or steps.
5. Builds the result into one editable runtime mesh.
6. Culls distant and out-of-view marks to remain within strict mesh limits.

The marker remains cheap and stream-safe while clients handle the expensive visual representation. The same pattern is used elsewhere for client-simulated corpses: the server owns only the authoritative root marker and metadata.

https://github.com/user-attachments/assets/0db751c9-21d1-4a1d-a271-5aa1e42c7144

### Graph-based procedural generation

Facility layouts are generated as data before any geometry is created. Authored room prefabs expose compatible sockets, while a discrete-grid graph defines connectivity.

The generator uses:

- Randomised branch growth and graph-degree-to-room assignment
- Breadth-first search and flood-fill traversal for reachability, distance, components, and empty regions
- Edge/node counts and component checks for cycle validation
- Required through-rooms and terminal-room constraints
- Post-layout repairs that collapse useless intersections and connect compatible dead ends
- Bounded zone retries followed by a full-layout retry when constraints cannot be satisfied

Two generation strategies share the same validation layer: a branching labyrinth and a hub-and-spine layout. Keeping generation in tables until validation completes makes failed layouts cheap to discard and prevents partial geometry from becoming game state.

### Runtime ownership split

- Round state and per-life state are separate, allowing role changes and reinforcement waves without rebuilding the round
- Item state survives drop and pickup while transient action state does not
- First-person presentation is local; authoritative inventory and damage remain server-owned
- Audio, ragdolls, and high-frequency visual systems are reconstructed locally from compact replicated state
- Role behavior is driven by shared configuration and specialised controllers rather than scattered role checks

https://github.com/user-attachments/assets/c8638c51-fc43-4025-827b-4cbcc26bc699

## Engineering approach

Across these projects, I use the same practical rules:

- Keep outcomes authoritative and move presentation to the client when replication cost is high
- Prefer deterministic reconstruction over repeatedly transmitting bulky state
- Model complex behavior as explicit data and state transitions
- Design persistence and rejoin behavior with stale state in mind
- Instrument live products, test assumptions against user behavior, and revise the system that actually limits the outcome

For directly browsable code, see the [Business Experiment Simulator](https://github.com/gruptillionaire/biz-experiment-sim) and [RFP Matrix](https://github.com/gruptillionaire/RFP_TENDERS) repositories.
