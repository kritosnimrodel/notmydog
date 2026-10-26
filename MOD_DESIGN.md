# Not My Dog — Design Spec

Inspired by Lookout3d's Jerry Dog Series.

A single-player horror Fabric mod for Minecraft Java Edition. Public-release quality. Should work alongside common mods, including Sodium and Iris. Fabric API is the main dependency.

## Project

- Minecraft 26.3, Java 25, Fabric Loader and Fabric API. Versions are in `gradle.properties`. Verify them against https://fabricmc.net/develop before release.
- Based on the official `fabric-example-mod` template (branch `26.3`). It has a common source set and a client source set, plus mixin configs.
- Always check Fabric and Minecraft APIs against the official docs (docs.fabricmc.net) and the template source. Don't rely on memory for API names.

## Project Identity (use these names everywhere)

- **Mod name:** Not My Dog
- **Mod ID:** `notmydog`
- **Base package:** `woof.example` (`woof.example.client` for client code, `woof.example.mixin` and `woof.example.client.mixin` for mixins)
- **Mod JAR:** `notmydog-1.0.0.jar` (from `settings.gradle` `rootProject.name`)
- **Key files:**
  - `src/main/resources/fabric.mod.json`: mod metadata and entrypoints
  - `src/main/java/woof/example/ExampleMod.java`: main entrypoint, `MOD_ID = "notmydog"`
  - `src/client/java/woof/example/client/ExampleModClient.java`: client entrypoint
  - `src/main/resources/notmydog.mixins.json` and `src/client/resources/notmydog.client.mixins.json`: mixin configs
  - `src/main/resources/assets/notmydog/`: assets (icon, textures, models, etc.)
  - `build.gradle`: Loom config with the `notmydog` mod entry
  - `gradle.properties`: Minecraft, Loom, Fabric API, and mod version settings
- **Java 25 is required** to build.
- **Build:** `gradlew.bat build` (Windows) or `./gradlew build`. `gradlew.bat runClient` starts the dev client.

## Feature 1 — More Wolves, Cheaper Taming

- More wolves spawn in the world, including in biomes where vanilla wolves don't spawn.
- Taming requires fewer bones than vanilla.

## Feature 2 — Blood, Bodies, and Corpses

- Players and wolves bleed when damaged.
- When a player or wolf dies, it leaves a blood pile.
- When a wolf dies, it leaves a wolf corpse.
- A carried corpse makes blood drip from its carrier.
- **Gore toggle (setting):** Turns off all gore, including blood on players and wolves, blood piles, and corpses.

## Feature 3 — The Wolf Mimic

### Concept
A creature that hunts and kills player-tamed wolves, then takes their place.

### States
- **Disguised:** Looks identical to a wolf. Its body can't be recovered after it kills a dog, but blood drops are left nearby.
- **True form:** A massive, lanky, wolf-like creature.
- **Grin:** The disguised form after the player has pet it while it is standing. It stalks normally, but one dog has a wide grin and dangerous eyes. This state is permanent.
- **Dead:** The mimic is killable. When it dies, a loud howl plays and a large burst of blood appears. The den is no longer guarded, and surviving dogs become immune to future mimics.

### Behavior (disguised)
- Rarely acts when the player is near.
- Struggles to follow sit/stand commands.
- May push the player off a large drop.
- Kills other dogs one at a time when the player is away from them. Leaves bodies and a lot of blood.
- Steals dog bodies the player moves or buries. Only when the player is far away and at least one solid block is between the player and the body.
- Can carry bodies only when it thinks it isn't being observed.
- With many wolves, kills one in secret. With pups, kills a pup in front of the others.
- Dogs and pups with a clear line of sight to a pup kill won't approach the mimic. Pups that grow up after witnessing a kill are very hostile toward that specific mimic.
- Only targets one dog at a time. It finishes its current target before choosing another. Newly tamed dogs are targeted only after the current target dies.
- Cats deter it in disguised form (it flees from them).
- Attacking it, or hitting it below the reveal threshold, reveals the true form.
- Damage taken while disguised works like a normal wolf. A wolf with 40 HP reveals the mimic at 12 HP, but only if player damage contributed.
- If it dies in wolf form from non-player damage, it roars, shifts to true form, and leaves a wolf corpse.
- Dogs fight it in any phase. If the player is watching, this can trigger the hostility phase.
- Can be petted in disguised form.
- If the player never leaves the dogs, nothing happens. This is rare.
- If the player has only one dog, the mimic replaces the lone wolf and acts a bit distant.

### Behavior (true form)
- Chases and attacks the player on sight.
- Attacks cats with focus.
- If the player escapes, it races the player to base. If it arrives first, it shifts back to disguise and acts as if it was always there.
- If the player arrives first, it screams and runs away, then roams the world, sometimes approaching the base. It attacks if noticed.
- Roaming is out of sight. The player never sees it run.

### The Race
- Timer starts when the player leaves the den. The duration is a fixed number of seconds calculated from route terrain, environment, and distance.
- Heartbeat gets louder and faster as time runs out. When the timer gets low, a red vignette appears and pulses with the heartbeat.
- Standing still or going the wrong way drains the timer quickly.
- If the timer hits zero, the player dies.
- Reaching base with **10 seconds or less** left shows: *"Somehow you outran it."*
- Reaching base with more time left shows: *"You outran it. You're safe."*
- If the mimic had already reached base before the player, shows: *"You outran it. Something seems missing, however."*
- The "Something seems missing" message means the mimic is back in the house in disguise. The player should believe the odd dog and the mimic are separate.

### Den
- Generated structure, placed a set distance from the house in any direction.
- Access: either part of the den is visible underground, or there's a suspicious spruce trapdoor in an otherwise empty spot.
- The mimic brings stolen bodies here and shows its true form inside.
- The den is open only while the mimic is roaming. When the mimic dies, the den stays open.
- When abandoned, the den is filled with natural terrain material (no ores), so nothing seems to have been there.
- Generates after the mimic has been with the player for a few days.
- Newly tamed dogs that get mimicked build a new den.

## Open Questions (to decide during implementation)

- **Unloaded chunks:** Whether the mimic's running time, roaming, and den state progress while the player is far away or the chunk is unloaded. Default to a sensible simulation approach and document it.
- **Mimic persistence:** The mimic persists across sessions with its own state. Save data per world.
- **Witness tracking:** Store which dogs saw each pup kill. Choose storage that survives saves.
- **Exact timer values:** Race durations and "a few days" need concrete numbers. Tune in playtesting.
- **Wrong-direction rule:** The exact definition of "not facing your bed" and the drain rate.
- **Stone fill:** Use the terrain's natural material.

## Compatibility

- Works with Sodium and Iris.
- Uses Fabric API. Small supporting mods are fine but not required.
- Settings screen should include at least the gore toggle.

## Suggested Structure

- Separate the mimic's state machine from its rendering and AI goals.
- Keep tuning values (timings, distances, thresholds) in one config.
- Keep persistent mimic/den state in a saved world data class.
- Keep UI (vignette, heartbeat, messages) client-side.
