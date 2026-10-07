# Plan — "Química computacional" (Tec de Madero, 50 min, español)

Source: `SCRIPT.md`. Output: `presentation.tex` (Cookie theme, Fortran purple, unchanged preamble
from `../adac_nci_mom6/presentation.tex` lines 1–49). Target **~42–46 frames** incl. section pages.
Everything on the slides in **Spanish** (with accents, UTF-8). Keep technical acronyms in English
where that is how chemists say them (SCF, LCAO, CCSD(T), QM/MM, DFT...).

## Global decisions
- `babel` spanish is NOT installed: keep `\usepackage[main=english]{babel}` from the preamble and
  just write Spanish text. Add `\usepackage{pgfplots}\pgfplotsset{compat=1.18}`, `braket`, `siunitx`, `mhchem`.
- Title: "Química computacional" / subtitle "Una amalgama de física, matemáticas, química y computación".
  Author Jorge Luis Gálvez Vallejo; institute NCI · ANU; `\cookieuni{Australian National University}`,
  `\cookielocation{Canberra, Australia}`; email jorge.galvezvallejo@anu.edu.au.
  Tagline: "Instituto Tecnológico de Ciudad Madero". Logos: `nci.png` only (copy from `../template/assets/`),
  headshot `jorge_c.png`. Closing QR: `https://github.com/JorgeG94`.
- Closing: `\cookieclosingtitle{¡Gracias!}`, final frame "¡Gracias!" + `{\large ...\textbf{¿Preguntas?}}`.
- **Figures:** do NOT pull copyrighted book images. Draw them ourselves in TikZ/pgfplots, and label the
  source honestly: "Datos: PySCF, cc-pVDZ (cálculo propio)" or "Esquema adaptado de Szabo & Ostlund (1996)" /
  "Helgaker, Jørgensen & Olsen (2000)" / "Perdew & Schmidt (2001)". For things that really need a photo
  (Cray-1, ENIAC, Frontier, a protein), define a `\figph{<descripción + fuente sugerida>}` macro that draws a
  dashed cookieMuted box with the text, so Jorge can drop real images in later. Use at most ~5 of these.
- Every equation-heavy frame: at most 2–3 displayed equations, each with a one-line Spanish gloss.
- Keep Jorge's voice and jokes ("el horror", "el horror parte 2", "dolor en general"). "Todo este pedo"
  → soften to something stage-safe but casual ("y todo lo demás que ya sabemos que duele").
- "Claude echame una mano y di que me echaste la mano" (sólidos): honour it with a `\cplx{}` aside on that
  frame, e.g. "\cplx{(Sección armada con ayuda de Claude — Jorge no hace sólidos)}".

## Real data already computed (use with pgfplots `table`)
- `assets/data/h2_dissociation.dat` — columns `R RHF UHF MP2 FCI` (Å, Hartree), H₂/cc-pVDZ.
  RHF goes to −0.76 Eh at 5 Å (wrong), UHF → −0.9986 (= 2×E(H), correct), FCI exact in basis, RMP2 diverges.
  Plot y-range about [−1.20, −0.70]. Annotate: "RHF: H⁺ + H⁻ espurio", "UHF rompe simetría de espín",
  "MP2 sobre RHF diverge", dashed line at −0.9986 "2 E(H)". Mark the Coulson–Fischer point (~1.2 Å) roughly.
  Also a second panel / frame: E_corr = E_FCI − E_RHF vs R (grows at long R → static correlation).
- `assets/data/he_basis.dat` — columns `X Ecorr_mEh err_mEh` for He FCI, cc-pVXZ X=2..5.
  Plot error vs X on log y, plus the X^{-3} fit line → motivates F12. Exact He E = −2.903724 Eh.

## Structure (sections → frames)
**0. Portada + índice** (toc=aftertitle gives index automatically).

