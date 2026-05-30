## 📊 Analyses of MD Trajectories
![Gromacs2025](https://img.shields.io/badge/Gromacs-2025-yellow)

Molecular Dynamics (MD) trajectories are data-rich, but analyzing them requires removing global motions to focus on internal dynamics. We will use the **Gromacs 2023** suite for these analyses. If you haven't installed it yet, you can do so via `sudo apt-get install -y gromacs`. For any kind of issues, please refer to the ![Gromacs 2025 Manual](https://manual.gromacs.org/documentation/2025.0/manual-2025.0.pdf).

### Step 1: Retrieving & Fitting the Structure
First of all, you need the first frame of the trajectory. This structure, commonly known as frame0, we'll be the topology of the system you're gonna use fo almost all of your analysis. You must extract and center in the origin of the axis (X, Y, and Z) the frame0 of the MD trajectory. Centering is a must-have of trajectory analysis because it removes the periodic boundary condition (PBC) artifacts and ensures that the protein remains within the viewing window during the visualization. Without this step, your protein might appear to 'jump' across the box or even disappear from the screen due to wrapping effects.

```bash
# Extract
gmx trjconv -s yourfile_nowat.pdb -f your_trajectory.xtc -dump 0 -o frame0.pdb
# Center
gmx editconf -f frame0.pdb -center 0 0 0 -o frame0_centered.pdb
```

In the first command, the flag *-dump* refers to the dumping frame you want to save in the output file. You can choose whethever frame you want, yes, but for the first you must insert 0.

Well, now. Before calculating metrics like RMSD, we must "fit" the trajectory to remove rotational and translational movements. This ensures the data reflects the molecule's true conformational changes. Watch thes two videos, are they different?

| Unfitted Trajectory | Fitted Trajectory |
| :---: | :---: |
| <img src="https://github.com/cidoimo/MDubiquitin_tutorial/raw/main/Analysis/nofit.gif" width="300"> | <img src="https://github.com/cidoimo/MDubiquitin_tutorial/raw/main/Analysis/fit.gif" width="300"> |

Watching this you can easily understand why it's important to fit the trajectory BEFORE doing the analysis. We don't want false results, no?



Use the `gmx trjconv` command to perform a progressive fit:
```bash
gmx trjconv -s yourfile_nowat.pdb -f your_trajectory.xtc -fit progressive -o your_trajectory_fitted.xtc
