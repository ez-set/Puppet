# Puppet

Shows your VRM avatar and moves it with motion data from SelfReflect or any other VMC sender.

Apple Silicon Macs. Built and tested on macOS 15. Not tested on Intel Macs or older macOS versions.

<img width="472" height="627" alt="Screenshot 2026-11-03 at 8 13 03 AM" src="https://github.com/user-attachments/assets/3da005c8-3246-44e5-953c-2840abb8ea16" />

## Install

1. Download `Puppet.zip` from [Releases](../../releases/latest) and unzip it.
2. Drag `Puppet.app` into Applications.
3. Open it. macOS will block it the first time, because the app is not signed by Apple. Click Done.
4. Open System Settings, then Privacy & Security. Scroll to Security and click **Open Anyway** next to Puppet. Enter your password.
5. Open Puppet again.

The Open Anyway button can disappear after about an hour. Open the app again to bring it back.
Right-click and Open does not work on macOS 15.

## Use

1. Drop a `.vrm` file on the window, or click the box in the middle. VRM 0.x and 1.0 both work. Puppet remembers it.
2. In SelfReflect, or any VMC sender, set the VMC output to the address at the bottom of the window. Both devices must be on the same network.
3. Allow incoming connections if macOS asks.

The dot beside the address turns green when data arrives. A few seconds later the address fades out.

Right-click for the background, mirror and calibration. Scroll to zoom from bust-up to full body.

## Calibrate

Camera-based senders often read a normal seated posture as tilted or leaning forward.
Sit the way you will while streaming, right-click, and click the crosshair. Puppet waits 3 seconds, then treats that posture as straight for the hips, spine, chest, neck and head. Arms and legs are not changed.
The arrow button clears it. Calibrate again if you change seats or move the phone.

Face, head, eyes, torso, arms, hands, fingers, hips and legs are driven. Hips rotate but never move, so a seated pose floats. Feet are kept flat on the floor, since camera-based senders often guess their angle badly.

## OBS

Add a Window Capture of Puppet. Set the background to green and add a Chroma Key filter.

## If something is off

| | |
|---|---|
| The dot stays grey | No data is arriving. Check both devices are on the same network, the address is typed exactly, and macOS did not block incoming connections. |
| The address shows `:39539 busy` | Another app is using that port. Quit it, or change `port` in the settings file. |
| `can’t open that file` | Puppet opens `.vrm` files only. |
| Arms or legs bend the wrong way | Open an issue with the model name and what bends. |

The settings file is `~/Library/Application Support/app.puppet.desktop/settings.json`.

## Build from source

Needs [Node](https://nodejs.org), [Rust](https://rustup.rs) and the Xcode command line tools.

```
git clone https://github.com/YOURNAME/puppet.git
cd puppet
npm install
npm run dev
```

The first run compiles Rust and takes a few minutes. To make the app:

```
npm run build -- --bundles app
```

It lands in `src-tauri/target/release/bundle/macos`.

```
npm test
python3 tests/render.py
```

`npm test` runs the TypeScript and Rust unit tests. `render.py` is optional. It renders real models headlessly and checks pose and facing. It needs `pip install playwright`, then `playwright install chromium`, then `npm run build:web`.

### Run options

```
puppet --port 39539 --vrm path/to/model.vrm
puppet --record capture.jsonl
puppet --replay capture.jsonl
python3 scripts/send_fixture.py
```

`--record` saves what the sender sends. `--replay` plays it back without a phone. `send_fixture.py` sends a recording to a running app as real VMC packets.

### Layout

| | |
|---|---|
| `src-tauri/src/vmc.rs` | decode VMC, UDP receiver, record and replay |
| `src-tauri/src/main.rs` | window, commands, one event stream to the UI |
| `src-tauri/src/settings.rs` | settings file |
| `src/vmc.ts` | bone mapping, Unity to model rotation conversion |
| `src/expressions.ts` | ARKit blendshapes to VRM expressions |
| `src/avatar.ts` | load a VRM, follow the pose |
| `src/stage.ts` | camera, lights, zoom |
| `src/main.ts` | wiring and the small UI |

The Rust side sends `frame` (bone rotations and blendshapes, about 60 per second) and `status` (address, live, error) events. Everything else is the UI's job.

VMC rotations are in Unity space. VRM 1.0 models are mirrored across X and VRM 0.x across Z; `conventionFor` picks from the model.
