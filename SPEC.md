# The .slfy model format, version 0.1 (draft)

Status: draft, 2026-11-10. The key words MUST, SHOULD and MAY are used as in RFC 2119.

## 1. Scope

A `.slfy` file describes one engineering model: what its parts are and how they are shaped, how they are
placed and move relative to each other, which sizes are adjustable, and the calculations and charts that go
with it. A **reader** opens files; a **viewer** also draws them; a **writer** creates them.

### Conformance levels

| Level | A conforming viewer… |
|---|---|
| **L1 Geometry** | draws every part from its shapes at its `position` and `rotation`, with parameters at their values |
| **L2 Mechanism** | also places parts by `mount` and plays `motion`, including parts driven through mounts |
| **L3 Engineering** | also evaluates `calculations` and `charts`, shows `annotations`, lets parameters change live, and shows the `question` |

A viewer MUST state the level it supports. Any reader MUST NOT fail on a valid file because it does not
support a higher level; it ignores what it does not use.

## 2. The file

UTF-8 JSON, one object:

| Field | | |
|---|---|---|
| `format` | MUST | the string `"slfy"` |
| `version` | MUST | `"MAJOR.MINOR"`, `"0.1"` for this document |
| `model` | MUST | what the model is (section 5) |
| `view` | MAY | how it is first shown (section 12); readers MAY ignore it |
| `meta` | MAY | `generator`, `created` (ISO date), `license` (SPDX id), `source` |
| `extensions` | MAY | namespaced additions (section 13) |
| `$schema` | MAY | URL of the JSON Schema: `https://slfy.solidify.studio/schema/slfy-0.1.schema.json` |

**Versions.** A reader of MAJOR.x SHOULD open any file of the same MAJOR, ignoring fields it does not know.
A newer MAJOR MAY change meanings; a reader SHOULD warn and try anyway. Before 1.0, minor versions MAY break.

**Size.** Files SHOULD stay under 2 MB. Lists have limits (see the schema): at most 32 top-level parts,
24 children per part, 32 shapes per part, 20 parameters, 20 calculations, 6 charts, 24 annotations.

## 3. Units and coordinates

- **Geometry is always in metres**, right-handed, **Y up** (as glTF). Angles in shapes, parts and mounts are
  **radians**, except where a field says degrees.
- `model.units` (`"mm"` or `"m"`) is only the unit sliders are shown in. Parameters keep their own units; a
  size expression converts explicitly: a diameter slider `d` in mm gives a radius `"d / 2000"`.
- **Round shapes turn about their local Y axis** (cylinders, tubes, cones, pulleys, gears, sprockets, tori,
  springs, lathe profiles). Every shape is centred on its own origin unless stated.

## 4. Expressions

Any field typed **DimExpr** is either a number or a string expression. Expressions are pure arithmetic: no
assignments, no code. A reader MUST NOT evaluate them as program code.

**Grammar** (whitespace ignored):

```
expr   := term (("+" | "-") term)*
term   := power (("*" | "/") power)*
power  := factor ("^" power)?            // right-associative
factor := ("+" | "-") factor | number | name | name "(" args? ")" | "(" expr ")"
args   := expr ("," expr)*
number := decimal, optional exponent (1e-3)
name   := [A-Za-z_][A-Za-z0-9_]*
```

**Names** resolve, in order, to: `pi` or `PI` (π); in a chart series, `x`; a parameter `id`; an earlier
calculation's `id`, or its `symbol` when the symbol is itself a valid name.

**Functions:** `sqrt pow hypot sin cos tan asin acos atan atan2 sinh cosh tanh exp log ln log10 abs min max
floor ceil round step mac`. `log` and `ln` are natural logarithms. `step(v)` is 1 when v ≥ 0, else 0 (Heaviside).
`mac(v, n)` is the Macaulay bracket ⟨v⟩ⁿ: 0 when v ≤ 0, else vⁿ (n defaults to 1).

**Errors.** For **sizes**, a viewer MUST still draw the part: a division by zero, an unknown name, a root of a
negative or a log of zero or less gives 0, and a size then falls back to the shape's minimum. For
**calculations**, the same cases make the step **broken**: shown as not workable (e.g. "—"), never as 0;
later steps read a broken step as 0.

## 5. model

| Field | | |
|---|---|---|
| `id` | MUST | identifier, unique within the file's origin |
| `title` | MUST | |
| `summary` | MUST | one or two sentences |
| `units` | MUST | `"mm"` or `"m"` (section 3) |
| `parameters` | MUST | list, may be empty (section 6) |
| `parts` | MUST | 1 or more (section 7) |
| `annotations`, `calculations`, `charts` | MAY | sections 9–11 |
| `question` | MAY | `{ text, find[], symbolic? }`: a problem being solved. Its givens are the parameters; its worked solution is the calculations, in order. `symbolic: true` means a derivation in symbols; parameter values then only size the drawing |

