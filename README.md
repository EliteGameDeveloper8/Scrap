# Scrapwheel

A 2D physics game that runs in the browser. Build a vehicle from parts on a grid, drive it through missions, earn coins and new parts, and build something better. No install, no build step, no server.

- A grid builder with frame blocks, wheels, engines, fuel tanks, rocket boosters, gyroscopes, bumpers and cargo beds
- Real physics: weight, centre of mass, suspension, grip, torque, thrust and damage all matter
- Parts break off on hard hits, and anything no longer joined to the driver's seat falls away
- Ten missions plus a test track: hills, rocks, gaps, steep climbs, low tunnels, rope bridges, long hauls, cargo deliveries and low gravity
- Three stars per mission: finish, beat the time, and use few parts
- New parts as mission rewards, and a shop for buying more with coins
- Saved builds, share codes for swapping designs, and progress kept on the device
- Keyboard and touch controls, synthesised sound, particles

## Running the game

**Quickest:** open `index.html` in a modern browser (Chrome, Edge, Firefox or Safari). Everything works offline; the physics engine is bundled in `lib/`.

**With a local web server** (recommended while developing):

```bash
npm start            # serves the folder at http://localhost:8080
# or, without Node:
python3 -m http.server 8080
```

**Publishing:** the folder is a static site. Upload it as-is to GitHub Pages, Netlify, itch.io (as an HTML game) or any web host.

Progress, saved builds and settings are stored in the browser's `localStorage`.

## How to play

1. **Play** and pick a mission. The garage opens with the mission's goals and rewards.
2. Build: pick a part at the bottom and click the grid. Every block must connect to the driver seat; each wheel hangs on the block next to it. The panel shows mass, speed, pull, steepest hill, fuel, boost and toughness, and warns about problems.
3. **Drive!** Reach the flag (or deliver the crate) without the driver's head touching the ground.
4. Win stars, coins and parts, then come back and improve the design.

Things worth thinking about while building: the centre of mass (the black and white dot), ground clearance, where boosters push, how much weight each engine has to pull, keeping the crate on the bed, and protecting the seat.

## Controls

| Action | Keys |
| --- | --- |
| Accelerate / brake and reverse | D / A or → / ← |
| Tilt back / forward (in the air and on the ground) | W / S or ↑ / ↓ |
| Rocket boost | Space |
| Restart / pause | R / Esc |

**Builder:** click or drag to place, right-click to remove, R to rotate wedges and boosters (or click a placed one again), X for the remove tool, 1–9 and 0 to pick a part, Ctrl+Z to undo.

On touch screens, on-screen pedals, tilt buttons and a boost button appear during the drive.

## How the code is organised

The game is plain JavaScript with no build step. Every file in `js/` is a classic `<script>`; they load in the order listed at the bottom of `index.html` and share one global object, `SW`. Each file starts with a comment explaining what it contains.

```
index.html               page markup: screens, garage, HUD, windows, and the script list (load order)
css/style.css            all styling
lib/planck.min.js        Planck.js 1.0.6, a Box2D physics port (third-party)
js/
  core/
    util.js              SW namespace, seeded random numbers, smooth noise, HTML escaping
    config.js            SW.CFG: physics constants, parts, starting garage, missions, themes
    save.js              SW.Save: profile, saved builds, share codes, settings
  build/
    blueprint.js         SW.Blueprint: a design on the grid, validity checks and stats
    garage.js            SW.Garage: the builder screen
  world/
    track.js             SW.Track: course generation from a mission's segments
    world.js             SW.World: the physics world for a course (ground, rocks, bridges, pickups)
  game/
    vehicle.js           SW.Vehicle: a design as a physics vehicle, controls, damage and breaking
    run.js               SW.Run: one drive, win and fail rules, stars and rewards
  render/
    parts.js             SW.Draw: how every part looks (shared by the builder and the drive)
    effects.js           SW.Effects: particles
    render.js            SW.Render: drawing a drive
  ui/
    ui.js                SW.UI: screens, missions, shop, builds, results, confirm and toasts
    hud.js               SW.HUD: fuel, progress, time and coins while driving
    input.js             SW.Input: keyboard and touch controls
    audio.js             SW.Audio: synthesised engine, boost and effect sounds
  main.js                SW.Game: game flow and the frame loop
tools/smoke-test.js      automated browser test
```

### Key state

| Name | What it is |
| --- | --- |
| `SW.Save.profile()` | Coins, owned parts, mission progress (stars, best times) and the design on the bench |
| `SW.Run.current()` | The drive in progress: track, physics world, vehicle, crate, time, coins |
| `SW.CFG` | All data: parts, missions, physics and balance numbers |

A blueprint is `{ name, susp, cells: [[col, row, part, rotation], ...] }`, so builds stay tiny in storage and in share codes.

### Common changes

- **New part:** add it to `PARTS` in `js/core/config.js` (mass, hp, price, and any of `power`, `fuel`, `thrust`, `gyro`, `cargo`, `armor`), draw it in `js/render/parts.js`, and add it to the palette order in `js/build/garage.js`. Behaviour that is not just a stat goes in `js/game/vehicle.js`.
- **New mission:** add an entry to `MISSIONS` in `js/core/config.js`: its track segments, par time, part limit and reward. Segment types are listed at the top of `js/world/track.js`.
- **New kind of terrain:** add a segment type in `js/world/track.js`, plus its physics in `js/world/world.js` if it has moving bodies.
- **Balance:** engine force, top speed, boost, fuel use, tilt strength and damage are at the top of `js/core/config.js`.

Formatting uses Prettier (`npm run format`, settings in `.prettierrc.json`).

## Testing

```bash
npm install
npx playwright install chromium
npm test
```

The smoke test opens the game in a headless browser and plays through it: the mission list and locks, placing, removing and undoing parts, design checks, saving builds and share codes, driving with the keyboard, finishing a mission and its rewards, crashing, unlocking, the shop, the test track, cargo delivery, keeping progress after a reload, and the phone layout. Screenshots go to `test-output/`.

## License

MIT, see [LICENSE](LICENSE). Planck.js is also MIT licensed.
