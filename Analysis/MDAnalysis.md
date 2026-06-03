## 📊 Analyses of MD Trajectories
![Gromacs2025](https://img.shields.io/badge/Gromacs-2025-yellow)
![VMD](https://img.shields.io/badge/VDM-1.9.3-green)

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
| <img src="https://github.com/cidoimo/MDubiquitin_tutorial/blob/main/Analysis/images/nofit.gif" width="300"> | <img src="https://github.com/cidoimo/MDubiquitin_tutorial/blob/main/Analysis/images/fit.gif" width="300"> |

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

You can visualize the resulting rmsd.xvg file using xmgrace/python/R. For simplicity, here you're gonna see the results file plotted with xmgrace (yeah, I'm a bit melancholic because it's the first program I used for representing things during my master's thesis). To open a file with xmgrace, simply ```xmgrace rmsd.xvg```

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

---
### Step 2b: Structural Compactness (Rg)
The Radius of Gyration (Rg) is a fundamental physical metric used to evaluate the overall compactness and global structural integrity of a protein. It is defined as the mass-weighted root mean square distance of a collection of atoms from their common center of mass.

* Global Compactness: A stable Rg value over the course of an MD trajectory indicates that the protein maintains its globular fold and defined shape.
* Structural Integrity: By monitoring the Rg, you can detect signs of structural unfolding. If the Rg exhibits a sudden or sustained increase, it suggests that the protein is losing its native compact state and becoming more expanded or disordered within the solvent.
* Equilibrium Indicator: Along with RMSD, Rg is a standard diagnostic tool to confirm that your simulation has reached a stable equilibrium state.

```bash
gmx gyrate -s frame0_centered.pdb -f your_trajectory_fittedCA.xtc -o rgyr.xvg
```

<div align="center">

| Radius of Gyration (Rg) |
| :---: |
| <img src="https://github.com/cidoimo/MDubiquitin_tutorial/blob/main/Analysis/images/Rgyr.png" width="500"> |

</div>

---

### BONUS step: Skipping the MD trajectory
When working with MD trajectories, you will often find that the resulting files are massive, frequently spanning several gigabytes or even terabytes. Processing every single frame is often computationally expensive and, more importantly, statistically redundant.
Why Skip Frames?
* Reduced Computational Cost: Calculating geometric parameters for every frame requires significant CPU/GPU resources. By analyzing a subset of frames, you can drastically reduce processing time without losing biological meaning.
* Mitigating Correlations: MD simulations often produce highly correlated snapshots. If your sampling frequency is faster than the physical process you are studying, consecutive frames provide little new information.
* Memory Management: Loading an entire multi-gigabyte trajectory into memory can crash your analysis environment. Skipping frames allows you to downsample the trajectory to a manageable size.

For this tutorial, you'll skip the MD trajectory every 50 frames with the *-skip* flag:

```bash
gmx trjconv -s frame0_centered.pdb -f your_trajectory_fittedCA.xtc -skip 50 -o your_trajectory_skipped50_fittedCA.xtc
```

---

### Step 3: Principal Component Analysis
Principal Component Analysis (PCA), also known as Essential Dynamics (ED) within the context of molecular dynamics, is a linear statistical technique used to reduce data dimensionality. It allows for the isolation of concerted, large-scale macromolecular motions (the "functional" movements) from local, random thermal noise.
PCA is based on the diagonalization of the covariance matrix of atomic fluctuations. The resulting vectors (eigenvectors) define the direction of the motions, while their associated values (eigenvalues) indicate the amplitude of the fluctuations along that specific direction.

Sampling the essential space in GROMACS is a two-step process using the ```gmx covar``` and ```gmx anaeig``` utilities.

#### A: Calculation and Diagonalization of the Covariance Matrix (gmx covar)
The ```gmx covar``` command first removes global translational and rotational motions of the protein (via a least-squares fit to a reference structure) and subsequently calculates and diagonalizes the covariance matrix for the selected atoms (typically alpha carbons, C-alpha).

```bash
gmx covar -s frame0_centered.pdb -f your_trajectory_skip50_fittedCA.xtc -o eigenval.xvg -v eigenvec.trr -av average.pdb -ascii covariance_matrix.dat -l covar.log -tu ns
```

Key Flags Explained:
* -s: Structure or topology file (.tpr) used as a reference for the least-squares fit.
* -f: Input trajectory (it is highly recommended to use a trajectory with periodic boundary conditions removed and already rototranslationally fitted).
* -o: Output file containing the eigenvalues in descending order. This helps determine how much of the total variance is explained by each component.
* -v: Output file containing the eigenvectors (the directions of motion).
* -av: Saves the average structure of the protein during the trajectory, which is useful for subsequent analyses.
* -ascii: Saves the covariance matrix in a file, useful if you want to plot DCCMs (dynamical cross correlation matrices).
* -l: Dump a log file of the command.
* -tu ns: Use the nanoseconds as a timescale for the analysis.

💡 Analysis Tip: When prompted by the terminal, select the C-alpha group for both the least-squares fit and the covariance matrix calculation.


#### B: Analysis and Projection of Motions (gmx anaeig)
Once you have obtained the eigenvectors and eigenvalues, you use ```gmx anaeig``` to analyze the trajectory by projecting it onto the principal components (usually the first two or three, which describe the vast majority of the essential dynamics). The 2D projection plot (PC1 vs PC2) generated from the proj.xvg file serves as a true map of the protein's conformational landscape. The distribution and spread of the data points across this two-dimensional space provide direct insights into the structural stability of the biological system.
When the plot shows a wide dispersion of points, spreading chaotically across a large area without clear boundaries, it signifies that the protein is highly flexible and unstable. In this scenario, the macromolecola is continuously sampling a vast array of different conformations due to a flat energy landscape. This behavior is typical for intrinsically disordered proteins, proteins undergoing denaturation, or specific mutants that compromise structural integrity.
Conversely, when the data points concentrate into tight, highly localized clusters with minimal spread, the protein is structurally stable. This indicates that the molecule is trapped within a deep potential energy well, fluctuating minimally around a well-defined native conformation. In cases where multiple dense, distinct clusters separated by empty regions are observed, the protein is undergoing transitions between different metastable states, which is the underlying mechanism for activation or conformational switching.

```bash
gmx anaeig -s frame0_centered.pdb -f your_trajectory_skip50_fittedCA.xtc -v eigenvec.trr -eig eigenval.xvg -2d 2dproj.xvg -tu ns -first 1 -second 2
```

Key Flags Explained:
* -v: Specifies the input eigenvectors file generated in the previous step.
* -eig: Reads the eigenvalue file to correctly scale or verify the components.
* -2d: Output file for the projections of the trajectory along the selected eigenvectors over time (.xvg).
* -first 1 -last 2: Restricts the analysis and projection to the first 2 principal components (PC1, PC2), which typically describe the vast majority of the system's total variance.
* -tu ns: Ensures time coordinates in the projection file are written in nanoseconds.

So, now plot the 2d projection using xmgrace:

```bash
xmgrace 2dproj.xvg
```

<div align="center">
  
| 2dproj (before) A | 2dproj (after) B |
| :---: | :---: |
| <img src="https://github.com/cidoimo/MDubiquitin_tutorial/blob/main/Analysis/images/2dproj_before.png" width="500"> | <img src="https://github.com/cidoimo/MDubiquitin_tutorial/blob/main/Analysis/images/2dproj.png" width="500"> |

</div>


You're gonna retrieve something like this (A). Pretty messy, no? Let's change a bit the representation!
Double click on a whetever point of the plot. Then change:
* Symbol properties --> type Circle
* Symbol properties --> Size 33
* Line properties --> None

Now double click on the X-axis or Y-axis and change the limits for both axis to (-3, 3). Adjust the tick properties (major spacing) if needed to 1.
A little bit better, no? (see B) Every point is a frame analyzed with the PCA. 


---

### Step 4a: Interaction Network - Hydrogen bonds
Hydrogen bonds are the "silent architects" of protein structure. While individual hydrogen bonds are relatively weak compared to covalent bonds, their sheer number within a single protein creates a robust, cooperative network that dictates the protein's final 3D architecture.
In the context of protein folding, hydrogen bonds occur primarily between the polar groups of the polypeptide backbone and, to a lesser extent, between the side chains of amino acids. They are the primary drivers for the formation of local structures!  
*Note*: Because hydrogen bonds are sensitive to pH, temperature, and ionic strength, they are often the first points of failure when a protein undergoes denaturation.

For the purpose of this tutorial, you'll analyze the HBs through the software VMD. How?

```bash
vmd frame0_centered.pdb your_trajectory_skip50_fittedCA.xtc
```

* Go to Extensions --> Analysis --> Hydrogen Bonds

You should get an interactive menu like this (A):

<div align="center">

| VMD Hydrogen Bonds (A) | Number of Hydrogen Bonds (B) |
| :---: | :---: |
| <img src="https://github.com/cidoimo/MDubiquitin_tutorial/blob/main/Analysis/images/hbonds_panelVMD.png" width="500"> | <img src="https://github.com/cidoimo/MDubiquitin_tutorial/blob/main/Analysis/images/number_HB.png" width="500"> |

</div>

For now, you can select just the protein, as you're gonna analyze the intra-protein HBs. You should know that, by selecting different regions of the protein, you can retrieve specific results related to what you're researching about.
It's really important to CHANGE the Donor-Acceptor distance and the Angle cutoff. Why? A hydrogen bond was assumed to exist if the donor–acceptor distance was shorter than 0.30 nm and the hydrogen-donor–acceptor angle was less than 30°. The primary reason to adjust these thresholds is that hydrogen bond geometry is highly dependent on the local protein environment and the specific force field used in MD simulations. A seminal reference for defining geometric constraints in protein hydrogen bonding is: _Baker, E. N., & Hubbard, R. E. (1984). Hydrogen bonding in globular proteins. Progress in Biophysics and Molecular Biology, 44(2), 97-179_.

So, now change the values in this way:
* Donor-Acceptor distance: 3.5
* Angle cutoff: 30
* Calculated detailed info for: from None to Residue Pairs

You can customize your output directory as needed. It is strongly recommend enabling the log file option to ensure you have a comprehensive record of your analysis parameters and execution history readily available.
Furthermore, ensure the "Write output to files" option is selected to generate the following essential datasets:
* Frame/Bond Statistics: This file records the total number of hydrogen bonds detected for every frame of the trajectory. You can easily visualize this data as a time-series plot using tools like xmgrace to identify fluctuations in structural stability over time, as shown above in (B).
* Detailed Hydrogen Bond Data (only works if you select the "calculated detailed info for Residue Pairs"): This file provides a granular breakdown, specifying the participating donor-acceptor pairs and their occupancy (the percentage of time each specific bond is maintained during the simulation). This is a critical metric for identifying key residues responsible for stabilizing the protein fold or mediating functional dynamics. By analyzing high-occupancy bonds over the course of your trajectory, you can effectively distinguish between "transient" interactions and "stable" structural motifs that define your protein's conformational ensemble.

### Step 5a: Interaction Network - Salt Bridges
Beyond hydrogen bonds, salt bridges (electrostatic interactions) are vital structural components. A salt bridge occurs through an electrostatic interaction between a positively charged group (e.g., the side chains of Lysine or Arginine) and a negatively charged group (e.g., Aspartic or Glutamic acid).
Salt bridges are essential to protein structure and function primarily because they provide a unique combination of long-range stability, thermal resistance, and conformational control. Unlike hydrogen bonds, the electrostatic interactions within salt bridges operate over longer distances. This allows them to act as physical anchors between distant protein domains, providing superior structural rigidity to the overall protein fold.
Additionally, these interactions serve as energetic "stitches" that prevent protein denaturation under harsh conditions. This is why proteins from thermophilic organisms—which thrive in extreme heat—often feature a much higher density of salt bridges to maintain their shapes.
Finally, salt bridges are critical for dynamic biological functions because they frequently act as conformational switches. Because they are highly sensitive to their environment, the disruption or formation of a single salt bridge (often triggered by localized pH changes or ligand binding) can cause a massive structural shift, effectively turning a protein's activity on or off.

Before starting the analysis, create a new directory. Trust me, you'll thank me later!

```bash
mkdir SB
```

Again, you'll analyze the SBs through the software VMD. How?

```bash
vmd frame0_centered.pdb your_trajectory_skip50_fittedCA.xtc
```

* Go to Extensions --> Analysis --> Salt Bridges

You should get an interactive menu like this:

<div align="center">

| Salt Bridges |
| :---: |
| <img src="https://github.com/cidoimo/MDubiquitin_tutorial/blob/main/Analysis/images/saltbridges_panelVMD.png" width="500"> |

</div>

Leave the default options. Be sure to select as output directory the new folder you've just created. This is essential because, for each SB found, VMD will retrive a timeserie file. They can be a lot, of course, depending on how big your protein is.

---

<div align="center">

| | **Hey there, gotcha!** | |
| :---: | :--- | :---: |
| | Congratulations! You've successfully navigated the core workflow of Molecular Dynamics. You have learned how to prepare a system, run a simulation, and, most importantly, extract meaningful data to validate your protein's behavior. You now have a solid, comprehensive overview of how to simulate and analyze a molecular trajectory. From the initial system setup to the final RMSD, RMSF, and Radius of Gyration analyses, you have covered the essential steps of a professional MD study. You are definitely on the right track to becoming a structural bioinformatician! Keep exploring! | <img src="https://media.tenor.com/on6WFzYlLJcAAAAM/pokemon-cute.gif" width="250"> |

</div>
