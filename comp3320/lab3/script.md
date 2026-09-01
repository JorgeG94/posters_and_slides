# COMP3320 Lab 3 script — Programming with SIMD

An interactive walk through the lab for the 90-minute session. The rhythm of every
beat is the lab's own: **predict, run, argue with the result**. Slides pause before
each reveal so the room commits to an answer first.

## The story you have been told

- "The compiler vectorises your loops. If it doesn't, it's aliasing; add `restrict`."
- This lab tests that claim instead of believing it. It is largely false on a modern
  compiler — and the loop that genuinely refuses does so for a reason that is
  invisible in the source.
- The answer: the vector unit CAN do conditional work (mask registers, since 2016),
  but C has no way to say so. So the compiler can't, and you have to.
- Not intrinsics: the Vector Class Library. Same instructions (they check this in
  Q11), readable source, compiles anywhere.

## How to run the session

- The lab is longer than the session and none of it is marked — both deliberate.
- Core path in session: 1, 2, 3, 5, 7, 9, 10, 17.
- Afterwards: Q13 first (bit-exact harness, no demonstrator needed), then Q11
  (deepest), then whatever interests them. _Extension._ = droppable. Part 5 optional.

## Setup traps (get these done in the first ten minutes)

1. Compute node: `qsub -I -q normal ... mem=4GB`. Why 4GB: Gadi charges
   max(cpus, mem/4GB) — 16GB costs 4x for the same work. Whole lab ~2 SU/hour.
2. Wrong node is SILENT: VCL emulates Vec8d on Broadwell, everything runs, numbers
   mean nothing. `lscpu | grep avx512`, and Q1 prints INSTRSET for exactly this.
3. `module unload nvidia-hpc-sdk; unset CPATH C_INCLUDE_PATH CPLUS_INCLUDE_PATH`
   — its x86intrin.h shadows gcc's and the build dies inside /apps/. make refuses.
4. `module load gcc/15.1.0` (system gcc is 2018; several answers differ).
5. Define `pin()`. Unpinned varied by 3x while the lab was being written.

## Part 1 — the vocabulary

- Intrinsics two-liner vs VCL two-liner side by side. Same speed, one is readable,
  and only one compiles on a machine without AVX-512.
- The whole idea: comparison → 8 bools in a mask register; `select(mask, a, b)`.
  C has no syntax for that pair — that's the lab in one sentence.
- Two footguns flagged now: `load_partial` zero-fills (Q13c asks when zero lies);
  `horizontal_max` is not per-lane, placement decides the loop's speed (Q13b).
- INTERACTIVE (Q2): before running anything, write down the predicted speedup of
  variant 2 over variant 0. Show a demonstrator. Q17 collects the debt.

## Part 2 — the folklore

- POLL (Q3): gcc -O3, `a[i] = a[i] + b[i]`, no restrict. Vectorised or not?
  Reveal: ask the compiler (`-fopt-info-vec`) — then sweep 2 compilers x 2 levels;
  restrict is decisive in exactly ONE of four configs, not the one you'd guess.
- Q5: what restrict actually does. The driver passes overlapping arrays on purpose;
  the promise is a lie; answers go wrong in groups. POLL: what sets the group size?
  Reveal: the register width the compiler happened to pick (objdump). Silent wrong
  answers — when SHOULD you use restrict?
- Q6: vecblock — no aliasing at all, restrict truthful, still refuses.
  POLL: variant 1 has no `if`; where is the branch? Reveal: `sqrt` must set errno
  for negative input — a branch the library put there. `-fno-math-errno` unlocks it.
  gcc 15 proves the argument non-negative by itself (Q6e) — "one compiler, 2018".

## Part 3 — the measurement

- Q7: the four-variant table. Mass is conserved exactly by the scheme, so the
  printed mass is a real correctness check. -ffast-math build: mass is CLOSER to
  exact — so what is the problem? (You didn't ask for the arithmetic to change and
  can't tell which way it moved anywhere else.)
- Q8 POLL: variants 0 and 1 have IDENTICAL times, every run. Noise? Reveal:
  nm/objdump — the compiler folded them into one function. Measuring a rewrite in
  the same binary can be measuring nothing.
- Q9/Q10: the refusal string again (conditional divide this time); three routes:
  VCL by hand / restructure the source (variant 3 — buys nothing here) /
  -ffast-math (changes the answer). Which do you ship when the requirement is
  "the answer does not change"?
- Q11 teaser: eight lanes, nowhere near 8x. Divider/sqrt throughput, not bandwidth
  — and the sweep only shows it if you HOLD TOTAL WORK FIXED (else you measure the
  clock ramping and it looks exactly like a cache curve).

## Afterwards

- Q13 first: a reduction is structurally different (n values → 1); harness checks
  bit-for-bit; the three traps are listed in the source. Don't let an AI write it.
- Part 5: same three obstacles in a 600-line production Fortran kernel. Q15 kills
  the "compiler can't see through the call" hypothesis. Q16 is the job: "here is
  the bottleneck, the fix, what it is worth, and why I am not doing it."
- Q17 standout: how wrong was the Q2 prediction, and where?
