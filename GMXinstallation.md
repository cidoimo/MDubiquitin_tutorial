# Complete GROMACS Guide for Master's Thesis
## Classical Molecular Dynamics Simulations

Welcome to this guide! This document is designed for master's thesis students and researchers who want to download, install, and configure **GROMACS** (GROningen MAchine for Chemical Simulations) to run classical molecular dynamics (MD) simulations.

GROMACS is **completely free and open-source** (LGPL license), making it one of the most widely used MD engines in the world for biomolecular simulations.

> **Current stable version:** GROMACS 2025.4

---

## 📌 Prerequisites and System Requirements

* **Operating System:** Linux (Ubuntu 20.04 or 22.04 LTS recommended).
  * *Windows users:* Install **WSL2** (Windows Subsystem for Linux) to run a Linux environment inside Windows.
  * *macOS users:* GROMACS can be compiled natively, but Linux is strongly preferred for production use.
* **Hardware (Highly Recommended):** An NVIDIA GPU supporting **CUDA** for GPU-accelerated simulations. CPU-only runs of 100 ns simulations can take weeks; a modern GPU completes them in hours or days.
* **Build Tools:**
  * CMake **3.28 or later**
  * C and C++ compilers (GCC 9+ recommended)
  * CUDA Toolkit (optional, for GPU support)

### Installing Dependencies (Ubuntu/Debian)

```bash
sudo apt update
sudo apt install -y build-essential cmake git \
                    libfftw3-dev libopenmpi-dev openmpi-bin \
                    zlib1g-dev libxml2-dev
```

---

## ⚡ Quick Install via apt (Easiest Method)

If you just want to get started quickly — for learning, testing, or preparing system files — you can install GROMACS directly from Ubuntu's package manager with a single command:

```bash
sudo apt update
sudo apt install -y gromacs
```

Verify it works:

```bash
gmx --version
```

**That's it.** No compilation required.

> ⚠️ **Important caveat:** The apt package ships an older version of GROMACS (2021.4 on Ubuntu 22.04, 2023.3 on Ubuntu 24.04) and is **compiled without GPU/CUDA support**. This means it will run only on CPU, which is fine for preparation steps and short tests, but **not suitable for long production simulations**. For a full 100 ns run, use the source compilation method below (Option B with CUDA).

**When to use `apt install`:**
* ✅ Learning GROMACS for the first time
* ✅ Preparing system files (`pdb2gmx`, `editconf`, `solvate`, `genion`)
* ✅ Short test runs (< 1 ns) to verify your setup
* ❌ Long production simulations (use source build with CUDA instead)

---

## 📥 Step 1: Download GROMACS (Source Build)

GROMACS is free and does not require registration. Download the latest source tarball directly:

```bash
wget https://ftp.gromacs.org/gromacs/gromacs-2025.4.tar.gz
```

