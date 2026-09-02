# COMP3320 Lab 3 script - SIMD and matrix multiplication

Session companion for the matmul lab (rebuilt 2026-09-01 to track lectures 4-1, 4-2, 5-1).
Kept deliberately lean: the lab is 8 questions, the deck is ~13 frames, the rhythm is
predict → run → check.

## From the lectures to the lab

- Three claims from the lectures, all testable in 90 minutes on Gadi:
  1. The compiler auto-vectorises simple loops (4-1).
  2. VCL lets you write SIMD by hand without intrinsics (4-2).
  3. Naive matmul runs at 1-2% of peak (5-1).
- Everything is C = A*B in doubles; harness prints GFLOP/s = 2n³/walltime and checks
  every element. Q1-7 in session, Q8 extension, Q5 (write the kernel) is the centrepiece.

## Setup (first ten minutes)

- qsub normal queue, mem=4GB (charging: max(cpus, mem/4GB) - 16GB costs 4x).
- module load gcc/15.1.0; unset CPATH C_INCLUDE_PATH CPLUS_INCLUDE_PATH (make hard-stops:
  an x86intrin.h on those paths shadows gcc's own and VCL will not build through it).
- pin() everything; wrong node is silent (VCL emulates Vec8d) - Q1 prints INSTRSET.

## What the compiler does (Q2-4)

- Two lecture kernels side by side: naive ijk vs flip-k. POLL: which vectorises?
  Reveal: BOTH - gcc 15 does the reduction in order. The real tell is the widths:
  16/32-byte vectors, never 64. Hold for Q6.
- Q3 measure: ~8x from loop order alone, zero SIMD written. Stride-n reads of B vs
  stride-1. (Lecture 5-1's analysis.)
- Q4: flip-k is NOT bit-identical to naive (~1e-13) with no unsafe flags. POLL: where
  from? Reveal: FMA contraction, on by default; fused rounds once; neither is wrong.

## Write it yourself (Q5-7)

- Q5: mm_vcl = flip-k, eight j's at once. Vec8d, broadcast, mul_add, load_partial tail.
  Bit-identical vs mm_flip_k at every n incl. 1003, or it's wrong. Write it yourself -
  the harness knows, and the exam is closed-AI.
- Q6: registers: compiler chose ymm; you chose zmm by naming the type. exe_mm_wide
  (-mprefer-vector-width=512, the lecture's flag) → honest tie. What hand-writing buys.
- Q7: peak = 2 FMA × 8 doubles × 2 FLOP × clock ≈ 100 GF/s; you're at ~6%. The gap is
  memory, not width → blocking (Q8) → BLAS (~90% of peak) → threads are week 8.

## Blocking & beyond (Q8, extension)

- Lecture's blocked kernel verbatim; here it lands BETWEEN naive and flip-k - analyse →
  iterate → verify, honestly. Check line: not bit-identical (you reassociated, no flag
  needed). Ladder: algorithmica; ship BLAS.
