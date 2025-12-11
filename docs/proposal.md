# Project Proposal: 2D Heat Diffusion Simulator (Python + C Extensions)

## 1. Overview

This project implements a **2D heat diffusion simulator** that starts as a simple Python prototype and then evolves into a **high-performance, parallel implementation** using **C extensions (Cython)** and **OpenMP**. The goal is to practice real-world high performance computing (HPC) techniques on a laptop-scale project while keeping the math and domain model simple.

The core idea: simulate how heat spreads on a rectangular metal plate over time using a discrete numerical stencil.

---

## 2. Goals

**Technical goals**

- Implement a **correct baseline** heat diffusion simulation in Python + NumPy.
- Move the performance-critical inner loop into a **Cython-based C extension**.
- Add **shared-memory parallelism** (OpenMP via Cython `prange`) to exploit multiple cores.
- Systematically **benchmark and profile**:
  - Python vs C extension vs C + OpenMP versions.
  - Performance across different grid sizes and time steps.
- Produce a short **report with plots** (runtime, speedup, efficiency).

**Learning goals**

- Practice structuring a project where:
  - Python is the **orchestration / UX layer**.
  - C/Cython handles **hot loops and heavy computation**.
- Build intuition about:
  - Cache behavior and data layout.
  - When parallelization actually helps (and when it doesn’t).
- Create a concrete **portfolio piece** that demonstrates interest and skill in HPC / performance engineering.

---

## 3. Problem Description

We model a aate as an `nx × ny` grid. Each cell `T[i, j]` stores a temperature value. At each time step, the temperature changes based on its four neighbours (a 5-point stencil):

\[
T_{\text{new}}(i, j) = T(i, j) + \alpha \big(
T(i-1, j) + T(i+1, j) + T(i, j-1) + T(i, j+1) - 4 T(i, j)
\big)
\]

- **Boundary conditions:** fixed or clamped (edges stay at their initial values).
- **Initial condition:** cool plate with a **hot region in the center**.
- **Output:** final temperature field after a given number of time steps, plus performance metrics (timings).

This is a classic **stencil computation**, common in HPC (PDEs, fluid dynamics, etc.).

---

## 4. Project Scope

In scope:

- Single-node, shared-memory parallelism on a laptop.
- Python + NumPy orchestration and visualization.
- Cython-based C extension for the core stencil loop.
- OpenMP threading in the C extension.
- Benchmarking and basic profiling (e.g., `time.perf_counter`, `perf`, `callgrind` or similar if available).
- Simple visualization of the final temperature field using Python (e.g., Matplotlib).

Out of scope (for now):

- Distributed-memory parallelism (MPI across multiple machines).
- Advanced PDE schemes, adaptive mesh refinement, or complex boundary conditions.
- GPU acceleration (CUDA, OpenCL).

---

## 5. Technologies and Tools

- **Language / runtimes**
  - Python 3.x
  - C/C++ via Cython-generated extensions

- **Libraries**
  - NumPy (array storage and convenience)
  - Cython (C extensions + `prange` + nogil)
  - OpenMP (shared-memory parallelism)
  - Matplotlib (basic plots/heatmaps)
  - `setuptools` (for `setup.py` build process)

- **Environment**
  - Single laptop with a multi-core CPU
  - Linux/macOS (OpenMP may need extra steps on macOS)

---

## 6. Project Structure (Planned)

```text
hpc-heat-sim/
├── README.md
├── proposal.md           # this document
├── setup.py              # build Cython extension
├── src/
│   └── heat/
│       ├── __init__.py
│       ├── simulate.py   # Python orchestration (baseline + wrapper)
│       └── heat_core.pyx # Cython core (C + OpenMP)
├── scripts/
│   ├── run_experiment.py # sweeps sizes / steps / threads, writes CSV
│   └── plot_results.py   # creates graphs from CSV
└── results/
    └── scaling.csv       # timing data (created later)
```

---

