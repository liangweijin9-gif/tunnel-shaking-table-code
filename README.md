# FLAC3D Model for Tunnel Excavation, Thermal Analysis, and Dynamic Response

## Related manuscript

**Shaking Table Test on Thermal Insulation and Seismic Damping Performance of Open Air-Beam Grid Rubber Energy-Absorbing Layer in High Rock Temperature Tunnel**

Corresponding author: Weijin Liang.

Experimental data: [Figshare dataset](https://doi.org/10.6084/m9.figshare.34753389).

## Overview and scope

`code.txt` is a FLAC3D command file. It imports a pre-existing mesh, establishes gravity equilibrium, simulates staged excavation and support installation, and calculates the response to an imported acceleration history. Stress and displacement histories are exported as text files.

The supplied file uses built-in FLAC3D commands and constitutive models; it does not contain a standalone solver or custom FISH function definitions. A licensed FLAC3D installation is required. The proprietary FLAC3D software and solver source code are not distributed here.

The author confirms that testing of the model, thermal calculations, thermal–mechanical analysis, and dynamic calculations has been completed successfully, and that the tested code covers the relevant numerical calculations presented in the manuscript. Experimental measurements are available separately through the Figshare link above.

**Configuration note:** In the accompanying local `code.txt`, the thermal configuration, properties, boundary conditions, thermal solution, and thermal-stress commands are commented out. Its default execution therefore runs the mechanical/dynamic workflow. Reproducing the completed thermal tests requires the corresponding author-tested thermal configuration; the default command file does not automatically execute those commented commands.

## Software and hardware requirements

- **FLAC3D 7.0**, with the capabilities required for the mechanical, thermal, and dynamic analyses. The author confirms that testing was completed in this software environment.
- A computer supported by FLAC3D 7.0 with sufficient memory for the supplied mesh.
- The script requests **8 computational threads** with `program thread 8`.
- No external Python packages, MATLAB, Origin, Excel, PowerPoint, or ANSYS execution is invoked by this command file. Any upstream mesh-generation workflow is outside this file.
- Testing has been completed by the author. Hardware specifications and measured wall-clock timings are not included in this README. The simulated dynamic duration of 14.98 is a model setting, not a measured execution time.

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

Both input files are essential to the example. The author confirms that the model and excitation files were used in the completed tests.

**Input filename:** The accompanying local folder contains an excitation file named `0.4g 40hz.txt`, whereas `code.txt` imports `yacc0.4g.txt`. For the run directory shown above, copy the supplied excitation file under the name `yacc0.4g.txt`. Alternatively, retain `0.4g 40hz.txt` and change the import command to `table '1' import '0.4g 40hz.txt'`. Use one consistent filename in the run directory and script.

The mesh must support the script's selectors `Default=1` through `Default=6`. During construction, groups `1`–`4` are assigned the null model; group `4` is subsequently reactivated for initial support, group `3` for the elastic layer, and group `2` for the secondary lining. The author confirms that the supplied model was tested successfully with the script. These group roles describe the accompanying command file.

`yacc0.4g.txt` supplies the acceleration history for the author-designated 0.4g case. The author confirms that the excitation-input test has been completed successfully. The command file applies the table using `acceleration-x 1.5 table '1'`; the applied motion depends on both the table values and this multiplier. Preserve the input values and scaling of the tested configuration.

## Calculation sequence

| Stage | Commands and intended operation | Saved model name in the script |
| --- | --- | --- |
| Initial equilibrium | Import mesh; assign Mohr–Coulomb properties; enable large strain; apply gravity and velocity constraints; solve to local ratio `1e-3`. | `zizhong.sav` |
| Excavation and initial support | Assign the null model to groups `1`–`4`; cycle 1,000 steps; reactivate group `4`; solve to local ratio `1e-4`. | `chuzhi.sav` |
| Elastic layer | Assign an elastic model and properties to group `3`; solve to local ratio `1e-3`. | `xiangjiao.sav` |
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

Use the same consistent unit system as the tested model and excitation file. This README does not assign additional units or normalization conventions beyond those of the supplied calculation setup.

Commented alternative material properties are not used. In particular, the `C30` comments and the `xiangjiao.sav` filename are labels in the source, not independent verification of the material identity or properties.

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

## Completed tests and short dynamic example

The author confirms that all tests have been completed successfully, including model loading, staged mechanical calculations, thermal calculations, thermal–mechanical analysis, dynamic response, and the test/example workflow. The author also confirms that the tested code covers the relevant calculations presented in the manuscript. This completion status is based on the author’s execution and verification.

The following instructions describe how readers can repeat a short dynamic example; they are not a list of outstanding author tests. For a shorter dynamic run, prepare a **separate test copy** as follows:

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

FLAC3D 7.0 also defaults to **appending** history exports to existing files. Running in a fresh directory prevents old and new history records from being combined. The default mechanical/dynamic command file exports the histories listed here. Thermal results belong to the author-tested thermal configuration described above. Manuscript figures are prepared through post-processing.



## FLAC3D 7.0 command references

- [Executing a command file: program call](https://docs.itascacg.com/flac3d700/common/kernel/doc/manual/program/commands/cmd_program.call.html)
- [Setting the working directory: program directory](https://docs.itascacg.com/flac3d700/common/kernel/doc/manual/program/commands/cmd_program.directory.html)
- [Required table input format: table import](https://docs.itascacg.com/flac3d700/common/kernel/doc/manual/table_manual/table_commands/cmd_table.import.html)
- [History export coordinates and file behavior](https://docs.itascacg.com/flac3d700/common/kernel/
