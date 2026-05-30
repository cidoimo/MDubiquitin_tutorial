# MDubiquitin_tutorial
Molecular Dynamics (MD) tutorial to simulate ubiquitin protein with AMBER

![AmberTools 2025](https://img.shields.io/badge/AmberTools-2025-blue)
![Amber2025](https://img.shields.io/badge/Amber-2025-red)
![VMD](https://img.shields.io/badge/Visualization-VMD-green)
![CUDA](https://img.shields.io/badge/GPU-CUDA%20Accelerated-orange)
![Gromacs2025](https://img.shields.io/badge/Gromacs-2025-yellow)

Comprehensive introduction to classical molecular dynamics (MD) simulations for Master's thesis students. This tutorial provides hands-on guidance for setting up, running, and analyzing an MD simulation of biomolecular systems.

## Target System & Scope
* **Protein:** Ubiquitin (small, stable, well-characterized regulatory protein)
* **Software:** Amber (AmberTools 2025 / Amber 2025 / Gromacs 2025)
* **Simulation Length:** 100 ns 
* **Ensemble:** NVT heating followed by NPT production

## Tutorial Workflow
The training pipeline is structured into three progressive modules:

1. **MD Simulation of Ubiquitin with AMBER (Work Block)**
   * System preparation via LEaP (solvation, neutralization) --> [CLICK ME](https://github.com/cidoimo/MDubiquitin_tutorial/tree/main/Topology%20Building)
   * Energy minimization (steepest descent & conjugate gradient) --> [CLICK ME](https://github.com/cidoimo/MDubiquitin_tutorial/tree/main/Energy%20Minimization)
   * System heating and equilibration with position restraints --> [CLICK ME](https://github.com/cidoimo/MDubiquitin_tutorial/tree/main/Equilibration)
   * 100 ns unrestrained production run using CUDA acceleration --> [CLICK ME](https://github.com/cidoimo/MDubiquitin_tutorial/tree/main/Production)

2. **Visualization of Trajectories with [VMD](https://github.com/cidoimo/MDubiquitin_tutorial/tree/main/VMD%20Visualization)**
   * Loading topology and coordinate trajectory tracks
   * Customizing graphical representations (NewCartoon, structural color-coding)

3. **Analyses of MD Trajectories**
   * Quantitative metrics extraction via `cpptraj` / Gromacs tools
   * Stability and Flexibility assessments: Root Mean Square Deviation (RMSD), Root Mean Square Fluctuation (RMSF), Radius of Gyration (RGyr), Principal Component Analysis (PCA) and building of Dynamical Cross-Correlation Matrices (DCCMs). Secondary Structure Analysis
   * Interactions Networks: Hydrogen Bond and Salt Bridges analyses

## Prerequisites
* Basic Linux/Unix command-line competency
* AMBERTools and AMBER25 installed on your pc
* Gromacs suite 2025 installed on your pc
* Xmgrace, Python or R installed on your pc
* Access to GPU-accelerated computing nodes (for production run efficiency)


---
---

| | 📝 **Credits & Documentation** | |
| :---: | :--- | :---: |
| | **Thanks to Alessia Pinto for beta-testing and debugging this tutorial!** <br><br> Congrats on surviving the setup! Welcome to the club! <br><br> 📚 *For all technical details and full parameters, please visit the [AMBER25 Manual](https://ambermd.org/doc12/Amber25.pdf).* | ![Pikachu in serious mood](https://media.tenor.com/_hzf0RfV_w0AAAAM/pikachu-in-serious-mood-pikachu.gif) 
