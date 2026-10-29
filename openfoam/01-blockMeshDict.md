# Lesson 1 — Understand blockMeshDict

**Target:** OpenFOAM Foundation v14 on Ubuntu/WSL.
**Goal:** create a rectangular mesh and explain its dimensions, bounding box, cell count, and cell size.

This is a mesh-only exercise. It does not solve a flow problem.
For general Ubuntu and Git commands, reuse [learning.txt](../learning.txt) and [ubuntu_command.txt](../ubuntu_command.txt).

## 1. What the file does

`blockMesh` is a program. `system/blockMeshDict` is its input dictionary.
The dictionary describes corner points, how they connect into blocks, the number of cells, and named boundary patches.
The generated mesh is written into `constant/polyMesh`.

A block is a region to subdivide; it is not one computational cell.
Here, one block will contain 400 cells.

## 2. Prepare an isolated exercise

Open an Ubuntu terminal with your OpenFOAM environment already loaded:

```bash
foamVersion
command -v blockMesh
```

If either command is unavailable, load your installed OpenFOAM environment before continuing. Do not reinstall OpenFOAM for a missing environment setting.

Choose a new directory that does not contain an existing simulation:

```bash
mkdir -p ~/OpenFOAM/learning/blockMesh-lesson-01
cd ~/OpenFOAM/learning/blockMesh-lesson-01
mkdir -p system constant
nano system/blockMeshDict
```

## 3. Complete system/blockMeshDict

Paste the following file. Coordinates are in metres because `scale` is 1.

```foam
FoamFile
{
    version     2.0;
    format      ascii;
    class       dictionary;
    object      blockMeshDict;
}

scale 1;

vertices
(
    (0    0    0)       // 0
    (0.10 0    0)       // 1
    (0.10 0.05 0)       // 2
    (0    0.05 0)       // 3
    (0    0    0.01)    // 4
    (0.10 0    0.01)    // 5
    (0.10 0.05 0.01)    // 6
    (0    0.05 0.01)    // 7
);

blocks
(
    hex (0 1 2 3 4 5 6 7) (20 10 2) simpleGrading (1 1 1)
);

edges
(
);

boundary
(
    inlet
    {
        type patch;
        faces ((0 4 7 3));
    }
    outlet
    {
        type patch;
        faces ((1 2 6 5));
    }
    walls
    {
        type wall;
        faces
        (
            (0 1 5 4)
            (3 7 6 2)
            (0 3 2 1)
            (4 5 6 7)
        );
    }
);

mergePatchPairs
(
);
```

Save with Ctrl+O, Enter, then exit with Ctrl+X.

## 4. Read the block definition

```foam
hex (0 1 2 3 4 5 6 7) (20 10 2) simpleGrading (1 1 1)
```

- `hex`: a hexahedral block.
- The eight labels refer to the ordered entries in `vertices`; numbering starts at zero.
- For this axis-aligned example, local block directions follow global x, y, and z: 0→1, 1→2, and 0→4.
- `(20 10 2)`: 20 cells along x, 10 along y, and 2 along z.
- `simpleGrading (1 1 1)`: equal cell widths within each direction. A grading value is an end-cell/start-cell width ratio, not a cell count.

Keep the vertex ordering: changing it casually can create an inverted block.
Patch types describe the mesh boundary. Velocity and pressure boundary conditions belong in field files and will be covered later.

## 5. Bounding box and cell sizes

The bounding box reports the minimum and maximum mesh coordinates:

```text
(0 0 0) (0.1 0.05 0.01)
```

Subtract minimum from maximum in each direction:

| Direction | Domain length | Cells | Uniform cell width |
|---|---:|---:|---:|
| x | 0.10 m = 100 mm | 20 | 0.005 m = 5 mm |
| y | 0.05 m = 50 mm | 10 | 0.005 m = 5 mm |
| z | 0.01 m = 10 mm | 2 | 0.005 m = 5 mm |

For this uniform rectangular block:

```text
Δx = Lx/Nx
Δy = Ly/Ny
Δz = Lz/Nz
Number of cells = Nx × Ny × Nz = 20 × 10 × 2 = 400
Cell volume = Δx × Δy × Δz = 1.25 × 10^-7 m³
Domain volume = 0.10 × 0.05 × 0.01 = 5 × 10^-5 m³
```

A message about “Block 0 cell size” refers to the first block, because block numbering also starts at zero. Here every cell is 5 mm wide in all three directions. With grading, sizes can vary from one end to the other; length/cell-count then gives an average rather than each individual width.

A bounding box is an enclosing coordinate range. For a curved or multi-block mesh it does not by itself describe the complete geometry.

## 6. Add a minimal system/controlDict

Create `system/controlDict` with:

```foam
FoamFile
{
    version     2.0;
    format      ascii;
    class       dictionary;
    object      controlDict;
}

application     blockMesh;
startFrom       startTime;
startTime       0;
stopAt          endTime;
endTime         1;
deltaT          1;
writeControl    timeStep;
writeInterval   1;
runTimeModifiable true;
```

This supplies basic case time controls for the utilities. It does not select or configure a flow simulation.

## 7. Generate and check

Run from the exercise directory containing `system` and `constant`:

```bash
blockMesh > log.blockMesh 2>&1
tail -n 30 log.blockMesh
```

Continue only if mesh generation completed without a fatal error:

```bash
checkMesh > log.checkMesh 2>&1
tail -n 40 log.checkMesh
```

Expected checks, calculated from the dictionary:

- Bounding box: `(0 0 0) (0.1 0.05 0.01)`.
- Cells: 400 hexahedra.
- Points: (20+1)(10+1)(2+1) = 693.
- Boundary patches: inlet, outlet, walls.
- Total mesh volume: 0.00005 m³.
- The mesh check should report `Mesh OK`.

These are expected results, not a recorded run. Console formatting can differ between versions.
Do not start a solver yet: no initial fields or physical properties have been supplied.

## 8. Change resolution without changing geometry

Change only `(20 10 2)` to `(40 20 4)`, then run the two utilities again.

Before running, predict:

- The bounding box stays the same.
- Cell widths become 2.5 mm in x, y, and z.
- Total cells become 40 × 20 × 4 = 3200: eight times the original.

Compare the new logs with your prediction. Restore `(20 10 2)` if you want the original mesh.

## 9. Common mistakes

| Symptom | Check |
|---|---|
| `blockMesh: command not found` | OpenFOAM environment is loaded in this terminal. |
| Cannot find `system/controlDict` | Run from the case directory and create the file in section 6. |
| Cannot find `blockMeshDict` | Exact case-sensitive name and location: `system/blockMeshDict`. |
| Unexpected domain size | Coordinate values and the scaling factor. |
| Inverted block or negative volume | Vertex ordering and outward-facing boundary face ordering. |
| Different cells than expected | Cell counts in `blocks`, not the number of vertices. |

This example is three-dimensional with two cells through z. A two-dimensional setup needs a deliberate one-cell-thick mesh and appropriate `empty` patches and field boundary conditions.

## 10. Check your understanding

1. Does increasing cell count make the physical domain larger? **No.**
2. Does a bounding box specify mesh resolution? **No.**
3. What changes if only Nx becomes 40? **Δx becomes 2.5 mm, total cells become 800, and the other widths remain 5 mm.**
4. What does `simpleGrading (1 1 1)` mean? **Uniform widths within each block direction.**

## Reference

[OpenFOAM Foundation v14 User Guide: blockMesh](https://doc.cfd.direct/openfoam/user-guide-v14/blockmesh).
Use this version's syntax; other OpenFOAM distributions and older tutorials can differ.
