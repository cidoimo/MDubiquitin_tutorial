## 🧬 Classical Molecular Dynamics (MD Production)

The production run is the final phase where the actual data collection for the properties of interest occurs. This is the most time-consuming part of the simulation, as it requires a massive number of steps to generate a statistically representative trajectory. 

The length of the production run depends strictly on the system being studied and the properties of interest. A general rule of thumb is to run the simulation for at least **10 times the relaxation time** of the system (the time it takes for the system to forget its initial state and reach a true equilibrium). Once the NPT system equilibration phase is completely stable, the system is ready for production.

### 🧮 Simulation Time Calculation (100 ns)
To set up a **100 ns** production run, we must adjust the `nstlim` parameter using the standard formula:

$$\text{nstlim} \times \text{dt} = \text{time in ps} \quad (\text{where } 1000\text{ ps} = 1\text{ ns})$$

* In the previous NPT phase, $2,500,000 \text{ steps} \times 0.002\text{ ps} = 5,000\text{ ps} = 5\text{ ns}$.
* For a **100 ns** ($100,000\text{ ps}$) production run, you need exactly **$50,000,000$ steps**:

$$50,000,000 \text{ steps} \times 0.002\text{ ps} = 100,000\text{ ps} = 100\text{ ns}$$

#
#### *IMPORTANT* See the example md input file in this folder! Tip: ntpr, ntwx, and ntwr have been increased to 5000 steps (every 10 ps). Writing to disk every 500 steps during a 50,000,000-step simulation would generate unnecessarily massive log and trajectory files.

Because this step calculates millions of interactions over 100 ns, it will take a significantly longer time to finish. The overall duration is strictly dependent on the total number of atoms in your system.
Make sure your target GPU is still selected (export CUDA_VISIBLE_DEVICES=x) and launch the job in the background using the restart file from your completed NPT run (restrt.npt):

```bash
nohup pmemd.cuda -O -i md.in -p complesso.prmtop -c restrt.npt -o mdout.md -inf mdinfo.md -r restrt.md -x mdcrd.md &
```

### 📊 Monitoring the Long Run

You can continuously monitor the progress, hardware load, and Estimated Time of Arrival (ETA) using the same commands as before:
* *nvidia-smi* - to check GPU compute usage and VRAM consumption.
* *tail -f mdinfo.md* - to track the current step, speed (ns/day), and remaining time until completion.

---

## 🏁 You made it!
Congrats, you've reached the end of this tutorial! Are you still sane? 

| | |
| :--- | :--- |
| The first time you run an MD simulation is a classic "rite of passage." You’ll move from panic (monitoring `mdinfo` every minute) to awe when you finally see those atoms vibrate in VMD. Expect a few "Fatal Errors," some disk space headaches, and a lot of patience, but it’s worth it. Now, let the GPU do the heavy lifting—you've done the hard part. Welcome to the club! | ![Sana e salva](https://img1.picmix.com/output/stamp/normal/0/9/0/4/1604090_a14a5.gif) |
