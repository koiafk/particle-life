\# AGENTS.md



\## What this is

This project is a zero-dependency 2D particle life simulator in a single index.html file. It models emergent multi-agent self-organization across 1000 particles and 6 color types governed by a 6x6 interaction matrix and toroidal spatial hashing.



\## Architecture

1\. Frame loop stages:

&#x20;  - Measure frame duration for adaptive particle downscaling.

&#x20;  - Paint semi-transparent background rectangle (`rgba(0,0,0,0.07)`) to preserve motion trails.

&#x20;  - Spatial hash construction: counting-sort particle indices into linear cell buckets sized to `rmax`.

&#x20;  - Force accumulation: evaluate neighbor cells, apply toroidal minimum distance, compute pairwise forces, and integrate velocity damping and speed clamping.

&#x20;  - Batched canvas rendering: set composite mode to `lighter`, draw circles grouped by type (6 draw batches), then composite radial vignette.

2\. Data layout:

&#x20;  - Particle positions: `x` (`Float32Array(N)`), `y` (`Float32Array(N)`).

&#x20;  - Particle velocities: `vx` (`Float32Array(N)`), `vy` (`Float32Array(N)`).

&#x20;  - Particle types: `type` (`Uint8Array(N)`), holding values `0..5`.

&#x20;  - Interaction matrix: `matrix` (`Float32Array(36)`), row-major format mapping `typeA \* 6 + typeB`.

&#x20;  - Spatial grid: `cellCounts` (`Int32Array(numCells)`), `cellStarts` (`Int32Array(numCells + 1)`), `cellOffsets` (`Int32Array(numCells)`), and `pIndices` (`Int32Array(N)`).

3\. Spatial hash counting sort:

&#x20;  - Pass 1: zero `cellCounts`, iterate `0..N-1` incrementing particle bin counts.

&#x20;  - Pass 2: compute running prefix sums into `cellStarts`.

&#x20;  - Pass 3: duplicate `cellStarts` into `cellOffsets`, write particle index `i` into `pIndices\[cellOffsets\[cell]++]`.

4\. Force accumulation:

&#x20;  - For particle `i`, loop through 9 neighbor cells `(cx ± 1, cy ± 1)` with toroidal modulo.

&#x20;  - For neighbor `j`, compute toroidal `dx, dy`. If `0 < d < rmax`: apply repulsion `d/rmin - 1` when `d < rmin`, else apply `matrix\[ti\*6 + tj] \* (1 - d/rmax)`.

5\. Batched rendering:

&#x20;  - Iterate types `t = 0..5`. Set `ctx.fillStyle = palette\[t]`, begin path, loop particles matching type `t`, issue `moveTo` and `arc`, call `ctx.fill()` once per type.



\## Invariants — do not break

\- No heap allocations inside the frame loop: reuse typed arrays and preallocated grid buffers.

\- Grid cell size must strictly equal `rmax`.

\- All distance math must use the toroidal minimum image convention.

\- The `palette` array and `matrix` must both have exactly 6 entries (36 coefficients).

\- `matrix` must remain a flat 36-length `Float32Array` in row-major order.

\- The URL hash is the sole state that persists; changes must serialize into hash without reloading.

\- UI must remain toggleable via the `H` key.



\## Tuning constants

\- `N\_INIT = 1000`: initial particle count; safe range: 200 to 3000.

\- `RMAX = 80`: maximum interaction radius; safe range: 50 to 140.

\- `BETA = 0.3`: repulsion boundary fraction (`rmin = BETA \* RMAX`); safe range: 0.15 to 0.45.

\- `DAMPING = 0.85`: per-frame velocity decay factor; safe range: 0.70 to 0.95.

\- `MAX\_SPEED = 6.0`: maximum velocity magnitude clamp; safe range: 3.0 to 12.0.

\- `FORCE\_SCALE = 0.6`: acceleration multiplier applied to force vectors; safe range: 0.2 to 1.5.

