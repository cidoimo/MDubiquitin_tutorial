## 📊 Analyses of MD Trajectories


Molecular Dynamics (MD) trajectories are data-rich, but analyzing them requires removing global motions to focus on internal dynamics. We will use the **Gromacs 2023** suite for these analyses. If you haven't installed it yet, you can do so via `sudo apt-get install -y gromacs`.

### Step 1: Retrieving & Fitting the Structure
Before calculating metrics like RMSD, we must "fit" the trajectory to remove rotational and translational movements. This ensures the data reflects the molecule's true conformational changes.

Use the `gmx trjconv` command to perform a progressive fit:
```bash
gmx trjconv -s yourfile_nowat.pdb -f your_trajectory.xtc -fit progressive -o your_trajectory_fitted.xtc
