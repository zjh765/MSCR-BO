# MSCR-BO: code and experimental data

This repository brings together the frozen `joint_selection_v1` MSCR-BO code and the data behind the source-quota study, component ablations, standard-problem comparison, and engineering comparisons.

For a GitHub repository, place the **contents of this archive at the repository root** so that `README.md`, the three reproduction scripts, `code/`, and `data/` are directly visible. Uploading only the ZIP file prevents reviewers from browsing individual files on GitHub.

## Start here: entry points and reproduction

Run these commands **from the repository root**. Python 3.12 is recommended; archive verification needs only Python 3.9 or newer.

```bash
python verify_archive.py
python -m pip install -r requirements_analysis.txt
python reproduce_score.py --experiment standard_comparison_10
python reproduce_score.py --experiment independent_recheck_5
python reproduce_score.py --experiment pooled_standard_15
```

`verify_archive.py` checks file hashes, the 1,280-row index, all 114-evaluation runs, and matched initial archives. Success prints `"status": "PASS"`. `reproduce_score.py` prints the directed equal-weight IGD, GD, fixed-reference HV and Spacing composite, paired 95% bootstrap interval, and one-sided paired label-swap result. The commands leave the archived data unchanged. To restore all cumulative MSCR-BO round files in an **empty directory outside this repository**, run:

```bash
python reconstruct_rounds.py --output ../mscr_restored_rounds
```

This reconstructs `round_00.csv` through `round_12.csv` for each run without rerunning objective evaluations.

### Dependencies by task

| Task | Requirements |
|---|---|
| Verify files and paired runs | Python 3.9+; standard library only |
| Recompute composite scores and uncertainty | Python 3.12 recommended; NumPy from `requirements_analysis.txt` |
| Restore cumulative round files | Python 3.9+; standard library only |
| Rerun the original optimization campaigns | MATLAB R2020a; the recorded Python environment (NumPy 2.5.4, PyTorch 2.11.0+cu130, BoTorch 0.12.0, GPyTorch 1.13, SciPy 1.18.1, Matplotlib 3.9.2, and `pymoo` 0.6.1.3 for EGBO); external benchmark implementations and configured local paths |

The standard MSCR-BO optimization entry point is `code/original_runners/MSCR_BO_新算法顺序实验_20261111/run_main_case.m`. The frozen core is `code/algorithm/MSCR_BO_staged.m`. Other executed campaign runners are under `code/original_runners/`. These source snapshots retain machine-specific Windows paths, so a full optimization rerun requires path configuration and benchmark installation. Verification and score recomputation above run directly from this package.

## Study map

| Study | Problems | Paired seeds | Runs in terminal index |
|---|---|---|---:|
| Source quotas | ZDT1, ZDT3, ZDT4, DTLZ1, DTLZ2, MW3, MW5, MW7; four 144-proposal allocations | 0–9 | 320 |
| Component ablations | Same eight problems; full algorithm and seven controls | 20–29 | 640 |
| Main comparison | Same eight problems; MSCR-BO and author EGBO | 10–19 | 160 |
| Engineering comparison | CEC 2021 RCM21, RCM27, RCM02; MSCR-BO and EGBO | 60–69 | 60 |
| New-seed paired recheck | Eight standard problems plus RCM21 and RCM27; both methods | 100–104 | 100 |

The selected allocation is **5:8:5** blocks of eight proposals from conditional diffusion, dispersed multiscale elite walking, and acquisition optimization, respectively. Each run begins with 18 shared true evaluations and adds 12 batches of eight, for **114 true evaluations**. The source pool contains 144 proposals per round. The acquisition uses raw joint qLogNEHVI with 16 Monte Carlo samples and a fixed reference point. The selected batch passes through constraint-aware selective repair where constraints exist. See the locked `protocol.json` files under `data/derived/` for the experiment-specific rules.

## Where to start

