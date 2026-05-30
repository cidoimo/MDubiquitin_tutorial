## 👁️ Visualization of Trajectories with VMD
![VMD](https://img.shields.io/badge/Visualization-VMD-green)

Isn't it beautiful? Molecular dynamics (MD) simulations are essential for understanding molecular behavior over time, but interpreting the complex data can be challenging. Visualization programs like **VMD (Visual Molecular Dynamics)** are the standard for making sense of this data.

### Key Benefits of Visualization
* **Enhanced Understanding:** See molecular movements and interactions in real-time.
* **Interaction Identification:** Spot vital conformational changes for drug design.
* **Validation:** Compare simulation results against experimental models.
* **Educational Value:** An engaging way to explore molecular dynamics.

---

### Preparing Trajectories with CPPTRAJ
Before loading a trajectory into VMD, you must strip away the water molecules and ions (the solvent used to mimic physiological conditions). We use **CPPTRAJ** (integrated into AmberTools) to convert your `.mdcrd` file into a cleaner `.xtc` format.

**Example workflow:**
```bash
cpptraj
parm yourfile_ionized.prmtop
trajin mdcrd.nvt1
strip :WAT,Na+    # Removes water and sodium ions
autoimage :1-46   # Corrects periodic boundary condition (PBC) artifacts
trajout md_nowat.xtc
go
quit
```

Once generated, you can visualize the result locally: *vmd yourfile_nowat.pdb md_nowat.xtc*

### Installing VMD

If you don't have VMD installed:
* Visit the VMD Download page.
* Register and download version 1.9.3.
* Extract the tar file.
* In the folder, run ```./configure``` and then cd src/ followed by ```sudo make install```.
* Everything would work, now, by simply typing ```vmd``` in your command line.

## 🎨 Personalize Your View
Once you have your trajectory loaded, the real fun begins! (In this folder you'll find the frame0 of the MD and the xtc file. You can use them, if you like).  
**Have fun experimenting:** change the representations (NewCartoon, Licorice, Surf), play with colors to highlight different domains, and explore every single frame of your protein in VMD. Make it look as good as the science behind it!
