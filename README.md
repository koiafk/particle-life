# particle life

A self-contained artificial life simulator running 1000 multi-type particles on a continuous toroidal 2D canvas.

https://demo.example.com/particle-life

No cell or creature is programmed; they emerge from the force rules. Particles follow pairwise attraction and repulsion matrices with smooth distance falloff, self-organizing into membrane-bound cells, predatory serpents, and symbiotic clusters. Overlapping particles bloom through additive blending while leaving persistent motion trails.

Drag to stir; press space to pause, r to randomize rules, c to clear, and h to toggle UI.

The 6x6 matrix and particle count serialize directly into the URL hash, making any emergent ecosystem shareable with a link.