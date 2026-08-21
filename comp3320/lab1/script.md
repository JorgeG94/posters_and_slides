# COMP3320 Lab 1 script 

## Intro to Scientific HPC Lab 

- HPC is fantastic, it is a key to many fields 
  -  Computational Fluid Dynamics:
    - Aerospace engineering fields (boeing, lockheed, airbus)
    - Cars (F1, nascar)
    - Materials dev (wind turbines, buildings)
    - Climate science (atmospheric and oceanic physics)
  - Computational Physics 
    - Astro (NASA, ESP, JEPL)
    - High Energy Physics (Nuclear, CERN, plasma)
  - Computational Chemistry
    - Drug design, molecular dynamics (Big pharma, Johnson, Astra Zeneca, etc.)
    - Catalysts design, materials (Big oil, plastic industry, etc.)
  - Software design/optimization
    - High performance libraries engineer (nvidia, intel, amd, cray)
    - Compiler engineering (arm, nvidia, intel, amd, cray, etc.)
    - Applied research, ML (literally anywhere nowadays)
  - Scientific research and collaboration 
    - National labs all over the world 
    - HPC Centres around the world 

## Lab 1: Time resolution, or: how do I time my programs?

- Timers: 
  - resolution, how is time measured?
  - overhead, how long does it take to call the timing function? 

```c
double measure_time();
double elapsed_time(start,end);

for(int i = 0; i < ni; i++){
  for(int j = 0; j < nj; j++){
    for(int k = 0; k < nk; k++){
      double my_time_start = measure_time();
      {
        do_ops(i,j,k) 
      }
      double my_time_end = measure_time();
      double my_elapsed_time = elapsed_time(my_time_start, my_time_end);
      printf(" took = %f \n ", my_elapsed_time);
    }
  }
}
```
  
## Timing

- Previous program is a 3-fold loop, `my_time_start` will thus be called `ni*nj*nk` times. 
- If `do_ops(i,j,k)` takes ~1ms and the timing subroutine has a similar overhead
  - What are you then measuring?
- How accurate is the `measure_time` routine?

## Resolution 

- If my clock can only measure ms accurately, what happens when my routines take microseconds? 
- Timing routines vary across languages and libraries

## Benchmarking and appropriate timing routines 

- If you are timing wrong you are not benchmarking properly 
- Proper benchmarking relies on a good clock 
- Profiling:
  - Instrument the code at the binary level to extract information 
  - Use instructions and runtime analysis to obtain precise measurements 

## Lab 1: Iterative non-linear solvers

- The program you will study solves a *nonlinear* PDE, the **Bratu problem**

```math
-\nabla^2 u = \lambda e^{u}
```

- Solved on the unit square, `u = 0` on the boundary, on an `n` by `n` grid

## Where does this equation come from?

- A classic model of **thermal self-ignition** — a balance of two terms:
  - `-∇²u`: how fast heat **conducts away**
  - `e^u`: how fast heat is **generated** (hotter ⇒ reacts faster ⇒ feedback loop)
- `lambda` sets generation vs. conduction
- Above a critical `lambda* ≈ 6.808` **no steady state exists** — it ignites
- We fix `lambda = 3.0`, safely below critical, so a solution exists

## How the code solves it

- Nonlinear ⇒ we cannot solve it in one step like `Ax = b`
- Repeatedly apply a fixed-point map `u ← G(u)` (one grid sweep), accelerated by
  **DIIS** (Anderson acceleration) — a workhorse of computational chemistry
- The code reports the time spent in each phase:

| phase        | what it does                          |
| ------------ | ------------------------------------- |
| `map (G)`    | one sweep of the grid                 |
| `error vec`  | forms `e = G(x) − x`                   |
| `B matrix`   | dot products of stored error vectors  |
| `small solve`| tiny dense solve for DIIS coefficients|
| `extrapolate`| combines history into the next iterate|

## What you'll do today

1. **Timer resolution & overhead** — `gettimeofday` vs `clock_gettime`
2. **CPU timing & scaling** — how does one iteration scale, `O(n^k)`? Does a
   better *algorithm* (DIIS) beat brute force?
3. **Profiling with Intel tools** — VTune (hotspots) & Advisor (loops)
4. **Profiling with Linaro Forge** — a second profiler; see what the *compiler* did

## The golden rule

- **Run the code.** 
- The compiler and the hardware do things you cannot see by reading `diis.c` (or maybe you can, I certianly can't)
- Every answer comes from **measurement on Gadi**, not inspection
- You can test on your own computer and compare how things look versus Gadi!


## Getting started

- **Fork** the repo to your account (as in the pre-lab)
- **Clone** it on Gadi:

```bash
git clone git@gitlab.comp.anu.edu.au:<YOUR_UID>/comp3320-2026-lab-1.git
```

- **Build & run**:

```bash
make walltime      # Part 1
make exe_diis      # Parts 2-4
./exe_diis 64
```


