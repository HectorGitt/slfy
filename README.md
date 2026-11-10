# .slfy: an open format for engineering models that move and calculate

A `.slfy` file is one JSON document describing a 3D engineering model **and its engineering**:

- **parameters** with ranges and units, shown as sliders
- **parts** built from shapes (gears, pulleys, springs, bolts, lathe profiles, extrusions…) with boolean cuts,
  where any size can be a formula of the parameters
- parts **placed by relation**: on a shaft, meshing with a gear, driven by a crank, through a Hooke's joint
- **motion**, **forces, moments and dimensions** attached to parts
- **calculations** with their equations, worked in order, with limits and answers
- **charts** sampled from formulas (bending moment diagrams, pump curves)
- the **question** a model answers, for worked problems

glTF carries meshes; STEP carries exact CAD; neither carries the mechanism or the working. `.slfy` does.

Status: **draft 0.1** (November 2026). Expect changes before 1.0. Website: <https://slfy.solidify.studio>.

| | |
|---|---|
| Extension | `.slfy` (readers should also accept `.solidify.json`) |
| Media type | `application/vnd.solidify+json` (not yet registered with IANA) |
| Encoding | UTF-8 JSON |
| Schema | [`schema/slfy-0.1.schema.json`](schema/slfy-0.1.schema.json) (JSON Schema 2020-12), at `https://slfy.solidify.studio/schema/slfy-0.1.schema.json` |
| Specification | [`SPEC.md`](SPEC.md) |
| Examples | [`examples/`](examples/) |

## The smallest file

```json
{
  "format": "slfy",
  "version": "0.1",
  "model": {
    "id": "minimal-block",
    "title": "A block on a plate",
    "summary": "Two parts, one placed against the other.",
    "units": "mm",
    "parameters": [],
    "parts": [
      { "id": "plate", "name": "Base plate", "shapes": [{ "kind": "box", "sx": 0.3, "sy": 0.02, "sz": 0.2 }] },
      { "id": "block", "name": "Block", "mount": { "kind": "against", "target": "plate", "face": "+y" },
        "shapes": [{ "kind": "box", "sx": 0.1, "sy": 0.08, "sz": 0.1 }] }
    ]
  }
}
```

## Examples

| File | Shows |
|---|---|
| [`minimal-block.slfy`](examples/minimal-block.slfy) | the least a file needs; placing by relation |
| [`shaft-in-torsion.slfy`](examples/shaft-in-torsion.slfy) | parameters, formulas as sizes, motion, calculations with a limit, a chart |
| [`spur-gear-pair.slfy`](examples/spur-gear-pair.slfy) | gears, `mesh` mounts, a driven gear turning from its driver |
| [`beam-question.slfy`](examples/beam-question.slfy) | a worked question: givens, steps, answers, loads and reactions, a bending moment diagram |
| [`helical-gear-pair.slfy`](examples/helical-gear-pair.slfy), [`worm-reducer.slfy`](examples/worm-reducer.slfy), [`open-differential.slfy`](examples/open-differential.slfy), [`scotch-yoke.slfy`](examples/scotch-yoke.slfy), [`quick-return-mechanism.slfy`](examples/quick-return-mechanism.slfy), [`muff-coupling.slfy`](examples/muff-coupling.slfy) | models from the Solidify library, reviewed by its team |

Solidify opens them under Create → Open a model file (`https://solidify.studio/#/create/open`, arriving with
its next release), and a link opens one straight in: `https://solidify.studio/#/create/open?url=<https link to
the file>`. The example files on this site work as such links.

## Checking a file

Any JSON Schema 2020-12 validator works, for example:

```
npx ajv-cli@5 validate --spec=draft2020 --strict=false -s schema/slfy-0.1.schema.json -d model.json
```

(Some tools pick a parser by extension; copy a `.slfy` to `.json` for them.) The schema checks structure; the
specification also says how files are drawn and worked out, which a schema can't.

## Licences

The specification, schema and examples: CC BY 4.0 (`LICENSE-SPEC`). Code (later: the reference reader and
viewer): Apache 2.0 (`LICENSE`). "Solidify" is the name of the product that created the format; the
format itself is free for anyone to read and write.

## Maintained by

Solidify (<https://solidify.studio>). The schema and examples are generated from the schemas Solidify itself
reads and writes models with, so they stay in step with the app. Issues and suggestions are welcome.
