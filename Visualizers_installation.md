# Prerequisites and Installation Guide (Linux)  

To run the analysis pipeline and visualize the GROMACS output data (such as RMSD, RMSF, Rg, and PCA), you need to install Python, XMGrace, and R on your Linux system. Follow the unified terminal commands below to set up your environment.
You can choose one of them or use all of three here proposed. It's your choice. For simplicity, in this tutorial xmgrace will be used ad standard.

---

## Installation via Command Line

On Ubuntu, Debian, or Windows Subsystem for Linux (WSL), open your terminal and run the following commands to update your package lists and install all the necessary tools and dependencies:

```bash
# Update the local package repository index
sudo apt update

# Install Python 3, pip, and essential data science libraries
sudo apt install python3 python3-pip python3-numpy python3-matplotlib

# Install XMGrace for native GROMACS .xvg plotting
sudo apt install grace

# Install the core R environment for statistical plotting
sudo apt install r-base

```

## Verification

To ensure that all applications were successfully installed and are properly linked to your environment path, execute the version checks below in your terminal:

```bash
python3 --version
xmgrace -v
R --version
```