## 7. Milestones and Timeline

### Milestone 1 – Baseline Python Implementation

- Implement `init_plate(nx, ny)` to create initial temperature field.
- Implement `step_python(curr, alpha)` with nested Python loops.
- Implement `run_sim_python(nx, ny, steps, alpha)` that runs the simulation.
- Verify correctness on small grids (e.g., 50×50, 100×100).
- Record baseline timings for a few sizes.

**Deliverable:** working pure-Python simulator + initial timings.

---

### Milestone 2 – C Extension via Cython

- Write `heat_core.pyx` with a `step_c(curr, next_grid, alpha)` function using C-level loops.
- Integrate into `simulate.py` via `run_sim(..., use_c=True/False)`.
- Configure `setup.py` and build extension.
- Compare performance:
  - Python vs C extension for multiple grid sizes and steps.
- Ensure numerical results match (within floating-point tolerance).

**Deliverable:** C-accelerated version with clear speedup over pure Python.

---

### Milestone 3 – Parallelization with OpenMP

- Add `step_c_parallel(...)` in `heat_core.pyx` using `prange` and `nogil`.
- Update `setup.py` to compile and link with OpenMP.
- Add `run_sim_parallel(...)` in `simulate.py`.
- Run experiments with different `OMP_NUM_THREADS` values.
- Observe scaling behaviour and diminishing returns.

**Deliverable:** threaded C extension demonstrating multi-core speedup.

---

### Milestone 4 – Experiments, Profiling, and Report

- Implement `scripts/run_experiment.py` to:
  - Sweep `nx = ny` over a set of sizes (e.g., 200, 500, 1000, 2000).
  - Run:
    - pure Python
    - C single-threaded
    - C + OpenMP
  - Log timings to `results/scaling.csv`.
- Implement `scripts/plot_results.py` to generate:
  - Runtime vs problem size.
  - Speedup vs problem size / threads.
- Optionally run a profiler (e.g., `perf`, `callgrind`) on one configuration to inspect hot spots.
- Write a short report (or section in `README.md`) summarizing:
  - Methods
  - Results (with plots)
  - Observations (e.g., memory vs compute bound, cache effects, OpenMP overhead)
  - Possible future extensions.

**Deliverable:** CSV data + plots + write-up with interpretation.

---

## 8. Evaluation Criteria

The project will be considered successful if:

1. **Correctness**
   - C and C+OpenMP implementations produce output consistent with the baseline Python version.

2. **Performance**
   - The C extension achieves a substantial speedup over pure Python for non-trivial grid sizes.
   - The OpenMP version shows improved performance over the single-threaded C version for at least some grid sizes and thread counts.

3. **Code quality**
   - Clear separation of concerns (Python orchestration vs C core).
   - Reasonable documentation and comments.
   - Simple command-line usage for experiments.

4. **Documentation**
   - `README.md` explains how to build, run, and reproduce results.
   - Plots and discussion show understanding of performance behavior, not just numbers.

---

## 9. Possible Future Extensions

If time and interest allow:

- Add **different boundary conditions** (Dirichlet, Neumann).
- Implement **blocked/tiling** updates to improve cache locality and compare vs naive loops.
- Add **simple GUI or interactive notebook** for visualizing the diffusion over time.
- Explore **GPU acceleration** (e.g., CuPy, numba.cuda, or a separate CUDA/C project).
- Investigate **distributed-memory** versions (e.g., MPI via `mpi4py`) on a multi-node environment.

---

## 10. Summary

This project is a small but realistic HPC workflow on a laptop:

- Start with a clear mathematical model (heat diffusion).
- Build a simple Python reference implementation.
- Move the critical loop into optimized C/Cython code.
- Add multi-core parallelism.
- Benchmark, profile, and document.

It directly reinforces concepts from **parallel programming, performance engineering, and numerical computing**, and produces a concrete artifact that can be showcased in a portfolio or discussed in interviews.
