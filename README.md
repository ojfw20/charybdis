# charybdis (ZMK config)

ZMK config for my wireless Charybdis nano: two nice!nano v2 halves over Bluetooth, trackball on the right, Colemak-DH. This is the source of truth for the firmware; builds run in GitHub Actions.

## Hardware
- Controller: nice!nano v2 (nRF52840) on each half.
- Split: direct BLE, no dongle. The right half is the central (talks to the host) and carries the PMW3610 trackball; the left half is the peripheral.
- ZMK comes from my own fork `ojfw20/zmkTrackballFork`, pinned by commit in `config/west.yml` (trackball driver, jitter fix, and the rotation option below).

## Build
1. Actions tab -> "Build ZMK firmware" -> pick the branch -> Run/Re-run.
2. Download the `firmware` artifact. It contains:
   - `charybdis_left-nice_nano_v2-zmk.uf2`
   - `charybdis_right-nice_nano_v2-zmk.uf2`
   - `settings_reset-nice_nano_v2-zmk.uf2`

Artifacts expire after 90 days, so re-run the build when you need fresh ones.

## Flash
Do both halves. Wipe with `settings_reset` first, then flash that side's firmware.

For each half:
1. Plug it into USB.
2. Double-tap reset -> it mounts as a `NICENANO` drive.
3. Drop `settings_reset` onto it (clears settings and BLE bonds; won't type afterwards).
4. Double-tap reset again.
5. Drop the matching side firmware: left gets `charybdis_left`, right gets `charybdis_right`. Don't swap sides - the trackball only works on the right image.

Once real firmware is on, the `&bootloader` key on the tri-layer is the easy way back into the bootloader for future updates.

## Pair to a Mac (or any BLE host)
1. Power both halves.
2. On the keyboard, reach the tri-layer (hold both nav thumbs) and press `&bt BT_SEL n` for an unused profile.
3. Host: Bluetooth settings -> connect. ZMK BLE is native to macOS - no extra host setup.

Stale bond after reflashing (the usual snag): "Forget This Device" on the host and `&bt BT_CLR` on that profile, then re-pair. If a half stops talking, re-flash `settings_reset` to both, then the real firmware.

## Trackball rotation (tilted-bracket correction)
The PMW3610 driver only offers 90-degree orientation steps, which can't correct for a bracket that yaws the right half by an odd angle (forward ends up diagonal). The fork adds `CONFIG_PMW3610_ROTATE_DEG`, applied after the coarse orientation/invert stage.

Tune it in `config/boards/shields/charybdis/charybdis_right.conf`:
- Edit `CONFIG_PMW3610_ROTATE_DEG` (degrees), rebuild, reflash the right half only, and push the ball straight forward.
- If the cursor deviates further, flip the sign. If it's close, adjust the magnitude. A few passes converges.
- `0` disables it with no runtime cost.