**§1 ¿Qué es la química? (≈5 min)**
1. Química — materia y cambio; definiciones primaria/secundaria/universidad (3-row escalation, funny).
2. De átomos a moléculas a propiedades — dureza, pH, fragilidad; TikZ ladder átomo → molécula → material.
3. Modelos de la química — moderno = reciente; Lewis, Bohr (small TikZ Bohr atom).
4. Modelo clásico y su ruptura — catástrofe ultravioleta: Rayleigh–Jeans vs Planck pgfplots
   (analytic functions, label "Esquema"), eq. $E=h\nu$, Planck's law. Mecánica/termodinámica estadística.

**§2 Mecánica cuántica (≈10 min)**
5. Schrödinger y Heisenberg — $\hat H\Psi=E\Psi$, $i\hbar\partial_t\Psi$, $[\hat x,\hat p]=i\hbar$; onda vs matrices, misma física.
6. El átomo de hidrógeno — solvable; $E_n=-\tfrac{1}{2n^2}$ Eh; problema de muchos cuerpos ya con He.
7. El Hamiltoniano molecular — full $\hat H$ in atomic units (T_n, T_e, V_ne, V_ee, V_nn), label each term.
8. Born–Oppenheimer — $m_p/m_e\approx1836$; $\Psi\approx\psi_e(\mathbf r;\mathbf R)\chi(\mathbf R)$; superficie de energía potencial.
9. Campo promedio — the $1/r_{ij}$ is the villain; Slater determinant; antisimetría.
10. Hartree–Fock — Fock operator $\hat f=\hat h+\sum_j(\hat J_j-\hat K_j)$; intercambio sin análogo clásico.
11. LCAO y Roothaan–Hall — $\phi_i=\sum_\mu C_{\mu i}\chi_\mu$; $\mathbf{FC}=\mathbf{SC}\boldsymbol\varepsilon$; Gaussian basis $e^{-\alpha r^2}$ (Boys).
12. El ciclo SCF — TikZ flowchart: guess → build F → diagonalize → new D → converged? ; problema no lineal; DIIS.
13. Lo que nos da HF — principio variacional $E_{HF}\ge E_0$; Hellmann–Feynman $\frac{dE}{d\lambda}=\langle\Psi|\partial_\lambda\hat H|\Psi\rangle$ (fuerzas → geometrías); virial $2\langle T\rangle=-\langle V\rangle$.
14. **Energía de correlación** — definition $E_{corr}=E_{exacta}-E_{HF}$ (Löwdin); ~1% of total but ≈ size of chemistry (kcal/mol); TikZ energy-level bar diagram: E_HF, E_exact, gap labelled E_corr. Note: 1 Eh = 627.5 kcal/mol.
15. **HF se rompe al disociar** — the H₂ pgfplots figure (data above). RHF vs UHF vs FCI vs MP2.
16. ¿Por qué falla RHF? — minimal-basis argument: $\sigma_g^2$ = 50% iónico (H⁺H⁻) + 50% covalente; UHF arregla la energía pero contamina espín ($\langle S^2\rangle\neq0$). alertblock.