\- `TRAIL\_ALPHA = 0.07`: opacity of the trail fade rect; safe range: 0.03 to 0.20.

\- `PARTICLE\_RADIUS = 2.0`: radius of rendered particle circles; safe range: 1.0 to 3.5.



\## Recipes

1\. Add a 7th particle type:

&#x20;  - Change type count constant from 6 to 7.

&#x20;  - Append 1 hex color to `palette` and allocate `matrix = new Float32Array(49)`.

&#x20;  - Update CSS grid definition `.matrix { grid-template-columns: repeat(7, 18px); }`.

&#x20;  - Update hash packing and unpacking from 36 matrix bytes to 49 bytes.

2\. Add a new force shape:

&#x20;  - Locate the force conditional in `updateForces()`.

&#x20;  - Replace `matrix\[ti \* 6 + tj] \* (1 - d / RMAX)` with custom curve (e.g. `matrix\[...] \* (1 - Math.abs(2\*d - (RMAX + rmin)) / (RMAX - rmin))`).

&#x20;  - Keep short-range repulsion `d < rmin` intact to prevent particle collapse.

3\. Add a preset world:

&#x20;  - Define a static 36-element array of float coefficients.

&#x20;  - Create a preset trigger handler assigning `matrix.set(presetArray)`.

&#x20;  - Call `renderMatrixUI()` and `syncHash()` to update editor visuals and URL.

4\. Add a second interaction mode on right-click:

&#x20;  - Add `isRightClick` flag to pointer state via `e.button === 2`.

&#x20;  - In pointer impulse loop, if right-click is true, apply tangential vortex force `vx += (-dy / d) \* str`, `vy += (dx / d) \* str`.

&#x20;  - Attach `e.preventDefault()` to canvas `contextmenu` to suppress browser menu.

5\. Add a PNG export button:

&#x20;  - Insert `<button id="btn-export">export</button>` in the controls DOM.

&#x20;  - Add click listener: `const a = document.createElement('a'); a.download = 'particles.png'; a.href = canvas.toDataURL('image/png'); a.click();`.

6\. Add an FPS counter:

&#x20;  - Insert `<span id="fps"></span>` in the bottom-right panel.

&#x20;  - In frame loop, track `frames++`; every 500ms compute `Math.round(frames \* 1000 / dt)`, update span text, and reset counter.

7\. Port the force loop to a WebGL compute path:

&#x20;  - Initialize WebGL2 compute or WebGPU device context.

&#x20;  - Bind position and velocity storage buffers and upload matrix uniform buffer.

&#x20;  - Execute compute shader computing spatial cell binning or direct pairwise interaction, then issue instanced point drawing.



\## Style rules

\- Plain functions only; no classes, no prototypes, no closures inside hot paths.

\- Typed arrays for all numeric simulation data.

\- Boring, concise variable names (`p`, `vx`, `vy`, `fx`, `fy`, `rmax`, `n`).

\- At most 3 comments in JavaScript, placed strictly where math or spatial hashing is non-obvious.

\- No emoji anywhere in code, markup, or documentation.

\- Zero external dependencies, fonts, CDNs, or build tools.



\## Verification checklist before committing

\- \[ ] Viewport test at 380px mobile width: canvas fills screen, no scrollbars, touch drag stirs.

\- \[ ] Viewport test on 4K desktop: canvas scales crisp with devicePixelRatio without lag.

\- \[ ] 60 FPS verified at 1000 particles on standard hardware.

\- \[ ] URL hash serialization: modifying matrix or count updates hash, and refreshing restores state.

\- \[ ] Pressing `H` hides all UI cleanly for screenshot recording.

\- \[ ] Zero console errors or warnings across load, interaction, and resize.

\- \[ ] Simulation produces cohesive moving structures (cells, chains, or clusters) within 5 seconds of load.

