# Title TBD 

- Jorge Luis Galvez Vallejo, NCI 

---

## Who is Jorge 

- My timeline:
  - Chemical Engineer (2 semesters)
  - Experimental chemist (4 semesters)
  - Closet theoretical physicist (2 semesters)
  - Computational chemist (2.5 years)
  - Computational scientist (2018-2025) in Chemistry
  - Computational scientist (2025-now) in Climate, Weather, Geophysics 

---

## What does Jorge do? 

- Mostly GPU porting and optimization of existing codebases 
- Overall maintenance and upgrade of codebases, think:
  - Quality of life upgrades: refactors, better build systems, automated testing 
  - Algorithmic upgrades: rewrite some bits of the code to be more parallelizable 
- Performance optimization 
  - Make code go as fast as humanely possible within the expectations of the project 

---

## Why is Jorge here? 

- My work comes from you reaching out
- You can't reach out if you don't know I exist 
- The more I learn about the underlying field the better I can do my job 

---

## What is Jorge doing? 

- Everything is related to water!
- ANUGA: A flooding simulator (unstructured grids, shallow water equations, purely 2D)
- MOM6: Circulating water around the world (structured grids, mainly 3D)
- Rakali: My hobby/learning platform for the above two codes 

---

## ANUGA 

- Collaboration with CDAC (India), ANU, GA, and the NCI through an MOU (Memorandum of Understanding)
  - CDAC uses ANUGA to model flooding emergency response around rivers during the Monsoon season 
  - If code is faster then they can do better, more accurate simulations in less time 
- ANUGA python orchestrated, C backend for speed 
- State of the code when I started:
  - 4 parallel backends: serial C, SIMD C, OpenMP, OpenACC 
  - GPU capabilities: sparse, poorly integrated, not tested 
  - Great unit testing and automated testing

---

## ANUGA: towards a GPU accelerated codebase

- For GPUs to be worth it, EVERYTHING needs to be accelerated
- You are only as fast as your slowest bit
- Rearchitecture how the codebase works 

---

## How does GPU acceleration work?

- GPUs are very fast because they have thousands of cores versus CPUs have hundreds of cores 
- Problem needs to be very parallel to be nice for GPUs (sometimes all you need is a large problem)
- Problems can be _memory_ or _compute_ bound 
  - Memory: how fast can you access elements to do a computation 
  - Compute: how fast can you do arithmetic with such elements 
- Separate memory address spaces
  - CPU has its memory 
  - GPU has its own memory too 
  - How do they talk? The BUS (a limiting factor, got to optimize around it)

---

## In a nutshell, how to write very fast GPU code:

- Allocate all memory at the start and reuse it 
- Minimize large memory transfers between the CPU-GPU during the main computational loop 
- Make sure your problem is large enough to use the GPU efficiently 
- Only leave very small, insignificant operations on the CPU 
- Profit 

---

## Expressing parallelism 

- loops are inherently a serial construct, do i=1, i=2, ...
- parallelism is expressed via `pragmas`: `omp parallel do`
- YOU are telling the program that this can be executed in parallel
 - if you mess up, well the code will be wrong 
- Lessons in parallelism:
  - No side effects outside the loop (no writing to a global state, thread safety)
  - Minimize branching (ifs, switches, cases) 
  - Make the code easier for the compiler to optimize (simple is always best)
  - Don't use fancy object oriented shenanigans inside hot loops 

---

## Road to GPU ANUGA:

- ANUGA was already parallelized with MPI (distributed memory)
- ANUGA is memory bound due to unstructured grids 
- OpenMP -> MPI parallelism 
  - MPI divides a domain into subdomains and they're each executed concurrently
  - OpenMP takes a full domain and launches _threads_ to evaluate it in parallel 
- MPI: 
  - Reduces _memory pressure_ by making the problem smaller and executing it serially
- OpenMP:
  - Keeps problem constant but launches parallel resources to evaluate it in parallel 
