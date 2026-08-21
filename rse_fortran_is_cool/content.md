## Title: Fortran is cool, don't let your friends convince you otherwise

Presenter: Jorge Luis Galvez Vallejo (NCI)

NCI logo

Target: 15 min slot -> ~13 min speaking, ~10 slides. Hard budget in each header.

---

## 1. Your friends are half right  [0:30 -> 2:00]

Side by side, same computation:

- LEFT: F77. implicit typing, COMMON blocks, GOTO, 6-space columns, ENTRY
- RIGHT: modern Fortran. explicit interfaces, allocatables, array syntax, associate

Line: the reputation is real, and it is about **code**, not about the **language**.
Every complaint your friends have is a complaint about 1985.

---

## 2. Why the debt exists  [2:00 -> 3:00]

- These codebases were written by scientists, not software engineers
- That is not a failing. Nobody staffed the engineering
- Climate, weather, CFD still run on this code. It is not going away
- So: a vacancy, not a lost cause. This is the RSE-shaped hole in the talk

---

## 3. Modern Fortran in one slide  [3:00 -> 4:30]

- Object orientation (F2003)
- (almost) safe dynamic memory: allocatable, move_alloc, automatic deallocation
- Arrays as first-class citizens: slicing, elemental, reductions, no loop needed
- Native parallelism: `do concurrent` (shared memory + GPU), coarrays (distributed)

Say out loud and move on, pays off on slide 7:
"I use do concurrent. For distributed memory I use MPI. I'll tell you why in a minute."

---

## 4. One loop, every device  [4:30 -> 6:00]

Reuse the MOM6 comparison verbatim from adac_nci_mom6/content.md.

```fortran
do concurrent(k=1:nk, j=1:nj, i=1:ni)
  c(i,j,k) = alpha * a(i,j,k) + b(i,j,k)
end do
```

versus the directive wall:

```fortran
!$omp target teams distribute parallel do collapse(3)
!$acc parallel loop collapse(3)
```

Point: the first one is just Fortran. No pragma dialect, no vendor, no second language.
Compiler flag chooses serial / multicore / GPU.

---

## 5. "Sure, for easy loops"  -- terco  [6:00 -> 7:30]

The objection writes itself, so answer it before it is asked.

- Hardest realistic case in my field: two-electron repulsion integrals.
  Irregular, deeply nested, the kernel everyone insists must be hand-written CUDA
- Prior art: Alkan et al (JCP 2024), del Angel (JCP 2026) -- OpenMP target + do concurrent
  inside GAMESS, per-shell-quartet
- terco: batches of shell pairs dispatched to the device, so the GPU is not starved
- OpenACC used **only** to map data host<->device. Unneeded under unified memory.
  All parallelism is do concurrent

**El numero / the number:**

- terco on a **single V100** vs **Q-Chem on 104 threads, Sapphire Rapids** -> **3.5x faster**
- V100 is a 2017 part. Say that out loud: this is plain do concurrent on seven-year-old
  silicon beating a commercial code on a current CPU node
- Deliberately NOT compared against cuEST: cuEST is density-fitted only, terco is
  conventional. That would not be a fair comparison and someone in the room knows it.
  Saying this unprompted is worth more than the number

TODO(jorge): state molecule / basis / method on the slide. The whole credibility of the
comparison rests on it being like-for-like, and you just made a point of rejecting cuEST
for not being like-for-like.

Two questions that will come, have answers ready:
- one GPU vs a whole CPU node: standard comparison in QC, but say which node
- is Q-Chem the right baseline: yes, it is commercial and well optimized. Good baseline

Name means stubborn. Callback on the last slide.

---

## 6. Two witnesses, four vendors  [7:30 -> 9:00]

Same conclusion reached independently in unrelated domains:

- **MOM6** -- 200k lines of climate code I did not write. do concurrent + a few directives
- **Rakali** -- coastal / river / flood / open ocean hydrodynamics, 100% Fortran
- **metalquicha** -- quantum chemistry, three engines behind one `qc_method_t` interface

Platform table (from Rakali): GPUs NVIDIA / AMD / Intel. CPUs Apple / ARM / Intel / AMD.

Line: climate and quantum chemistry, different codebases, same answer. Two witnesses is
qualitatively different from one.

---

## 7. THE TURN -- the language is portable, the ecosystem is not  [9:00 -> 10:30]

This is the pivot of the talk. Slow down here.

- stdlib is genuinely good and in practice builds with GNU
- `mpi_f08` needs vapaa (Hammond) to cross compilers at all
- coarrays: elegant in F2008, but GCC needs OpenCoarrays, Intel ships its own runtime,
  Cray's is good, NVIDIA and AMD are nowhere
- I could put a coarray backend behind pic-mpi. I chose not to: most coarray runtimes are
  implemented on MPI anyway, so I would be wrapping a wrapper, and I would owe it a CI
  matrix on compilers where coarrays barely work

My criterion is not "native is good". It is **portable and production-ready, or it does
not count**. do concurrent passes. Coarrays do not, yet.

That is the kind of decision Fortran codebases do not get made for them often enough.

---

## 8. So I built the missing layer  [10:30 -> 11:30]

One diagram, 40 seconds, do not open any of them:

- **pic** -- stdlib-level routines in native Fortran: sorting, arrays, strings, loggers, timers
- **pic-blas** -- portable BLAS surface
- **pic-mpi** -- distributed memory
- **pic-device** -- device layer
- interop, because you are never trapped: **libfint** (libcint ported to Fortran),
  cuEST module interface

fpm installable. Builds where the science needs to build.

TODO(jorge): compiler list for this slide. Which compilers, and is it CI-enforced or
by hand? This number is the direct rebuttal to "stdlib only builds with GNU" and someone
will ask.

---

## 9. What RSE actually looks like in Fortran  [11:30 -> 12:45]

metalquicha as the existence proof. Show the badge row, then the thing that matters:

- Three engines behind one interface -- tblite (semi-empirical CPU), libcint (ab initio CPU),
  cuEST (GPU). Swap the engine, fragmentation and screening are unchanged
- libcint path exists to be **disagreed with**: it gives the GPU path a second independent
  implementation to check against, and every method is validated against PySCF
- CI, coverage, ReadTheDocs, FORD, pre-commit, contributing guide

This is differential testing plus an external oracle. It is the single most credible slide
in the deck, because it is exactly what scientific Fortran usually lacks.

**The ask:** Fortran does not need rewriting. It needs engineering. CI matrices across six
compilers, packaging, API design, tests. That is RSE work, and it is unclaimed.

---

## 10. Close  [12:45 -> 13:00]

Callback: terco means stubborn. The project is a refusal to accept that something cannot be
done in plain old Fortran. So is this talk.

Links: pic, pic-mpi, pic-blas, pic-device, rakali, metalquicha, terco, libfint. fortran-lang.

---

## Backup slides (do not present, have ready)

- coarray Q&A: "elegant design, but distributed memory is where the ecosystem beats the
  language. MPI is portable today, coarrays are not, and I will not recommend something I
  would not ship. If the vendors converge I will change my mind."
- terco batching detail
- pic-device internals
- MOM6 async / sync figures (assets/async.png, assets/sync.png)