Alternatively, you can download it manually from the [official GROMACS download page](https://www.gromacs.org/Downloads).

### Extract the Archive

```bash
tar xfz gromacs-2025.4.tar.gz
cd gromacs-2025.4
```

---

## 🛠️ Step 2: Installation and Compilation

Always build GROMACS in a separate `build/` directory — never compile directly in the source folder.

```bash
mkdir build
cd build
```

### Option A: Standard CPU Installation

Use this option if you do not have a dedicated NVIDIA GPU. GROMACS will build its own internal FFTW library and download regression tests automatically:

```bash
cmake .. \
  -DGMX_BUILD_OWN_FFTW=ON \
  -DREGRESSIONTEST_DOWNLOAD=ON

make -j$(nproc)
make check
sudo make install
```

> ⚠️ `make -j$(nproc)` uses all available CPU cores to speed up compilation. This can take 10–30 minutes depending on your machine.

### Option B: GPU-Accelerated Installation with CUDA (Strongly Recommended)

First verify that the NVIDIA CUDA Toolkit is installed on your system:

```bash
nvcc --version
```

Then compile with CUDA support enabled:

```bash
cmake .. \
  -DGMX_BUILD_OWN_FFTW=ON \
  -DREGRESSIONTEST_DOWNLOAD=ON \
  -DGMX_GPU=CUDA \
  -DCUDA_TOOLKIT_ROOT_DIR=/usr/local/cuda

make -j$(nproc)
make check
sudo make install
```

### Option C: MPI Installation (for HPC Clusters)

If you plan to run parallel simulations across multiple nodes on an HPC cluster, compile with MPI support:

```bash
cmake .. \
  -DGMX_BUILD_OWN_FFTW=ON \
  -DGMX_MPI=ON \
  -DGMX_GPU=CUDA

make -j$(nproc)
sudo make install
```

> 💡 On HPC clusters, GROMACS is usually already installed as a module. Ask your sysadmin or check with `module avail gromacs`.

---

## ⚙️ Step 3: Configure Environment Variables

After installation, source the GROMACS environment script to make all commands available in your terminal:

```bash
source /usr/local/gromacs/bin/GMXRC
```

To make this permanent, add it to your `~/.bashrc`:

```bash
echo "source /usr/local/gromacs/bin/GMXRC" >> ~/.bashrc
source ~/.bashrc
```

Verify the installation was successful:

```bash
gmx --version
```

---

## 🚀 Running Your First Simulation

GROMACS uses a single unified command `gmx` followed by a subcommand (tool name). The main engine for running simulations is `gmx mdrun`.

* **`gmx mdrun`** (CPU): Standard simulation engine.
* **`gmx mdrun -gpu_id 0`**: Offloads non-bonded interactions to the GPU.
* **`gmx mdrun -ntmpi 1 -ntomp 8`**: Controls the number of MPI ranks and OpenMP threads.

### Typical Workflow

The standard MD simulation pipeline in GROMACS follows these steps:

1. **System Preparation (`gmx pdb2gmx`):** Convert your PDB structure file into GROMACS topology (`.top`) and coordinate (`.gro`) files, selecting a force field (e.g. CHARMM36, AMBER99SB-ILDN, OPLS-AA).

2. **Define the Simulation Box (`gmx editconf`):** Set the periodic boundary condition (PBC) box around your molecule with an appropriate margin (e.g. 1.0 nm).

3. **Solvation (`gmx solvate`):** Fill the simulation box with explicit water molecules (e.g. TIP3P, SPC/E).

4. **Add Ions (`gmx genion`):** Neutralize the system charge and set the desired salt concentration (e.g. 0.15 M NaCl) by replacing water molecules with Na⁺ and Cl⁻ ions.

5. **Energy Minimization (`gmx mdrun`):** Relax the system geometry to remove steric clashes before dynamics.

6. **NVT Equilibration:** Equilibrate the temperature at constant volume, coupling the system to a thermostat (e.g. V-rescale) at 300 K for ~100 ps.

7. **NPT Equilibration:** Equilibrate pressure at constant temperature and pressure using a barostat (e.g. Parrinello-Rahman) for ~100 ps.

8. **Production Run (`gmx mdrun`):** Launch the main trajectory for analysis (e.g. 100–500 ns). This is the step that requires GPU acceleration or an HPC cluster.

### Key Input File: the `.mdp` File

Every GROMACS simulation step is controlled by a **`.mdp` parameter file**. A minimal example for a production run:

```ini
; Run parameters
integrator  = md        ; leap-frog integrator
nsteps      = 50000000  ; 100 ns (50M steps × 2 fs)
dt          = 0.002     ; 2 fs timestep

; Output
nstxout-compressed = 5000   ; save coordinates every 10 ps

; Temperature coupling
tcoupl      = V-rescale
tc-grps     = Protein Non-Protein
tau_t       = 0.1 0.1
ref_t       = 300 300

; Pressure coupling
pcoupl      = Parrinello-Rahman
ref_p       = 1.0
tau_p       = 2.0
```

---

## 🔍 Useful Post-Processing Tools

GROMACS includes a rich set of built-in analysis tools accessible via `gmx <tool>`:

* **`gmx rms`** — Calculate RMSD of the backbone over time.
* **`gmx rmsf`** — Calculate per-residue RMSF (flexibility).
* **`gmx gyrate`** — Radius of gyration over time.
* **`gmx hbond`** — Hydrogen bond analysis.
* **`gmx trjconv`** — Convert, center, and wrap trajectories (run this before any analysis).

---

## 💡 Key Advice for Your Thesis: Local PC vs. HPC Cluster

> Simulating a solvated protein system for 100+ ns requires significant computational power.

* **Personal Computer:** Use your local machine only for the **preparation steps** (`pdb2gmx`, `editconf`, `solvate`, `genion`) and for **short test runs** (10–100 ps) to verify your setup is correct before launching the full simulation.

* **HPC / Computing Cluster:** Ask your thesis supervisor for access to your university's supercomputing cluster or national centers (such as **CINECA** in Italy, or **PRACE** infrastructure in Europe). GROMACS scales extremely well across multiple GPUs and nodes — a 100 ns simulation of a ~100k atom system can complete in under a day on modern hardware.

## 📚 Useful Resources

* [Official GROMACS Documentation](https://manual.gromacs.org/current/)
* [GROMACS Tutorials by Justin Lemkul](http://www.mdtutorials.com/gmx/) — the best free beginner tutorials available
* [GROMACS User Forum](https://gromacs.bioexcel.eu/)
* [BioExcel Best Practices for GROMACS](https://bioexcel.eu/research/best-practice-guides/)
