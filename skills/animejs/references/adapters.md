# Adapters: Three.js and custom targets

Checked against: `adapters/index.d.ts`, `adapters/registry.d.ts` (`registerAdapter`, `Adapter`, `TargetAdapter`, `TargetAdapterEntry`), `adapters/three/index.d.ts`, `adapters/three/adapter.d.ts` (`threeAdapter`), `adapters/three/instance.d.ts` (`getInstances`, `commitChanges`, `Instance`); behavior and property list from `src/adapters/three/index.js` and `src/adapters/three/object3d.js`; usage from `examples/threejs/transforms`, `tests/suites/threejs.test.js`.

**Not available with the CDN UMD bundle this setup uses.** The `anime` global has no `registerAdapter`, `threeAdapter`, `getInstances` or `commitChanges`, so Three.js targets and custom adapters need the ESM package (npm `animejs` with a bundler, or an import map). Say so before writing any code from this page; the examples below are ESM on purpose.

**Adapters are NOT exported from the root `'animejs'` package or the UMD/ESM bundles.** Import them from subpaths:
- `animejs/adapters`: `registerAdapter` only.
- `animejs/adapters/three`: `threeAdapter`, `getInstances`, `commitChanges`. Importing it is a **side effect** that registers the three.js adapters (`package.json` marks `dist/modules/adapters/**` as `sideEffects`).

Peer dependency: `three >= 0.150.0` (optional).

## Three.js
```js
import 'animejs/adapters/three';             // or: import { getInstances } from 'animejs/adapters/three';
import { animate, createTimeline, stagger, utils } from 'animejs';

animate(mesh,   { x: 2, rotateY: 360, scale: 1.5, opacity: .3, color: '#f0a' });
animate(camera, { fov: [50, 30], focalLength: 35 });
animate(light,  { intensity: [0, 2], color: '#ffaa00' });
utils.set(scene, { background: '#101010' });
```

Mapped properties on `Object3D` (mesh, group, camera, light...):
- Position `x`, `y`, `z`; rotation `rotateX`, `rotateY`, `rotateZ` in **degrees** (Euler order `'XYZ'` assumed); `scaleX`, `scaleY`, `scaleZ`, `scale` (uniform); `opacity`, `color`, `visible`.
- Class specific: `groundColor` (HemisphereLight); `volume` (Audio); `refDistance`, `rolloffFactor`, `maxDistance` (PositionalAudio); `fov`, `aspect`, `near`, `far`, `zoom`, `focalLength` (PerspectiveCamera); `left`, `right`, `top`, `bottom`, `near`, `far`, `zoom` (OrthographicCamera).
- Unmapped names on a mesh are looked up on `mesh.material` (numeric, boolean, `Color` props such as `metalness`, `roughness`, `emissive`, `wireframe`). Materials, textures, fog, `Color`, `Vector2/3/4` can also be animated directly.
- TSL `NodeMaterial` uniforms: `animate(mesh, { colorNode: '#0ff', offsetNodeY: .5 })`, `animate(uniform(0), { value: 1 })`.

Caveats (from source):
- `opacity` writes `material.opacity`: set `material.transparent = true`. `opacity: 0` and `scale: 0` flip `visible = false`.
- Material-routed writes only touch `mesh.material`, not nested children: animate materials directly or pass an array of meshes.
- If a name exists on both mesh and material, the mesh wins.

### Instanced / batched meshes
```js
import { getInstances, commitChanges } from 'animejs/adapters/three';
import { animate, stagger } from 'animejs';

const instances = getInstances(instancedMesh);          // (Instance | null)[]
animate(instances, { y: [0, 2], rotateY: 360, delay: stagger(5), loop: true, alternate: true });
animate(instances[42], { scale: 2 });
```
Instance props: `x y z rotateX rotateY rotateZ scaleX scaleY scaleZ scale skewX skewY skewZ transformOriginX transformOriginY transformOriginZ opacity visible`. Writes are flushed in `mesh.onBeforeRender`; call `commitChanges(mesh)` if you read `mesh.instanceMatrix` between a tick and a render. `opacity` affects the shared material (all instances). `visible` is a no-op on `InstancedMesh` (use `scale: 0`).

### Render loop
Keep your own `renderer.setAnimationLoop(render)`; anime updates values on its own rAF. To tick anime from the same loop, see engine.md (`engine.useDefaultMainLoop = false; engine.update()`).

## Custom adapters (`registerAdapter`)
```ts
registerAdapter(detect?: (t: any) => boolean): Adapter
adapter.registerTargetAdapter(detect: (t) => boolean): TargetAdapter
targetAdapter.registerProperty(name, getter: (t) => any, setter: (target, value: number, tween) => void, gate?: (t) => boolean): void
adapter.registerPropertyResolver(resolver: (target, name) => { get, set, gate? } | null): void
```
For color/complex tweens the setter's `value` is `undefined`; read `tween._numbers`.

```js
import { registerAdapter } from 'animejs/adapters';
import { animate } from 'animejs';

class Sprite { constructor() { this.pos = { x: 0, y: 0 }; } }
const adapter = registerAdapter(t => t instanceof Sprite);
const spriteProps = adapter.registerTargetAdapter(t => t instanceof Sprite);
spriteProps.registerProperty('x', t => t.pos.x, (t, v) => { t.pos.x = v; });
spriteProps.registerProperty('y', t => t.pos.y, (t, v) => { t.pos.y = v; });

animate(new Sprite(), { x: 200, y: 100 });
```
Plain JS objects with numeric properties need no adapter (`animate(obj, { value: 1 })`); adapters are for objects whose state lives in nested structures or setters.
