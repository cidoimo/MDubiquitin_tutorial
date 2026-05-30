## 📊 Analyses of MD Trajectories
![Gromacs2025](https://img.shields.io/badge/Gromacs-2025-yellow)

Molecular Dynamics (MD) trajectories are data-rich, but analyzing them requires removing global motions to focus on internal dynamics. We will use the **Gromacs 2023** suite for these analyses. If you haven't installed it yet, you can do so via `sudo apt-get install -y gromacs`. For any kind of issues, please refer to the ![Gromacs 2025 Manual](https://manual.gromacs.org/documentation/2025.0/manual-2025.0.pdf).

### Step 1: Retrieving & Fitting the Structure/Trajectory
First of all, you need the first frame of the trajectory. This structure, commonly known as frame0, we'll be the topology of the system you're gonna use fo almost all of your analysis. You must extract and center in the origin of the axis (X, Y, and Z) the frame0 of the MD trajectory. Centering is a must-have of trajectory analysis because it removes the periodic boundary condition (PBC) artifacts and ensures that the protein remains within the viewing window during the visualization. Without this step, your protein might appear to 'jump' across the box or even disappear from the screen due to wrapping effects.

```bash
# Extract
gmx trjconv -s yourfile_nowat.pdb -f your_trajectory.xtc -dump 0 -o frame0.pdb
# Center
gmx editconf -f frame0.pdb -center 0 0 0 -o frame0_centered.pdb
```

In the first command, the flag *-dump* refers to the dumping frame you want to save in the output file. You can choose whethever frame you want, yes, but for the first you must insert 0.

Well, now. Before calculating metrics like RMSD, we must "fit" the trajectory to remove rotational and translational movements. This ensures the data reflects the molecule's true conformational changes. Watch thes two videos, are they different?

<div align="center">
  
| Unfitted Trajectory | Fitted Trajectory |
| :---: | :---: |
| <img src="https://github.com/cidoimo/MDubiquitin_tutorial/raw/main/Analysis/nofit.gif" width="300"> | <img src="https://github.com/cidoimo/MDubiquitin_tutorial/raw/main/Analysis/fit.gif" width="300"> |

</div>

Watching this you can easily understand why it's important to fit the trajectory BEFORE doing the analysis. We don't want any kind of false results, no?

Usually I fit the protein structure onto the carbon alpha atoms. Use the `gmx trjconv` command to perform a progressive fit:
```bash
gmx trjconv -s frame0_centered.pdb -f your_trajectory.xtc -fit progressive -o your_trajectory_fittedCA.xtc
```
with the prompt command, when you'll launch the command:
* Fitting index: C-alpha (group 3)
* Saving index: System (group 0)

You can either write C-alpha/System or simply type 3/0 in the interactive gmx menu.



### Why choose "progressive" fitting?
In GROMACS, the `gmx trjconv` command offers several fitting strategies (`-fit`), such as `rot+trans`, `rotxy+transxy`, and `progressive`. While each serves a specific purpose, the **progressive fit** is the standard choice for visualizing long Molecular Dynamics (MD) trajectories.

Here is why:

1. **How it works:** Unlike a standard fit (where every frame is superimposed onto a single reference structure, usually frame 0), a **progressive fit** superimposes frame *i+1* onto the already fitted frame *i*. This chain-like process is repeated throughout the entire trajectory.
2. **Standard vs. Progressive:** If the protein undergoes significant conformational changes, fitting everything to a single starting structure can create distortions or "jerky" movements in the visualization. The progressive fit follows the protein's gradual movement, minimizing abrupt rotations between consecutive frames.
3. **The Result:** This method produces a much smoother trajectory, effectively isolating the internal structural fluctuations (like loop motions or helix breathing) from the global tumbling of the molecule in the box.

While other fitting methods are useful for rigorous statistical analysis against a fixed crystal structure, the **progressive fit** is the "gold standard" for visualization. It ensures your protein doesn't appear to spin wildly in the box, allowing you to focus on its true internal dynamics.

---
### Step 2a: Structural Stability & Flexibility (RMSD & RMSF)
RMSD (Root Mean Square Deviation) is a measure used to quantify the difference between two three- dimensional structures of a molecule. It is calculated as the square root of the average of the squared distances between corresponding atoms in the two structures. RMSD is often used for evaluating structural stability, comparing conformations or studying conformational transitions. Usually a typical output should have on the x-axis the simulation time. On the y-axis, there is the RMSD value expressed in nm.
Typically, the curve shows a rapid increase in the early part of the simulation, then stabilizes around a value and oscillates around it. This means that the simulation took the initial part of the time to complete equilibration, then reached a stable phase called a plateau or convergence.
For the purposes of this tutorial, the RMSD will be calculated on the entire protein without the hydrogens. To calculate the RMSD, use the command:

```bash
gmx rms -s frame0_centered.pdb -f your_trajectory_fittedCA.xtc -tu ns -o rmsd.xvg
```
Flags breakdown:
* -s: Specifies the reference structure (typically your prepared .pdb or .tpr file).
* -f: Specifies the input trajectory (use the fitted trajectory for meaningful results).
* -tu ns: Instructs GROMACS to convert time units from picoseconds to nanoseconds.
* -o: Defines the name of the output .xvg file.

Once the command is executed, you will be prompted to select groups for fitting and analysis. Select Protein-H (type 2) as the reference group, and Protein-H (type 2) again as the group to calculate the RMSD.

You can visualize the resulting rmsd.xvg file using xmgrace/python/R. For simplicity, here you're gonna see the results file plotted with xmgrace (yeah, I'm a bit melancolhol because it's the first program I used for representing things during my master's thesis). To open a file with xmgrace, simply ```xmgrace rmsd.xvg```

---

RMSF (Root Mean Square Fluctuation) is a measure used to quantify the flexibility or mobility of each atom in a molecule throughout a MD simulation. It is calculated as the square root of the average of the squared fluctuations of each atom's position relative to its average position in the trajectory.
RMSF is often used for identifying flexible regions in proteins, such as loop regions or protein-ligand binding sites. The N and C-terms are a lot flexible, as shown in figure for N-terminal part.
A typical output plot of RMSF shows on the x-axis the residue index or number, and on the y-axis, the RMSF value expressed in nm.
For the purposes of this tutorial, RMSF will be calculated for each residue of the protein without the hydrogens. To calculate RMSF, use the command:

```bash
gmx rmsf -s frame0_centered.pdb -f your_trajectory_fittedCA.xtc -res -o rmsf.xvg
```
Flags breakdown:
* -s: Specifies the reference structure (typically your prepared .pdb or .tpr file).
* -f: Specifies the input trajectory (use the fitted trajectory for meaningful results).
* -res: Print the residues of the protein in a column and the RMSF value in the other.
* -o: Defines the name of the output .xvg file.

<div align="center">
  
| RMSD | RMSF |
| :---: | :---: |
| <img src="https://github.com/cidoimo/MDubiquitin_tutorial/blob/main/Analysis/images/rmsd.png" width="500"> | <img src="https://github.com/cidoimo/MDubiquitin_tutorial/blob/main/Analysis/images/rmsf.png" width="500"> |

</div>
