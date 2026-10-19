# Procedural Jellyfish

A swimming jellyfish built procedurally in Houdini (SOPs · VEX · Vellum). One animated bell drives everything: the veins and organs ride on it, and the oral arms and tentacles are simulated hanging from it.

<p align="center"><img src="media/jellyfish.gif" alt="Animated jellyfish pulsing and swimming, with arms and tentacles trailing" width="300"></p>
<p align="center"><a href="media/jellyfish.mp4">▶ Full-resolution video (mp4)</a></p>

## How it fits together

```mermaid
flowchart LR
    BELL["<b>Bell</b><br/>Line · Bend ×2 · Revolve<br/>Refine · Extrude"] -- "Time Shift f1" --> REST(("rest bell"))
    REST --> SURF["Veins · Organs"] -- "Point Deform" --> OUT[["OUT_JELLYFISH"]]
    REST --> CURVES["Arm spines · Tentacles"] -- "pinned roots" --> SIM["Vellum Hair<br/>one solver"]
    SIM --> OUT
    BELL -. "deform / pin target" .-> SURF
    BELL -. "looped, pre-rolled" .-> SIM
```

Every part is built on the bell frozen at frame 1, then carried by the animated bell. Surface parts use **Point Deform**. Simulated parts are **pinned with Match Animation** and run in a single Vellum solver that pre-rolls one full swim cycle, so the animation loops seamlessly.

## Parts

<table>
<tr>
<td><img src="media/bell.png" alt="Translucent bell highlighted" height="240"></td>
<td><b>Bell</b><br>A line bent twice and revolved into a dome, with noise and a thin extruded shell. Keyframed bend angles and height on a <code>CONTROLLER</code> null drive the pulse. Displayed translucent so the organs show through.<br><sub><code>Line</code> · <code>Bend ×2</code> · <code>Revolve</code> · <code>Attribute Noise</code> · <code>Poly Extrude</code></sub></td>
</tr>
<tr>
<td align="center"><img src="media/arms.png" alt="Four oral arms highlighted" height="240"></td>
<td><b>Oral arms</b><br>A grid ruffled in VEX, twisted and remeshed, tapered where it leaves the mouth. Four copies flare out from the mouth and hang as long as the tentacles. Each arm follows a simulated Vellum spine that is softly pinned to its rest shape, and the mesh is bent along it in VEX.<br><sub><code>Grid</code> · <code>Wrangle</code> · <code>Bend (twist)</code> · <code>Remesh</code> · <code>Copy to Points</code> · <code>Vellum Hair</code></sub></td>
</tr>
<tr>
<td><img src="media/veins.png" alt="Branching veins on the bell highlighted" height="240"></td>
<td><b>Veins</b><br>Shortest paths on the remeshed bell, from roots near the crown to tips on the rim and mid-bell. A vertically stretched noise cost makes them wander and share trunks, which become thicker where more paths overlap.<br><sub><code>Remesh</code> · <code>Find Shortest Path</code> · <code>Fuse</code> · <code>Polypath</code> · <code>Resample</code> · <code>Smooth</code> · <code>Sweep</code> · <code>Point Deform</code></sub></td>
</tr>
<tr>
<td><img src="media/organs.png" alt="Four horseshoe-shaped organs under the crown highlighted" height="240"></td>
<td><b>Organs</b><br>Four wobbly horseshoe curves generated in VEX, opening toward the center. They are dropped onto the underside of the crown and swept into tubes with rounded ends.<br><sub><code>Wrangle</code> · <code>Sweep</code> · <code>Point Deform</code></sub></td>
</tr>
<tr>
<td align="center"><img src="media/tentacles.png" alt="Long tentacles hanging from the rim highlighted" height="240"></td>
<td><b>Tentacles</b><br>40 thin curves (2.25–3.75 long) hung from the rim, with roots snapped exactly onto the moving rim. Vellum Hair with low gravity and strong drag to feel like water, and the bell as a collider.<br><sub><code>Wrangle</code> · <code>Vellum Hair</code> · <code>Vellum Solver</code> · <code>Sweep</code></sub></td>
</tr>
</table>

## Extra Credit: Refined bell

<p align="center"><img src="media/refined_bell.png" alt="Refined bell with scalloped lappets, pleats and dents, relaxed and contracted" width="70%"></p>
<p align="center"><img src="media/refined_bell.gif" alt="Refined bell pulsing, pleats deepening as it contracts" width="360"><br><sub>relaxed (left) vs. contracted (right), and the pulse</sub></p>

One VEX pass after the revolve, placed in surface coordinates (angle around the axis, distance from the rim) so the details stick to the bell while it animates:

- **Scalloped margin**: 16 rounded lappets, made by lifting the rim between lobes with `1 - |sin(8θ)|^0.5`.
- **Pleats**: radial grooves running up from each notch. They deepen as the bell contracts by reading the same `bendMain` channel that drives the pulse.
- **Dents**: noise dimples sampled in a polar projection so they stay smooth at the crown.

Everything downstream (veins, organs, tentacles, the simulation's collider) picks up the new shape automatically.

## Gotchas

- Vellum writes hair thickness to `pscale`, and Sweep multiplies its radius by it. Turn off *Transform Using Curve Attributes*, or the tentacles all but vanish.
- To loop a simulation, cycle the bell with a Time Shift (`$FF - 139*floor(($FF-1)/139)`) and start the solver one cycle early, so frame 140 matches frame 1.