- `data/derived/canonical_terminal_results.csv`: 1,280 uniformly named terminal records and links to the originating run records. Prefix a `source_file` value with `data/raw/` when it points to a packaged MSCR-BO record. Some comparison rows point to the packaged summary CSVs under `data/derived/`.
- `data/derived/report_consistent/`: metric table and analyses using the common fixed HV evaluation box used in the manuscript. These are the **publication-analysis values** for quota, ablation and the ten-seed standard comparison.
- `data/raw/`: successful MSCR-BO runs with one cumulative `round_12.csv` (all 114 true evaluations), statuses, selected archive fronts, and selection logs. The earlier round files were exact prefixes of the final file in all 1,120 runs. `reconstruct_rounds.py` restores them on demand. Superseded failed attempts are excluded.
- `data/egbo/`: EGBO's observed training designs, objective values, constraints, author HV traces where available, and numerical run settings.
- `data/initial/`: exact shared 18-observation archives for every used case and seed.
- `data/reference/`: fixed acquisition references and evaluation fronts. Engineering `.mat` files contain the locked numerical benchmark front.
- `code/algorithm/`: one byte-identical frozen core shared by all three campaigns. `code/algorithm_helpers/` contains the MSCR-BO helper and bridge sources; `code/original_runners/` preserves the executed campaign and analysis scripts.
- `code/egbo/author_source_snapshot/`: the two EGBO source files and the upstream MIT `License.md`. Both source files match the upstream repository byte for byte. `code/egbo/local_experiment_adapters/` contains our MATLAB evaluator and experiment runner. See `EGBO_SOURCE_PROVENANCE.md` for links and hashes.
- Figures are omitted from this upload-sized package. The raw evaluations, reference fronts, metric trajectories and plotting source remain available for regenerating figures.
- `MANIFEST.csv`: SHA-256 digest and size for every packaged file, including documentation and verification code.
- `RIGHTS_NOTICE.md`: bilingual authorship, provenance and rights-contact notice. It is not a project license; licensing terms for original MSCR-BO code and experiment data have not been assigned in this archive.

`RCM02` is retained for a complete audit of the executed engineering campaign; the manuscript reports RCM21 and RCM27. The package includes all completed results regardless of which figures appear in the paper.

## Verify and recompute

From this folder, run `python verify_archive.py` with Python 3.9 or newer. This checks the manifest, 1,280-row index, every 114-evaluation run, and shared initial archives without MATLAB or extra Python packages.

Run `python reconstruct_rounds.py --output <directory>` to restore the 13 cumulative CSVs per MSCR-BO run in a separate directory. The script only slices stored rows; it does not rerun any objective evaluations.

With NumPy installed, run `python reproduce_score.py --experiment standard_comparison_10`, `--experiment independent_recheck_5`, or `--experiment pooled_standard_15`. The script recomputes the directed equal-weight four-metric score, 20,000 paired bootstrap resamples, and 100,000 paired label swaps. Results are stochastic but reproducible with the stored RNG seeds. The standard ten-seed point estimate is **0.1174043469250529**; the independent five-seed point estimate is **0.10128503927854829**.

The raw `run_status.json` files preserve the run-time metric values. An earlier summary calculated HV with a different evaluation box on some cases and gave a ten-seed score of 0.126647. The manuscript uses the common-box recomputation (0.117404); the earlier summary is intentionally not presented as a publication result. `corrected_archive_metric_audit.json` gives the corrected quota and ablation comparisons.

## Full optimization reruns and rights

The original runner scripts are exact source snapshots and still refer to machine-specific Windows paths. Rerunning the optimizer also needs the benchmark implementations and MATLAB/Python environments documented in `PUBLICATION_CHECKLIST.md`; this folder currently supports independent data verification and score reproduction, while full optimization reruns require those dependencies to be installed and paths configured.

No license has been assigned in this archive to original MSCR-BO code or experiment-generated data. The included EGBO upstream source retains its separate MIT license and attribution. External benchmark materials retain their own rights. A fixed GitHub release tag or archival DOI should identify the version cited in a manuscript.
