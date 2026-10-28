# FLAC3D Model for Tunnel Excavation, Support Installation, and Dynamic Response

## Related manuscript

**Shaking Table Test on Thermal Insulation and Seismic Damping Performance of Open Air-Beam Grid Rubber Energy-Absorbing Layer in High Rock Temperature Tunnel**

Corresponding author: Weijin Liang.

Experimental data: [Figshare dataset](https://doi.org/10.6084/m9.figshare.34753389).

## Overview and scope

`code.txt` is a FLAC3D command file. It imports a pre-existing mesh, establishes gravity equilibrium, simulates staged excavation and support installation, and calculates the response to an imported acceleration history. Stress and displacement histories are exported as text files.

The supplied file uses built-in FLAC3D commands and constitutive models; it does not contain a standalone solver or custom FISH function definitions. A licensed FLAC3D installation is required. The proprietary FLAC3D software and solver source code are not distributed here.

**Thermal calculations are not executed by this version of the script.** The thermal configuration, thermal properties, temperature boundary conditions, thermal solution, and thermal-stress commands are commented out with semicolons. This file therefore documents the active mechanical/dynamic workflow, not a complete reproduction of the manuscript's thermal analyses or all experimental results.

## Software and hardware requirements

- **FLAC3D 7.0**, as specified by the author, with access to the dynamic-analysis capability.
- A computer supported by FLAC3D 7.0 with sufficient memory for the supplied mesh.
- The script requests **8 computational threads** with `program thread 8`.
- No external Python packages, MATLAB, Origin, Excel, PowerPoint, or ANSYS execution is invoked by this command file. Any upstream mesh-generation workflow is outside this file.
- Exact FLAC3D build, tested hardware, memory use, and wall-clock runtime have not yet been documented. The requested simulated dynamic duration of 14.98 is not the wall-clock runtime.

## Required repository files

Place these files in the same directory, preserving their names:

```text
repository/
|-- README.md
|-- code.txt
|-- Flacmodel_haitang.Flac3D
`-- yacc0.4g.txt
```

| File | Purpose |
| --- | --- |
| `code.txt` | Main FLAC3D command file and run entry point. |
| `Flacmodel_haitang.Flac3D` | Mesh imported by `zone import`; must contain the geometry and group assignments expected by the script. |
| `yacc0.4g.txt` | Acceleration-history table for the author-designated 0.4g input case, imported as table `1` for the dynamic boundary load. |

Both input files are essential: `code.txt` alone is not a runnable example. The author intends to distribute both files with the code. Their contents were not available when this README was prepared.

The mesh must support the script's selectors `Default=1` through `Default=6`. During construction, groups `1`–`4` are assigned the null model; group `4` is subsequently reactivated for initial support, group `3` for the elastic layer, and group `2` for the secondary lining. These roles follow the script and must be checked against the actual mesh.

`yacc0.4g.txt` must be readable by FLAC3D 7.0's `table import` command. The author identifies this as the 0.4g input case. Its values, time coordinate, units, and normalization must be documented by the author. **The filename alone does not establish the final applied peak acceleration:** the updated script still uses `acceleration-x 1.5 table '1'`. The input-table values and this loading factor must be checked together before describing the applied boundary motion as 0.4g.

## Calculation sequence

| Stage | Commands and intended operation | Saved model name in the script |
| --- | --- | --- |
| Initial equilibrium | Import mesh; assign Mohr–Coulomb properties; enable large strain; apply gravity and velocity constraints; solve to local ratio `1e-3`. | `zizhong.sav` |
| Excavation and initial support | Assign the null model to groups `1`–`4`; cycle 1,000 steps; reactivate group `4`; solve to local ratio `1e-4`. | `chuzhi.sav` |
| Elastic layer | Assign an elastic model and properties to group `3`; solve to local ratio `1e-3`. | `EVApaomian.sav` |
| Secondary lining | Reactivate group `2` with Mohr–Coulomb properties; solve to local ratio `1e-4`. | `erchen.sav` |
| Dynamic response | Configure dynamic analysis; reset velocities and displacements; apply boundary conditions and excitation; record histories; solve to total dynamic time `14.98`. | `earthquake` |

FLAC3D may add its default extension to a saved-model name without an explicit extension.

### Principal active settings

| Setting | Value in `code.txt` |
| --- | --- |
| Gravity vector | `(0, 0, -10)` |
| Initial Mohr–Coulomb properties for groups `1`–`6` | Bulk modulus `1.54e9`; shear modulus `0.428e9`; friction `25`; cohesion `1.5e5`; tensile strength `1e5`; density `1900` |
| Reactivated support/lining groups `4` and `2` | Same listed properties, with dilation `3` |
| Elastic layer, group `3` | Bulk modulus `1e6`; shear modulus `1e5`; density `75.2` |
| Excitation input | `yacc0.4g.txt`, imported as table `1` (author-designated 0.4g input case) |
| Dynamic loading | `acceleration-x 1.5 table '1'` at faces selected near `z = 0`; the loading factor remains `1.5` in the updated script |
| Boundary commands | Bottom quiet-normal, quiet-strike, and quiet-dip; dynamic free-field enabled |
| Dynamic damping | Local damping parameter `0.157` |
| Dynamic time step | Fixed at `1e-5` |
| History interval | Every `100` steps |
| Requested total dynamic time | `14.98` |

Use a consistent unit system. The code does not declare one explicitly. If the author confirms an SI system, the corresponding length, time, stress/modulus, density, and acceleration units are m, s, Pa, kg/m³, and m/s². Do not treat the input values as confirmed SI values without checking the model and excitation file.

Commented alternative material properties are not used. In particular, the `C30` comments and the `EVApaomian.sav` filename are labels in the source, not independent verification of the material identity or properties.

## How to run the supplied example

1. Download the repository and extract it into a new, writable run directory containing the three required files. Use a separate directory for each run.
2. Start FLAC3D 7.0. Preserve any other model before running: the script begins with `model new`.
3. Set the working directory to the extracted folder. For example, enter the following in the **FLAC3D console**, replacing the example path with your actual folder:

   ```text
   program directory custom 'D:/FLAC3D/tunnel_run'
   ```

4. Execute the command file from the FLAC3D console:

   ```text
   program call 'code.txt'
   ```

5. Monitor the console for import errors, invalid selections, convergence messages, and dynamic-analysis errors. A missing input file or incompatible mesh must be resolved before continuing.
6. After successful completion, inspect the saved models and exported histories in the working directory.

Do not run this file as a Python script or as a Windows shell command. Its `.txt` extension does not change the fact that it contains FLAC3D commands.

## Short dynamic test

The supplied model and excitation can be used as an example. For a shorter dynamic run, prepare a **separate test copy** as follows:

1. Copy the required files into a fresh test directory.
2. Copy `code.txt` to `code_quick_test.txt` in that directory.
3. In the test copy only, replace:

   ```text
   model solve time-total 14.98
   ```

   with:

   ```text
   model solve time-total 0.01
   ```

4. Set the FLAC3D working directory to the test directory and run:

   ```text
   program call 'code_quick_test.txt'
   ```

5. Check that the model and excitation load successfully, all static stages complete, the dynamic calculation reaches `0.01`, and the 32 requested history files are produced with finite numerical values.

**This short-test procedure has not been executed during README preparation.** It shortens only the dynamic stage; all mesh loading and static equilibrium stages still run, so it is not guaranteed to finish quickly. It is a basic execution check, not validation of the manuscript's full-duration results. The author should run the test, upload the resulting test source file, and add a checked reference output and measured runtime before submission.

## Output files

The script requests **24 stress-history files** and **8 displacement-history files**, in addition to saved model states.

| Files | Recorded quantity and sampling position |
| --- | --- |
| `30.txt`, `31.txt`, `32.txt` | `stress-xy`, `stress-xx`, `stress-yy` at `(0, 25, 28.65)` |
| `33.txt`, `34.txt`, `35.txt` | Same components at `(0, 25, 28.78)` |
| `40.txt`, `41.txt`, `42.txt` | Same components at `(6.78, 25, 22.56)` |
| `43.txt`, `44.txt`, `45.txt` | Same components at `(6.90, 25, 22.56)` |
| `50.txt`, `51.txt`, `52.txt` | Same components at `(0, 25, 17.13)` |
| `53.txt`, `54.txt`, `55.txt` | Same components at `(0, 25, 16.93)` |
| `60.txt`, `61.txt`, `62.txt` | Same components at `(-6.78, 25, 22.56)` |
| `63.txt`, `64.txt`, `65.txt` | Same components at `(-6.90, 25, 22.56)` |
| `230.txt`, `231.txt` | `displacement-x`, `displacement-z` at `(0, 25, 28.65)` |
| `240.txt`, `241.txt` | Same components at `(6.78, 25, 22.56)` |
| `250.txt`, `251.txt` | Same components at `(0, 25, 17.13)` |
| `260.txt`, `261.txt` | Same components at `(-6.78, 25, 22.56)` |

The script records total dynamic time internally but does not explicitly export that history or use it as the horizontal coordinate in the export commands. In FLAC3D 7.0, `history export` uses **step number by default**, not physical time. Check the output before assigning a time axis in downstream plots.

FLAC3D 7.0 also defaults to **appending** history exports to existing files. Running in a fresh directory prevents old and new history records from being combined. The script does not automatically generate manuscript figures or export temperatures.

## Reproducibility status and checks before journal submission

This README was prepared by inspecting `code.txt`; the model was not executed, and the mesh and excitation contents were not inspected. FLAC3D 7.0 was specified by the author. Successful execution, numerical accuracy, and reproduction of particular manuscript figures are not independently verified here.

Before submission, the author should:

- Include both required input files and the tested quick-test file as individual repository files.
- State the input/output units, excitation normalization, and meaning of the loading multiplier `1.5`.
- Check mesh groups, boundary selections, and monitoring positions. For example, the script contains a boundary selector near `y = 18` while histories sample at `y = 25`; the mesh is needed to establish whether this is intentional.
- Check the reversed endpoints in `position-x -17.99 -18.01` and verify the intended boundary selection in FLAC3D.
- Verify the simultaneous quiet-boundary and prescribed-acceleration setup, and the stability of the fixed dynamic time step for this mesh and its materials.
- Confirm that active material properties and the disabled thermal section match the analysis being reported.
- Document the exact FLAC3D build, tested hardware, runtime, reference output, and correspondence to manuscript figures/tables.
- Add a contact email and an appropriate license for author-owned code. No open-source license was supplied with the reviewed file; this README does not grant one or license the proprietary solver.

These checks distinguish preparation of a usable repository from scientific validation and journal acceptance.

## FLAC3D 7.0 command references

- [Executing a command file: program call](https://docs.itascacg.com/flac3d700/common/kernel/doc/manual/program/commands/cmd_program.call.html)
- [Setting the working directory: program directory](https://docs.itascacg.com/flac3d700/common/kernel/doc/manual/program/commands/cmd_program.directory.html)
- [Required table input format: table import](https://docs.itascacg.com/flac3d700/common/kernel/doc/manual/table_manual/table_commands/cmd_table.import.html)
- [History export coordinates and file behavior](https://docs.itascacg.com/flac3d700/common/kernel/doc/manual/history_manual/history_commands/cmd_history.export.html)
