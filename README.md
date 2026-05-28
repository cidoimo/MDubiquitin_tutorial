# MDubiquitin_tutorial
Molecular Dynamics (MD) tutorial to simulate ubiquitin protein with AMBER

![AmberTools 2025](https://img.shields.io/badge/AmberTools-2025-blue)
![VMD](https://img.shields.io/badge/Visualization-VMD-green)
![CUDA](https://img.shields.io/badge/GPU-CUDA%20Accelerated-orange)

Comprehensive introduction to classical molecular dynamics (MD) simulations for Master's thesis students. This tutorial provides hands-on guidance for setting up, running, and analyzing an MD simulation of biomolecular systems.

## Target System & Scope
* **Protein:** Ubiquitin (small, stable, well-characterized regulatory protein)
* **Software:** Amber (AmberTools 2025 / Amber 2025)
* **Simulation Length:** 100 ns 
* **Ensemble:** NVT heating followed by NPT production

## Tutorial Workflow
The training pipeline is structured into three progressive modules:

1. **MD Simulation of Ubiquitin with AMBER (Work Block)**
   * System preparation via LEaP (solvation, neutralization)
   * Energy minimization (steepest descent & conjugate gradient)
   * System heating and equilibration with position restraints
   * 100 ns unrestrained production run using CUDA acceleration

2. **Visualization of Trajectories with VMD**
   * Loading topology and coordinate trajectory tracks
   * Customizing graphical representations (NewCartoon, structural color-coding)

3. **Analyses of MD Trajectories**
   * Quantitative metrics extraction via `cpptraj` / Gromacs tools
   * Stability and Flexibility assessments: Root Mean Square Deviation (RMSD), Root Mean Square Fluctuation (RMSF), Radius of Gyration (RGyr), Principal Component Analysis (PCA) and building of Dynamical Cross-Correlation Matrices (DCCMs). Secondary Structure Analysis
   * Interactions Networks: Hydrogen Bond and Salt Bridges analyses

## Prerequisites
* Basic Linux/Unix command-line competency
* Access to GPU-accelerated computing nodes (for production run efficiency)