**§3 Más allá de Hartree–Fock (≈12 min)**
17. DFT — Hohenberg–Kohn: $E[\rho]$, $\rho(\mathbf r)$ 3 variables vs 3N; Kohn–Sham $E=T_s+V_{ne}+J+E_{xc}$; the unknown $E_{xc}$.
18. La escalera de Jacob — TikZ ladder (LDA, GGA, meta-GGA, híbridos, doble-híbridos) from "Tierra de Hartree" to "Cielo de exactitud química" (Perdew & Schmidt 2001), examples (SVWN, PBE, TPSS/SCAN, B3LYP/PBE0, B2PLYP).
19. Correlación dinámica y el horror — Rayleigh–Schrödinger PT; Møller–Plesset partition $\hat H=\hat F+\hat V$; MP2 formula with $(ia|jb)$ and $\varepsilon$ denominators.
20. Coupled cluster — $|\Psi\rangle=e^{\hat T}|\Phi_0\rangle$, $\hat T=\hat T_1+\hat T_2+\dots$; CCSD(T) "estándar de oro"; size-extensivity.
21. Interacción de configuraciones — $|\Psi\rangle=c_0|\Phi_0\rangle+\sum c_i^a|\Phi_i^a\rangle+\dots$; FCI exact in basis but $\binom{K}{N}$ explosion (give numbers).
22. **El problema de la cúspide y los métodos F12** — Kato cusp $\left.\frac{\partial\bar\Psi}{\partial r_{12}}\right|_{r_{12}=0}=\tfrac12\Psi(r_{12}=0)$; orbital products can't make a kink → slow $X^{-3}$ convergence: the He pgfplots (data above).
23. **Hylleraas y F12** — Hylleraas (1929) He with explicit $r_{12}$: $\Psi=e^{-\zeta s}\sum c_{lmn}s^l t^{2m} u^n$, $u=r_{12}$ — exact to many digits already in 1929; modern: Slater geminal $f_{12}=-\tfrac{1}{\gamma}e^{-\gamma r_{12}}$ (Ten-no), MP2-F12 / CCSD(T)-F12 (Kutzelnigg, Klopper, Werner, Valeev): triple-ζ F12 ≈ quintuple-ζ conventional. exampleblock takeaway.
24. Correlación estática y el horror parte 2 — near-degeneracy, bond breaking, multiple determinants with comparable weight; tie back to frame 15.
25. Métodos multirreferencia — CASSCF (active space, TikZ orbital boxes: inactive/active/virtual), MCSCF, MRCI, EOM-CC (excited states), FCI; CAS(n,m) cost.
26. Más allá de Born–Oppenheimer — non-adiabatic coupling $\mathbf d_{IJ}=\langle\psi_I|\nabla_R\psi_J\rangle=\frac{\langle\psi_I|\nabla_R\hat H|\psi_J\rangle}{E_J-E_I}$; blows up at degeneracy → intersecciones cónicas (TikZ double cone sketch), surface hopping (Tully), fotoquímica (visión: retinal).

**§4 El problema de la escala (≈10 min)**
27. Química computacional = cómputo paralelo — scaling table: HF N⁴, MP2 N⁵, CCSD N⁶, CCSD(T) N⁷, FCI exponential; booktabs table + "duplicar el sistema → ×128 para CCSD(T)".
28. Integrales de repulsión electrónica — $(\mu\nu|\lambda\sigma)=\iint\chi_\mu\chi_\nu r_{12}^{-1}\chi_\lambda\chi_\sigma$; O(N⁴) number, 8-fold symmetry, Schwarz screening, density fitting/RI as aside; they feed everything.
29–35. **Historia del cómputo paralelo** (Claude to fill — one frame each, short):
   29. Una CPU, von Neumann (ENIAC 1945, `\figph`), frequency scaling era.
   30. Ley de Moore & escalado de Dennard — pgfplots log-scale transistor counts (4004 1971 2.3e3; 8086 1978 2.9e4; 80386 1985 2.75e5; Pentium 1993 3.1e6; Pentium 4 2000 4.2e7; Core 2 2006 2.9e8; Nehalem-EX 2010 2.3e9; 2017 ~1.9e10; Apple M1 Ultra 2022 1.14e11) + note "Dennard muere ~2005: muro de potencia".
   31. Vectorización / SIMD y las Cray (Cray-1 1976, 160 MFLOPS, `\figph`) — vector registers, one instruction many data.
   32. Memoria compartida: multinúcleo, OpenMP (tiny cookiecode Fortran `!$omp parallel do`, ≤40 chars wide).
   33. Memoria distribuida: clústeres Beowulf (1994), MPI (1994) — tiny MPI snippet or TikZ nodes+network.
   34. GPUs: CUDA 2007, from graphics to science; Frontier 2022 first exascale (1.1 EFLOPS) (`\figph`); GPUs dominate TOP500.
   35. FPGAs, ASICs, TPUs, cómputo cuántico — specialization; quantum computing as the natural "simulate quantum with quantum" (Feynman 1982).
   36. Lo que esto significa para la química — codes Jorge-world: GAMESS, NWChem, ORCA, Q-Chem, CP2K; exascale ports; "el algoritmo importa tanto como el hardware".