## 6. Parameters

`{ id, label, symbol?, value, min, max, step, unit }`. `value` is the starting value; a viewer MUST keep it
within `[min, max]`. `symbol` is display text (it may be LaTeX-like, `z_1`).

## 7. Parts

| Field | | |
|---|---|---|
| `id`, `name` | MUST | `id` unique within the file |
| `shapes` | MUST | 1–32 shapes (section 8), combined in order |
| `children` | MAY | parts placed in this part's frame; they move with it |
| `position`, `rotation` | MAY | Vec3Expr: metres; radians, Euler order XYZ. In the parent's frame |
| `mount` | MAY | placement by relation (section 7.2); wins over `position` |
| `motion` | MAY | section 7.3 |
| `material` | MAY | a material key (7.1); default `steel` |
| `role` | MAY | `housing` (cut-aways open it; x-ray makes it see-through), `moving`, `fluid`, `static` |
| `explode` | MAY | [x, y, z] metres the part moves outward in an exploded view |
| `cut` | MAY | `false` keeps the part whole in cut-aways |
| `info` | MAY | `{ summary, role? }`: what the part is and what it does |

### 7.1 Materials

`castIron steel steelMatte steelDark chrome forged aluminium bronze brass copper bearing rubber friction
gasket ceramic plastic wood paintGreen paintYellow paintRed paintGrey paintBlack paintBlue paintOrange
galvanised leather nylon concrete crop straw glass oil hydraulicOil coolant fuel water soil grain`. In 0.1
these are appearance keys. (Physical properties are planned for a later version.)

### 7.2 Mounts

A mount places a part relative to a **target** part (by `id`). Targets MUST be placed before the parts that
mount on them; cycles are invalid. `shift` (Vec3Expr, metres, world axes) moves the result further.
`AxisDir` is one of `+x -x +y -y +z -z`.

