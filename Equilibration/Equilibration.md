# Equilibration (NVT) and Pressurization (NPT)

After completing the four static energy minimization steps, the system must be equilibrated—first in temperature (**NVT** ensemble) and then in pressure (**NPT** ensemble). 

An NVT simulation allows the atoms to move and interact based on classical mechanics, gradually reaching an equilibrium state where positions, velocities, and energies fluctuate around stable average values.

---

## 🌡️ Step 5: Equilibration (NVT)

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
                                                                                                                                                                                                                                                                                      
