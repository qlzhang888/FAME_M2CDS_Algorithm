# FAME_M2CDS_Algorithm

Source code, benchmark instances, and experimental data for the paper
**"A connectivity-aware local search with frequency-adaptive memory-guided escape for the minimum 2-fold connected dominating set problem"**.

This repository provides the **complete source code of FAME_M2CDS, the solver proposed in the paper, and of the five comparison algorithms** (GFTCDS, GRASP, GRASP_MAR, MSLS, RLS_Tabu), together with the 110 benchmark instances and the raw experimental data reported in the paper.

FAME_M2CDS solves the minimum 2-fold connected dominating set problem (M(2-fold)CDSP): given a connected undirected graph, find a minimum vertex set D such that every vertex outside D has at least two neighbors in D and the subgraph induced by D is connected.

## Repository contents

| Item | Description |
|---|---|
| `code.zip` | Source code of FAME_M2CDS (proposed solver) and the five comparison algorithms |
| `data.zip` | 18 CSV files of raw per-seed experimental results reported in the paper |
| `instance.zip` | 110 benchmark instances (`.mtx` format) |
| `forced_vertex_stats.csv` | Per-instance forced-vertex statistics (paper Table 14), also included in `data.zip` |
| `LICENSE` | MIT License |

### code.zip

```
code/
├── FAME_M2CDS/                 the proposed solver
│   ├── benchmark/              batch entry point (run_benchmark.py)
│   ├── solver/                 Python driver chain (graph_env, cpp_bridge, hybrid_solver, diagnostics)
│   └── cpp_engine/             C++ local search engine sources
│       ├── engine_m2cds.cpp    the engine
│       ├── engine.h            C interface of the engine
│       └── m2cds_solver_main.cpp   standalone command-line solver
└── comparison_algorithms/      five comparison algorithms (re-implemented in C++)
    ├── GFTCDS/     ├── GRASP/
    ├── GRASP_MAR/  ├── MSLS/
    └── RLS_Tabu/
```

### instance.zip

| Suite | #Instances | Vertices | Description |
|---|---|---|---|
| RandomUDG | 24 | 15–30 | random unit disk graphs |
| CommonUDG | 41 | 80–400 | common unit disk graphs |
| LPNMR09 | 9 | 40–90 | LPNMR'09 competition graphs |
| NDR_M2CDS | 36 | 100–21498 | graphs from the Network Data Repository |

### data.zip

| File(s) | Rows | Content |
|---|---|---|
| `fame_full.csv` | 1100 | per-seed results of FAME_M2CDS (110 instances × 10 seeds) |
| `ablation_no_{ale,car,fame,memory,nsp,stats}.csv` (×6) | 1100 each | the six ablation variants |
| `baseline_{GFTCDS,GRASP,GRASP_MAR,MSLS,RLS_Tabu}.csv` (×5) | 1100 each | per-seed results of the five comparison algorithms |
| `baseline_validation.csv` | 424 | validation of the re-implemented competitors against published results (paper Table 3) |
| `tuning_parameters.csv` | 270 | one-factor-at-a-time parameter tuning, 26 configurations plus the default (54 instances × 5 seeds) |
| `forced_vertex_stats.csv` | 110 | per-instance forced-vertex statistics (paper Table 14) |
| `new_30/exact_gap.csv` | 30 | 30 generated instances with CPLEX-proven optima vs FAME_M2CDS (paper Table 10) |
| `new_30/fame_results.csv` | 300 | per-seed results of FAME_M2CDS on the 30 generated instances |
| `new_30/cplex_results.csv` | 30 | CPLEX solving log of the 30 generated instances |

Solver result files use the schema `dataset,instance,seed,cost,solve_time_s`, where `cost` is the size of the best 2-fold connected dominating set found within the 600 s cutoff and `solve_time_s` is the time at which it was first reached. Baseline files use the same schema with the time column named `time_s`. An empty `cost` means no feasible solution was found within the cutoff.

## Requirements

- g++ 7 or newer (compilation flag `-O3`)
- Python 3.8+ with `numpy` (`pandas` optional, used only for faster graph loading)

## Build and run

### FAME_M2CDS

**Note on the engine library.** The Python bridge `solver/cpp_bridge.py` loads the compiled engine from `FAME_M2CDS/solver/cpp_engine/`, while the engine sources are shipped under `FAME_M2CDS/cpp_engine/`. Compile the engine once and write the shared library to the expected path:

```bash
cd code/FAME_M2CDS
mkdir -p solver/cpp_engine
g++ -O3 -shared -fPIC cpp_engine/engine_m2cds.cpp -o solver/cpp_engine/cpp_engine_m2cds.so
```

Use `cpp_engine_m2cds.dylib` as the output name on macOS and `cpp_engine_m2cds.dll` on Windows.

Batch mode over a task CSV (columns `dataset,instance,seed`):

```bash
cd code/FAME_M2CDS
python -m benchmark.run_benchmark --problem M2CDS \
    --task_csv tasks.csv \
    --data_root /path/to/instance \
    --outdir results \
    --cutoff 600 --max_batch 48 --hard_cutoff --anytime
```

Standalone command-line solver (no Python required):

```bash
cd code/FAME_M2CDS
g++ -O3 cpp_engine/m2cds_solver_main.cpp cpp_engine/engine_m2cds.cpp -o m2cds_solver
./m2cds_solver <instance_file> <seed> <cutoff_s> <output_file>
```

### Comparison algorithms

Each directory under `comparison_algorithms/` builds a standalone binary:

```bash
cd code/comparison_algorithms/GRASP
g++ -O3 main.cpp tarjan.cpp -o grasp
./grasp <instance_file> <seed> <time_limit_seconds>
```

The same commands apply to `GFTCDS`, `GRASP_MAR`, `MSLS`, and `RLS_Tabu`. Every solver prints a final line of the form `RESULT <instance> <cost> <size> <time>`.

## Acknowledgments

The benchmark instances distributed in `instance.zip` were collected from publicly available sources on the Internet. We gratefully thank their authors and maintainers.

## Citation

The paper is currently under review. If you use this code or data in your research, please cite the paper once it is published. The citation entry will be updated here after publication.

## License

This project is licensed under the MIT License. See [LICENSE](LICENSE) for details.
