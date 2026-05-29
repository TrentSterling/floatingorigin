# floatingorigin

**The floating-origin trick, explained, with a live Three.js demo.** Walk an infinite procedural IKEA at full sprint; when you drift more than 300 units from spawn the engine teleports the world back to `(0, 0, 0)` so 32-bit float precision never bites.

- **Live tutorial:** <https://tront.xyz/floatingorigin/>
- **Live demo:** <https://tront.xyz/floatingorigin/demo/>

## The trick

Game engines store positions as 32-bit floats; 7 digits of precision. At `X = 50,000` your smallest representable step is `0.004`. At `X = 1,000,000` it's `0.06`, bigger than a footstep. The camera shakes, physics rattles, vertices shimmer. Every open-world game hits this wall.

The fix: keep the player physically near `(0, 0, 0)` at all times. When they drift past a threshold, teleport them back to zero and shift the entire world the same amount in the same direction. From inside the engine, nothing changed; the coordinates the renderer sees stayed small and precise.

Maintain a `globalOffset` in 64-bit doubles for the absolute world position. Hash chunk world-coords with a fixed seed for deterministic procgen with zero persistence.

```js
if (length(engineCam.xz) > THRESHOLD) {
  const shift = { x: engineCam.x, z: engineCam.z };
  globalOffset.x += shift.x; globalOffset.z += shift.z;
  engineCam.x  -= shift.x;   engineCam.z  -= shift.z;
  for (const mesh of activeChunks.values()) {
    mesh.position.x -= shift.x;
    mesh.position.z -= shift.z;
  }
}
```

Full walkthrough on the tutorial page: pseudocode, Unity C# implementation (`FloatingOrigin.cs`), threshold tuning, why parenting one big `worldRoot` doesn't work, and the gotchas (rigidbodies, particles, trails, audio, navmesh, networked clients).

## Files

| Path | What |
|------|------|
| `index.html` | The tutorial (long-form document, embeds the demo via iframe) |
| `demo/index.html` | The standalone Three.js sandbox; infinite procedural IKEA, telemetry, threshold ring, chunk minimap, in-demo tutorial modal |

## Credits

Floating origin goes back at least to FlightGear in the late 90s; flight sims hit the precision wall before anyone else because their playable area was the actual surface of the Earth. Unity calls it "floating origin"; Unreal ships "World Origin Rebasing". Three.js + IKEA implementation, tutorial, and page by Tront.

## License

MIT (see `LICENSE`). If you ship something built on this, drop a note in the [tront.xyz Discord](https://tront.xyz/discord/).