| `kind` | Fields | Placement |
|---|---|---|
| `coaxial` | `at` (0–1, default 0.5), `offset` | on the target's axis, `at` along its length (0 one end, 1 the other), `offset` metres further along; the part's axis aligned with the target's |
| `mesh` | `side` (AxisDir), `angle` (degrees) | a gear beside the target gear: axes parallel, centres the sum of the pitch radii (module × teeth / 2 each) apart, on `side` (default up, or +z when the axis is vertical), turned `angle` about the target's axis from there. It turns at the ratio of the teeth, in the opposite sense |
| `against` | `face` (AxisDir) | touching the target's `face`, centred on it (`+y`: sitting on top) |
| `centered` | | same centre as the target |
| `slider` | `throw`, `rod`, `angle`, `direction` | a piston or slider driven by the target crank: on the line from the crank centre along `direction` (default up), at r cos θ + √(l² − r² sin² θ), r = `throw`, l = `rod`, θ the crank angle (`angle` degrees from top dead centre, plus the crank's turning) |
| `link` | `to`, `throw`, `angle`, `direction` | a connecting rod from the target's crank pin to the centre of part `to`, its long side along that line |
| `lever` | `pivot`, `throw`, `angle`, `direction` | a slotted lever pivoting on part `pivot`, always pointing through the target's crank pin (quick-return) |
| `follower` | `direction`, `of` (`pin` or `tip`), `throw`, `angle`, `offset` | slides along `direction` with the target's crank pin (scotch yoke) or a lever's far end (shaper ram); fixed across that direction |
| `hooke` | `angle` (degrees) | the output shaft of a Hooke's joint at the end of input shaft `target`, its axis `angle` off the input's; it turns by tan θ₂ = tan θ₁ / cos β |

A part placed by a mount that transmits motion (`mesh`, `slider`, `link`, `lever`, `follower`, `hooke`, and
`coaxial` on a turning shaft) moves because its target moves; it SHOULD NOT also have its own `motion`.

### 7.3 Motion

`{ kind, axis, speed, amplitude?, phase? }` on a driving part. `axis` is `x`, `y` or `z` in the part's frame.

| `kind` | `speed` | `amplitude` | Movement at time t |
|---|---|---|---|
| `rotate` | rad/s | | angle = speed·t + phase, about the part's own axis through its centre |
| `oscillate` | Hz | radians (default 0.35) | angle = amplitude·sin(2π·speed·t + phase) |
| `translate` | Hz | metres | offset = amplitude·sin(2π·speed·t + phase) |

A viewer MAY play time slower than real time so motion is easy to follow, and SHOULD say so.

## 8. Shapes

Every shape has `kind`, and MAY have `op` (`add` default, `subtract`, `intersect`: applied to the shapes
before it in the same part, in order), `position` and `rotation` (in the part's frame), and a circular
pattern: `radialCount` copies spaced evenly round the part's Y axis at radius `radialRadius`, in the XZ plane.

| `kind` | Fields (defaults) | Definition |
|---|---|---|
| `box` | `sx sy sz` (0.08 each) | centred box |
| `cylinder` | `r`, or `rTop` and `rBot` (0.04); `h` (0.1) | about Y, centred; a frustum when the radii differ |
| `cone` | `rBot` or `r` (0.05), `rTop` (0.001), `h` (0.1) | about Y, centred |
| `tube` | `rOuter` or `r` (0.05), `rInner` (0.6 rOuter), `h` (0.1) | hollow cylinder about Y, centred |
| `sphere` | `r` (0.04) | centred |
| `torus` | `r` (0.05), `tubeRadius` (0.01) | ring about Y |
| `lathe` | `profile`: [[r, y], …] (≤ 64 points) | the profile turned a full circle about Y |
| `extrude` | `polygon`: [[x, y], …], `depth` (0.04) | the polygon in the XY plane, extruded along Z, centred in Z |
| `gear` | `teeth` (20; 6–120), `module` (metres, 0.004), `h` face width (0.024), `rInner` bore (teeth·module/6), `pressureAngle` (degrees, 20; 14.5–30), `helixAngle` (degrees, 0; −45–45) | involute spur or helical gear about Y; pitch radius = module·teeth/2 |
| `bevelGear` | as `gear` | in 0.1 drawn as `gear`; a true bevel form is planned |
| `rack` | `sx` length (0.24), `sy` (0.024), `sz` (0.024) | a toothed bar along X |
| `pulley` | `rOuter` (0.06), `rInner` bore (0.012), `h` (0.028) | V-grooved pulley about Y |
| `sprocket` | `teeth` (16; 8–80), `module` (0.004), `h` (0.01), `rInner` (teeth·module/5) | chain sprocket about Y |
| `spring` | `r` coil radius (0.025), `wireRadius` (0.0035), `h` (0.1), `turns` (6; 2–40) | helical coil spring about Y, centred |
| `hexNut` | `d` thread diameter (0.02), `h` (0.8 d) | hexagon nut about Y, centred |
| `hexBolt` | `d` (0.016), `h` shank length (5 d) | hexagon-head bolt about Y |

Sizes below a shape's minimum are raised to it (about 1–2 mm for most) so every part draws. A viewer meeting
an unknown `kind` SHOULD draw a small placeholder block rather than fail.

## 9. Annotations

`{ id, kind, label, … }`, `kind` one of `force`, `moment`, `dimension`, `label`. Prefer `anchor`:
`{ part, at?, along? }`, which keeps the annotation on its part wherever the assembly puts it (`at`: `center`,
a face `+y`…, or `start`/`end` of the part's axis; `along` 0–1 picks a point along the axis). A **force**
arrow ends on that point and acts along `direction` (default `-y`), `length` metres long; a **moment** turns
about the part's axis there; a **dimension** spans the part's length. Without an anchor, `from` and `to`
give the points directly. `magnitude` (DimExpr) is the live value shown with `unit`; `value` is fixed text.

## 10. Calculations

An ordered list of `{ id, label, symbol, formula, unit, latex?, description?, step?, answer?, limit? }`.
Each `formula` is evaluated in order, seeing the parameters and every earlier calculation (section 4). `latex`
is display only. In a worked solution, `step` says in one sentence what the step does, and `answer: true` marks
a result the question asks for. `limit: { kind: "max" | "min", value, label? }` flags the result when it is
over (or under) the limit.

## 11. Charts

`{ id, title, x: { label, unit, from, to }, y: { label, unit }, series: [{ id, label, formula }], samples? }`.
Each series formula is sampled at `samples + 1` points (default 120; 8–400) from `x.from` to `x.to`, with `x`
bound to the sample, plus parameters and calculations. At most 4 series per chart.

## 12. view

| Field | |
|---|---|
| `camera` | `{ position, target }`, metres |
| `extent` | the scene's size in metres, for framing and the floor grid |
| `floorY` | the floor's height |
| `cut` | `{ enabled, normal, pos }`: the cut-away it opens with (a plane through `pos` along `normal`) |
| `projectionPlanes` | `none`, `first-angle`, `third-angle` |

A viewer MAY frame the model itself instead (Solidify keeps the camera's direction and fits the distance to
the assembled model).

## 13. Extensions

`extensions` is an object whose keys are namespaced (`"vendor.feature"`, e.g. `"acme.fea"`). A reader MUST
ignore extensions it does not know, and SHOULD say that it did. Extensions MUST NOT change the meaning of core
fields.

## 14. Security

A file is data. Expressions are arithmetic only (section 4) and MUST NOT be run as code. Readers SHOULD limit
file size, list lengths and string lengths (the schema's limits), and SHOULD treat every text field as plain
text, never markup. A viewer opening files from links SHOULD only fetch over HTTPS.

## 15. Changes

- **0.1** (2026-11-10): first draft, from the model specification Solidify has used since 2026.