- A mixture is how you want to achieve GPUs:
  - One MPI subdomain per GPU
  - Each GPU gets a very large subdomain

---

## How to get GPUs?

- Thanks to ANUGA already being parallelized with OpenMP the trick was:
  - switch `#pragma omp parallel for` to `#pragma omp target teams loop`
- The pain was in the memory 
  - The memory backend was refactored from a single global domain state to an _owning_ object 
  - Memory was allocated all upfront and managed efficiently

```
domain.set_memory(n_triangles, quantities)
domain.distribute(n_gpus)
for t in range(t_end):
  domain.evolve(t)
  if t == t_print:
    domain.print_console_update()

```

--- 

## What did we achieve?

- CDAC simulation used to take 4 hours on 4416 cores
  - Can run the same sim on 4 nodes (16 GPUs) in less than an hour 
  - In terms of savings for SUs: ~10X (speedup is trickier to calculate across different units)
- Before you needed a big compute node to do a simulation of around 16M triangles (realistic)
  - You can now run this on your laptop (provided you have a GPU with at least 8GB of memory)

---

## MOM6 and the GPU debacle 

- The same ideas apply to MOM6
- MOM6 has more constraints:
  - Keep CPU performance
  - Minimize duplicated code 
  - Keep bitwise reproducibility 
  - Keep consortium happy 

---

## Why is MOM6 harder? 

- Because we cannot upend the codebase to write fast code without killing CPU performance or making very large changes 
- Performance is limited by the algorithm that can keep both CPU and GPU happy 
  - Luckily there are good compromises 
- Iteration is slow
  - MOM6 is not very well tested 
    - Well tested means: fast, automated tests with little room for programmer error 
    - MOM6 is extensively tested by many people but the automation is tricky (especially for GPUs) 


---

## MOM6 goal:

- ACCESS-NRI: CPU/GPU parity in terms of SUs 
- NOAA-GFDL: GPU must be faster than CPUs on their machines 

---

## MOM6 work:

- Driven by NOAA-GFDL and ACCESS-NRI
- My work has been to provide them with support
- I have done a lot of investigating corner cases and performance issues 
- Multi GPU support, compiler bugs, etc. 

---

## How do we do GPUs in MOM6?

- `do concurrent` plus data mapping protocol 
  - A nice way to express parallelism 
  - No need to move to C++ 

--> insert here some of the slides from the CDAC talk

---


## How do I make sure we get the best performance? 

- If there are no constraints I am the happiest:
  - rewrite the algorithm entirely 
  - explore hardware level optimizations 
  - re-express the parallelism in terms of faste operations, such as BLAS library calls 
- Profiler driven optimization:
  - Tools that tell you how far you are from the optimal runtime 
- Knowing the physics to understand what is legal and what is not

---

## Hobby/learning project: Rakali

- Rakali is my playground model where I learned SWE and ocean dynamics 
- GPU accelerated from scratch, all the best practices for data transfer, memory, etc. 
- Written in Fortran to demonstrate you don't need C/C++
- A good way to learn how to do stuff I don't (or didn't) fully understand 

---

## Rakali

- Explicit 2D/3D hydrodynamic model with structured/unstructured grids 
  - think flooding, erosion, tides, easy to couple with waves
  - CFL keeps timestep in check, good for estuarine physics where things are awful
- Implicit 2D/3D hydrodynamic model with structured/unstructured grids 
  - Same as the explicit but with implicit timestepping 
  - Think SCHISM, Delft3D, maybe a bit ROMS
- Full global circulation model 
  - basically MOM6/MITgcm with regional capabilities baked in  

---

## Reason for Rakali

- Performance optimization
- Explore different algorithmic schemes, pathways without bothering anyone 
- Simpler build and deployment 
- Simpler portability across vendors due to more control over what gets built
- I can break things and no one but me cares
- Designing a code from scratch is extremely fun 
- Testing the limits of AI assisted code development