**§5 Escalas, campos de fuerza y sólidos (≈7 min)**
37. El límite de la química cuántica — quantum effects matter locally; multiscale TikZ (QM ⊂ MM ⊂ continuo) with length/time scale axes.
38. QM/MM — $E=E_{QM}+E_{MM}+E_{QM/MM}$; link atoms, mecánico vs electrostático embedding, ONIOM; Nobel 2013 (Karplus, Levitt, Warshel); "dolor en general" alertblock (frontier, cost balance).
39. Mecánica molecular — force field equation (bonds harmonic, angles, dihedrals, LJ 12-6, Coulomb); AMBER/CHARMM/OPLS; GROMACS, LAMMPS; proteins (`\figph` optional).
40. Cómo se hacen los campos de fuerza — parametrization against QM + experiment, transferibility; ML potentials (ANI, MACE, NequIP) as modern training.
41. Campos de fuerza reactivos — ReaxFF: bond order $BO_{ij}(r_{ij})$, allows bond breaking; limits (parametrización por sistema, carga, transferibilidad).
42. Sistemas extendidos: sólidos — Bloch $\psi_{n\mathbf k}(\mathbf r)=e^{i\mathbf k\cdot\mathbf r}u_{n\mathbf k}(\mathbf r)$, periodic boundary conditions, plane waves + pseudopotentials/PAW, k-points, bands; VASP, Quantum ESPRESSO, CP2K; band-gap problem of DFT. `\cplx` Claude aside.

**§6 Aplicaciones y el futuro (≈6 min)** — be verbose as asked, but split into 2–3 frames, two columns:
43. Aplicaciones I: descubrimiento de materiales (Materials Project, high-throughput DFT), baterías (voltajes, electrolitos, cátodos Li/Na), catálisis (Haber–Bosch, barreras), celdas solares/fotovoltaica.
44. Aplicaciones II: fármacos (docking, FEP, AlphaFold 2 — Nobel 2024), espectroscopía (IR, RMN, UV-vis), química atmosférica/ambiental, teoría básica (entender el enlace), petroquímica (relevant to Madero / ingeniería química: catalizadores de refinación, zeolitas, desulfuración, corrosión).
45. Química computacional y ciencias de la computación — algorithmic innovations (FMM, linear scaling, density fitting, tensor decompositions), lenguajes (Fortran → C++ → Python/Julia, Kokkos/SYCL), AI/ML (ML potentials, Δ-learning, generative design, LLM-agents).
46. Avenidas de investigación (Claude fill, tie to ingeniería química + CS): (a) ML potentials para reactores y catálisis heterogénea; (b) química cuántica en GPUs/exascala; (c) multiescala: de DFT a CFD de reactores (microcinética); (d) química cuántica en computadoras cuánticas (VQE); (e) software científico sostenible (RSE). End with exampleblock "¿Cómo empezar?" (Python + PySCF, Psi4, ORCA gratis académico, libros: Szabo & Ostlund; Jensen; Cramer).
47. Conclusiones (3 bullets).
48. ¡Gracias! + ¿Preguntas?

## Bibliography frame (optional, before thanks, small font)
Szabo & Ostlund *Modern Quantum Chemistry*; Helgaker/Jørgensen/Olsen *Molecular Electronic-Structure Theory*;
Jensen *Introduction to Computational Chemistry*; Perdew & Schmidt AIP Conf. Proc. 577 (2001);
Hylleraas Z. Phys. 54, 347 (1929); Kong, Bischoff & Valeev Chem. Rev. 112, 75 (2012); Tully JCP 93, 1061 (1990).

## Build & QA
`cp ../template/beamerthemecookie.sty . ; cp ../cdac_anuga/Makefile .` then `make`.
0 Overfull (or only trivially small ones), render & look at title, H₂ plot, He plot, Jacob's ladder, a code frame.
