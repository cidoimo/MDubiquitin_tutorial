# Complete AMBER Guide for Master's Thesis
## Classical Molecular Dynamics Simulations

Welcome to this guide! This document is designed specifically for master's thesis students and researchers who want to download, install, and configure **AMBER** and **AmberTools** to run classical molecular dynamics (MD) simulations.

Since 2023, AMBER has become **completely free for academic and non-profit use**, making it an extremely accessible tool for thesis work and independent research.

---

## 📌 Prerequisites and System Requirements

Before starting the installation, make sure your system meets the following minimum and recommended requirements:

* **Operating System:** Linux (Ubuntu 20.04 or 22.04 LTS recommended).
  * *Windows users:* You need to install **WSL2** (Windows Subsystem for Linux) to run a Linux environment inside Windows.
* **Hardware (Highly Recommended):** An NVIDIA GPU supporting the **CUDA** architecture. Running a 100 ns simulation on CPU alone can take months, while a modern GPU can complete it in a few hours or days.
* **Build Tools:** C/C++ and Fortran compilers, plus CMake.

### Installing Dependencies (Ubuntu/Debian)

Update your repositories and run the following command in the terminal to install all required packages:

```bash
sudo apt update
sudo apt install -y build-essential gfortran tcsh make cmake \
                    libbz2-dev libz-dev libpng-dev libx11-dev \
                    libxt-dev libxmu-dev libxi-dev xorg-dev \
                    bison flex
```

---

## 📥 Step 1: Request the Academic License and Download

1. Go to the [Official AMBER Download Page](https://ambermd.org/GetAmber.php).
2. Choose the **Academic/Non-Profit License** option (which includes both the main AMBER module and AmberTools).
3. **Crucial step:** Fill in the registration form using exclusively your **Institutional/University Email Address** (e.g. `@student.unimi.it`, `@student.unito.it`, `.edu`, etc.). The verification system is automated and will instantly approve university domains.
4. Once you receive approval by email, download the source package (usually named `AmberXX.tar.bz2`).

---

## 🛠️ Step 2: Installation and Compilation

Move the downloaded archive to the folder where you want to install the software (e.g. `/home/user/software/` or `/opt/`) and extract it via terminal:

```bash
# Extract the tarball
tar xvfj Amber24.tar.bz2
cd amber24_src/
```

### Option A: Standard CPU Installation

Use this option if you do not have a dedicated NVIDIA graphics card. Compilation will target CPU processing only:

```bash
mkdir build
cd build
cmake .. -DDOWNLOAD_MINICONDA=TRUE
make install
```

### Option B: GPU-Accelerated Installation (Strongly Recommended)

First verify that the NVIDIA CUDA Toolkit is correctly installed on your system by running `nvcc --version`. Then proceed with the configuration enabling CUDA support:

```bash
mkdir build
cd build
cmake .. -DDOWNLOAD_MINICONDA=TRUE -DINSTALL_TESTS=FALSE -DCUDA=TRUE
make install
```

### Configuring Environment Variables

After installation, add the initialization script to your `~/.bashrc` file to make AMBER commands globally accessible from any terminal:

```bash
echo "source /path/to/your/amber24/amber.sh" >> ~/.bashrc
source ~/.bashrc
```

> ⚠️ Remember to replace `/path/to/your/` with the actual path where the AMBER folder is located on your machine.

---

## 🚀 Running Your First Simulation

AMBER has two main computation engines for molecular dynamics:

* **sander:** Ideal for CPU tests, small systems, or custom force field implementations.
* **pmemd:** High-performance optimized version. If you compiled with CUDA support, use `pmemd.cuda` to leverage your GPU.

### Typical Workflow

The preparation and execution of a standard MD simulation generally follows these successive steps:

1. **System Preparation (`tleap`):** Generate topology files (`.prmtop`) and initial coordinates (`.inpcrd`).
2. **Energy Minimization (`pmemd.cuda`):** Geometric relaxation of the system to eliminate any steric clashes (atomic overlaps).
3. **Heating:** Gradual increase of the system temperature from 0 K up to the target temperature (e.g. 300 K).
4. **Equilibration:** Stabilization of the system density and pressure under the NPT thermodynamic ensemble.
5. **Production Run:** Launch of the main trajectory from which analyses will be performed (e.g. a 100 ns simulation).

---

## 💡 Key Advice for Your Thesis: Local PC vs. HPC Cluster

> Simulating complex biomolecules (such as the Ubiquitin protein explicitly solvated in water) for 100 ns requires significant computational power.

* **Personal Computer:** Never launch long production simulations on your standard laptop. The machine risks thermal overheating and the process could take weeks. Use your local machine exclusively for the **preparation phase** (`tleap`), structure visualization, and running **short control tests** (10–50 ps).

* **HPC / Computing Cluster:** Ask your thesis supervisor for access to your university's supercomputing cluster or national computing centers (such as **CINECA** in Italy). On these machines AMBER is generally already installed and configured — you can leverage enterprise GPUs (e.g. NVIDIA A100 or H100) to complete your 100 ns simulations quickly and safely.
