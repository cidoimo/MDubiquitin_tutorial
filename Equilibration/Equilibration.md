# Equilibration (NVT) and Pressurization (NPT)

After completing the four static energy minimization steps, the system must be equilibrated—first in temperature (**NVT** ensemble) and then in pressure (**NPT** ensemble). 

An NVT simulation allows the atoms to move and interact based on classical mechanics, gradually reaching an equilibrium state where positions, velocities, and energies fluctuate around stable average values.

---

## 🌡️ Equilibration (NVT)

The thermal equilibration is performed over **four sequential runs** with gradually decreasing harmonic constraints on the solute (residues 1-354):
* **`nvt1.in`**: Tight constraints at **0.5 kcal/mol** (includes the heating ramp)
* **`nvt2.in`**: Constraints at **0.25 kcal/mol**
* **`nvt3.in`**: Constraints at **0.10 kcal/mol**
* **`nvt4.in`**: Light constraints at **0.05 kcal/mol**

### ⚠️ Critical Note on Temperature & Heating (`nvt1.in`)
At the start of `nvt1.in`, the system's temperature is at **0 K**. Therefore, the first run must force a linear heating ramp from **0 K to 300 K** over the course of the simulation using `nmropt=1` and `&wt` blocks. The subsequent runs (`nvt2` to `nvt4`) keep the temperature stable at a constant 300 K.

# 
The parameter file should be structured similarly to the previous minimization file. It should include the definition of the constraints and the range of residues to which they should be applied. For all the parameters to specify the parameter file, please refer to the ![link](https://ambermd.org/tutorials/basic/tutorial0/index.php)  

### NVT Input File Templates (`.in`)

#### 1. `nvt1.in` (Heating from 0K to 300K | 0.5 kcal/mol restraints)
```amber
NVT Equilibration Stage 1 - Heating to 300K
 &cntrl
  imin=0, irest=0, ntx=1, iwrap=1, ntb=1,
  cut=9.0, ntr=1, restraintmask=':1-354', restraint_wt=0.5,
  ntc=2, ntf=2, nmropt=1, temp0=300.0, ntt=3, gamma_ln=1.0,
  nstlim=2500000, dt=0.002, ntpr=500, ntwx=500, ntwr=500, ioutfm=1,
 /
 &wt type='TEMP0', istep1=0, istep2=2500000, value1=0.0, value2=300.0, /
 &wt type='END', /
```
                                                                                                                                                                                                                                                                                      
Note: nstlim=2500000 with dt=0.002 calculates to exactly 5 ns of simulated time (2,500,000×0.002 ps=5,000 ps=5 ns).

#### 2. `nvt2.in, nvt3.in, nvt4.in` (Constant 300K | Restraints decrease)
For the remaining NVT stages, modify only the restraint_wt and the temperature values in the &wt block to maintain a steady 300 K:

```amber
NVT Equilibration Stage 2/3/4 - Constant 300K
 &cntrl
  imin=0, irest=1, ntx=5, iwrap=1, ntb=1,
  cut=9.0, ntr=1, restraintmask=':1-354', 
  restraint_wt=0.25,     ! Change to 0.10 for nvt3, and 0.05 for nvt4
  ntc=2, ntf=2, nmropt=1, temp0=300.0, ntt=3, gamma_ln=1.0,
  nstlim=2500000, dt=0.002, ntpr=500, ntwx=500, ntwr=500, ioutfm=1,
 /
 &wt type='TEMP0', istep1=0, istep2=2500000, value1=300.0, value2=300.0, /
 &wt type='END', /
```

### 🚀 GPU Execution & Job Monitoring
From NVT onwards, simulations run on GPUs using pmemd.cuda. Follow this execution protocol:

#### 1. Check GPU Status
```bash
nvidia-smi
```
Inspect the output table. Look at the GPU ID (0 or 1) and ensure the Volatile GPU Utilization (%) is at 0%, indicating it is idle and free to use.

#### 2. Assign the Target GPU

```bash
export CUDA_VISIBLE_DEVICES=x   # Replace 'x' with your free GPU ID (0 or 1)
```

#### 3. Launch the Job in the Background

Run the jobs sequentially using *nohup* and & to safely disconnect from the terminal session without killing the process.
Step 1 (Starting from the 4th minimization restart file):

```bash
nohup pmemd.cuda -O -i nvt1.in -p complesso.prmtop -c restrt.em4 -o mdout.nvt1 -inf mdinfo.nvt1 -r restrt.nvt1 -x mdcrd.nvt1 &
nohup pmemd.cuda -O -i nvt2.in -p complesso.prmtop -c restrt.nvt1 -o mdout.nvt2 -inf mdinfo.nvt2 -r restrt.nvt2 -x mdcrd.nvt2 &
```
(Continue this sequential pattern for nvt3 and nvt4).


📊 Real-Time Monitoring Commands

* *htop*: Monitor host CPU and RAM usage.
* *nvidia-smi*: Check active VRAM allocations and GPU compute load.
* *tail -f mdinfo.nvt1*: Track calculation progress, current temperature, and Estimated Time of Arrival (ETA).

---

## 🌡️ Pressurization (NPT)
The NPT ensemble maintains a constant Number of particles (N), Pressure (P), and Temperature (T). This mimics realistic experimental environments where the molecular system is in contact with atmospheric conditions.

Following the four NVT runs, a single fully unconstrained NPT run (5 ns) is required to stabilize the system's density and pressure. In the NPT Input File (npt.in) all positional restraints are removed (ntr=0), and the barostat is activated (ntb=2, ntp=1).
Once npt.in is ready, launch it in the background using the restart file generated at the end of the final NVT run (restrt.nvt4):

```bash
nohup pmemd.cuda -O -i npt.in -p complesso.prmtop -c restrt.nvt4 -o mdout.npt -inf mdinfo.npt -r restrt.npt -x mdcrd.npt &
```
Once this phase completes successfully, your system's volume, temperature, and pressure will be fully equilibrated, making it completely ready for long-term Production MD runs.
