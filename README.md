# CellSim

**CellSim simulates living cells and how they respond to drugs.** Each
simulated cell grows, copies its DNA, divides, repairs damage or dies,
and a drug changes those decisions from inside the cell. You can walk
into a 3D lab and use it, run experiments on a web page, or call it from
Python.

Free and open source (MIT). Runs on an ordinary laptop, no graphics card
needed.

[![The 3D lab: a microscope bench and an incubator, with daylight through the windows](web/lab3d/screenshot.jpg)](https://rockman6.github.io/CellSim/lab3d/)

## Try it in your browser

Nothing to install: the engine runs on your own computer, and nothing is
uploaded. The first visit downloads Python and the engine (about 40 MB),
so allow a minute or two.

| | |
|---|---|
| **[Walk into the lab →](https://rockman6.github.io/CellSim/lab3d/)** | A biology lab in first person. Prepare slides and use the microscope, or treat living cancer cells and count them. Needs a keyboard and mouse. |
| **[Run an experiment →](https://rockman6.github.io/CellSim/lab/)** | Pick cells, a drug and a dosing schedule, press Run, and see how the colony responds. |
| **[Watch inside one cell →](https://rockman6.github.io/CellSim/lab/cell.html)** | Every molecule the engine tracks, minute by minute, as one cell divides, stops or dies. Type any drug's name, or design your own. |
| **[Open a simulated tumour →](https://rockman6.github.io/CellSim/)** | 3D tumour spheroids grown by the engine, simulated again every time the site is updated. |

## The 3D lab

Walk with **W A S D** and the mouse. **Left-click** picks things up,
**right-click** uses them, and **1–9** choose what's in your hand. Press
**F** and type a name to find anything (spelling mistakes are fine).
Each experiment gives you its steps one at a time.

| Experiment | What you do | The cells |
|---|---|---|
| Onion skin | Peel one layer of cells, stain it with iodine, focus up to 400× | Drawn |
| Your cheek cells | Swab your cheek, smear it on a slide, stain it blue | Drawn |
| Pondweed leaf | Watch the chloroplasts stream round each cell | Drawn |
| Mitosis in a root tip | Soften a root tip in warm acid, stain and squash it, then name the stage of each cell | Drawn |
| Osmosis in red onion | Draw salt water under the cover slip and watch the cells shrink from their walls | Drawn |
| Living cancer cells | Treat a dish of lung cancer cells with a drug, incubate it, compare it with an untreated dish | **Simulated** |
| Dose–response curve | Pipette rising doses into a 6-well plate, incubate for 72 h, read off the IC50 | **Simulated** |
| Counting cells | Stain cells with trypan blue and count the live and dead ones in a hemocytometer | **Simulated** |

*Drawn* slides show cells at their real sizes but are not simulated.
*Simulated* cells come from the CellSim engine, cell by cell. Graphics
settings run from older laptops to fast graphics cards. Source,
controls and credits: [`web/lab3d/`](web/lab3d/).

## What the engine does

- **Each cell runs its own machinery:** a cell cycle (G1, S, G2, M), a
  DNA-damage alarm (the ATM–p53–MDM2 circuit), and the switch that makes
  a badly damaged cell kill itself (apoptosis).
- **Drugs act from inside the cell.** How fast a drug gets in, is pumped
  out and binds sets its concentration inside each cell, and that drives
  the damage and the decisions.
- **Cells live in a dish.** Mixed freely, one simulation gives a whole
  dose–response curve. Placed on a grid, they grow as a flat layer or as
  a 3D spheroid, with oxygen and drug seeping in from outside, cells
  short of oxygen further in, and a dead core.

## What it is good for, and what it isn't

**Good for questions about one kind of cell:** whether to give a drug as
a short pulse or keep it on, how long cells take to recover once the drug
is washed out, whether two drugs help or hinder each other, which cells a
treatment leaves behind, and how deep a drug gets into a tumour.

**Not for ranking cell lines against each other for ordinary
chemotherapy.** This was measured, not assumed: eleven standard markers,
fitted as well as possible across 156–683 cell lines, explain only
9–20 % of the differences between lines.

**The exception is a drug whose target is part of the modelled
machinery.** An MDM2 inhibitor works by stabilising p53, and for it the
engine calls all eleven test lines correctly, where always guessing the
same answer gets five. That includes HeLa, whose p53 gene is normal but
whose p53 protein is destroyed by a virus protein (HPV E6) instead of by
MDM2.

## How far to trust it

- **Every fitted number is named and counted:** one potency constant per
  drug and one spatial constant for the dish. Everything else is a
  published measurement, cited where it is used.
- **It is checked against real data.** Each drug's potency comes out about
  right: for cisplatin on A549 cells the engine gives an IC50 of 10.9 µM
  against 9.8 µM measured, and 20 of 25 predictions it was not fitted to
  fall inside the spread of repeated lab screens. Colony growth matches a
  real HeLa time-lapse within 3–4 % at 46 h, and oxygen and drug
  diffusion match exact solutions.
- **The misses are published too.** The spheroid's dead core comes out
  too large, and real HeLa cells split into fast dividers and
  non-dividers, which the engine does not do. Results are re-measured
  across random seeds and doses before anything is built on them; during
  development one was withdrawn and another narrowed.
- **An IC50 from the engine moves 10–20 % between random seeds,** because
  it depends on which resistant cells happened to be drawn. Two IC50s
  closer than that are not different; `ic50_spread()` measures it.

Every number, with how to reproduce it and its caveats:
[`docs/VALIDATION.md`](docs/VALIDATION.md).

## Use it from Python

The cell simulator needs only NumPy and SciPy, so it installs with plain
pip: no conda, no compilers. (CI checks this on a bare virtual
environment: `tests/test_pip_install.py`.)

```bash
pip install cellsim                 # or: pip install -e . from a checkout

# A dose-response curve and an IC50 for one cell line and drug.
cellsim cell-sim --line A549 --drug cisplatin --hours 72

# Grow a spheroid for five days and watch oxygen run out in its core.
cellsim dish --line DLD-1 --geometry spheroid --hours 120 \
    --cells 3000 --grid 48 --every 8 --out spheroid.jsonl

# Then open spheroid.jsonl in web/viewer/index.html with its
# "Open .jsonl" button (no server needed for a local file).
```

In a notebook, `cellsim.api` returns tidy tables (a `pandas.DataFrame`
when pandas is installed, a list of dicts otherwise):

```python
from cellsim.api import curve, exposure, washout, combination, spheroid

curve("A549", "paclitaxel")               # viability against concentration, with the IC50
exposure("A549", "paclitaxel")            # the concentration for half kill at each
                                          # exposure time: the scheduling question
washout("A549", "cisplatin", 20.0, 12.0)  # a 12 h pulse, then recovery
combination("A549", "cisplatin", "paclitaxel", 10.0, 0.03)   # against independent action
spheroid("DLD-1", days=6)                 # size, hypoxia and necrotic core over time
```

`exposure()` returns `inf` where no concentration reaches half kill.
That is the honest answer for a taxane given for three hours, and the
reason total exposure (AUC) is the wrong measure for that class of drug.

[`docs/cell_tutorial.md`](docs/cell_tutorial.md) works through one real
question end to end, *should I pulse this drug or leave it on?*, with
every command and its actual output.

## Docking and the molecular tools

CellSim also has a molecular layer: docking, drug-likeness (ADMET) and
quantum descriptors. It feeds the cell engine as an input, labelled with
where each number came from and how accurate that kind of number has
proved to be. It is not the main product. It needs the conda
environment:

```bash
# 1. One-time setup (conda or mamba).
mamba env create -f environment.yml
conda activate cellsim
pip install -e .                    # registers the `cellsim` command
cellsim doctor                      # expect "42/42 checks passed"

# 2. Screen the bundled 5-compound batch against streptavidin (PDB 1STP).
cellsim dock \
    --smi benchmarks/dock/1stp_batch_5.smi \
    --receptor benchmarks/dock/1stp.pdb \
    --out-csv /tmp/run/report.csv \
    --mc 4 --profile-top-k 3 \
    --crystal-pdb benchmarks/dock/1stp.pdb --crystal-resname BTN
```

You get a ranked CSV (`triage`, `dG_kcalmol`, `Kd_human`, `strain_band`,
`pocket_ok`, ADMET columns) and a one-page profile picture for each top
hit. Leave out `--center` / `--box` for any other receptor and fpocket
finds the binding site. `cellsim help` lists every command;
[`docs/docking_tutorial.md`](docs/docking_tutorial.md) walks through a
full screen and, in §8, says which kinds of target to trust.

## Everything it can do, and how each part was checked

| Part | What you can run | How it is validated |
|---|---|---|
| Cell drug-response engine (`cellsim cell-sim`, `cellsim cell-stream`) | Pick a cell line, a drug and a **dose schedule**; get the colony over time, viability, and a per-cell state stream (JSON Lines) for a UI. Built for *within-line* questions: schedule, wash-out, resistance, combinations | Each drug's potency scale is right (A549 cisplatin 10.9 µM vs 9.8 measured; 20/25 held-out predictions inside GDSC's replicate span). **Ranking one cell line against another is explicitly out of scope**, measured as unreachable from canonical markers ([why](docs/VALIDATION.md)) |
| Spatial dish (`cellsim dish`) and web viewer (`web/viewer/`) | Grow cells as a monolayer or a spheroid under a dose schedule: oxygen and drug fields, contact inhibition, hypoxic quiescence, necrosis, clones. The viewer shows 3D cells coloured by phase, p53, caspase-3, oxygen or lineage, with time scrubbing, charts and CSV export | Field solvers match closed forms (Grimes's 233 µm oxygen limit; the analytic slab profile). Re-running the Cell Tracking Challenge HeLa movie: colony size within 3–4 % at 46 h. Spheroid growth **calibrated** on DLD-1 (one constant). **Open misses:** the necrotic core is too large, and real HeLa splits into fast cyclers and non-dividers, which the engine does not ([details](docs/VALIDATION.md)) |
| Calibrated uncertainty (`scripts/calibrate_cell_uncertainty.py`) | Every predicted IC50 comes with an interval whose coverage has been scored | Leave-one-drug-out: ±2.6× covers 64 % (68 % nominal), ±4.8× covers 85 % (90 %), ±6.5× covers 95 % (95 %). Wide on purpose: GDSC's own replicate screens disagree by a median 5.3× |
| Your own plate (`cellsim plate`) | Read a plate-reader export (grid or tidy), normalise to the plate's own controls, fit a dose-response curve, and get GR metrics comparable across lines | Coverage measured against known IC50s: a nominal 95 % interval contains the truth ~90 % of the time on ten concentrations, and only **68 % on five**, which is flagged. Flat plates are refused; ranges that clip the response are marked extrapolated |
| Knock a gene out (`api.knockout`) | 13 pathway genes, knocked out, knocked down or overexpressed, and what the rest of the network does without them | TP53 loss lands where the p53-mutant lines are; PUMA/BAX/CASP3 knockouts are identical because they are one pathway; ABCB1 protects against doxorubicin and **not** cisplatin, which it does not transport |
| Starve a tumour (`api.starve`) | Glucose as a second diffusing nutrient. Starved cells burn more oxygen (Casciari 1992), so lowering glucose thins the viable rim through hypoxia | **Directions validated** against Mueller-Klieser 1986: the rim thins from 175 to 111 µm across the published 16.5–0.8 mM range. Magnitudes unchecked: the published rim widths are image-only |
| Cells that crawl (`DishParams.migration_*`) | A random walk on the lattice, optionally up the oxygen gradient | Matches D = k·h²/4 to within 1 % at 2 hops/h |
| Immune killing (`api.killing`) | Effector cells on a tumour monolayer. Each contact delivers a *sublethal* hit; a target dies after three within ~50 min, so killing needs several cells working together (Weigelin et al. 2021) | **Mechanism validated**: hits per kill falls from 418 at E:T 0.1 to 4.1 at E:T 3, and lysis responds to the repair window in the right direction. Absolute lysis against a measured killing curve not yet checked |
| Cell-level PK/PD modules (`cellsim/cell`) | Occupancy, permeation, pH trapping, efflux, binding sink, tissue penetration, fate, resistance, clearance, cell cycle, lattice agents | Each module is checked against its analytic limit (22 tests) |
| Docking triage (`cellsim dock`) | SMILES list + receptor PDB → ranked CSV with ΔG ± CI, pose-trust (UFF strain), PoseBusters, ADMET, one triage column | 15-cocrystal blind re-dock, structures and SMILES fetched from RCSB: top-3 pose recovery 87 %, top-1 73 %, PoseBusters validity 100 % |
| Target-class reliability (`cellsim/uq`) | Measured docking error per receptor family, so every ΔG carries an **accuracy** flag, not just seed scatter | trypsin-like 0.9, kinase ATP-site 2.2, ultra-tight binders 5.0 kcal/mol MAE (n = 4–6 each) |
| ADMET descriptors (`cellsim admet`, `profile`) | Lipinski, TPSA, QED, ESOL logS, BBB / hERG / Ames rule flags, one-page profile PNG | Published formulae, cited at the point of use in `cellsim/chem/admet.py` |
| CYP3A4 site of metabolism (`cellsim som`) | xTB C–H bond-dissociation ranking with a heme-accessibility re-rank | 2/3 on the bundled literature set; blind to N-dealkylation. **Advisory only** |
| Alchemical FEP (`cellsim/fep`) | Hydration and binding ΔG scaffolds on openmmtools + MBAR | **Experimental, not a product path.** FreeSolv-12 MAE 1.42 kcal/mol on 10/12 with a size-dependent bias (no barostat). Binding ΔG has never produced a number on a real binder |

## Where it's going

As of October 2026, phases 0–2 of the [plan](docs/PLAN.md) are done and
phase 3 is in review.

| Phase | What it delivers | Target |
|---|---|---|
| 0, done | Fixes merged, CI green, one identity, an honest validation page | Oct 2026 |
| 1, closed with a finding | The single-cell drug-response engine: cell cycle and p53 circuit, death that actually happens, drug uptake on top, calibrated against GDSC. Closed early: the per-line prediction it aimed at was measured to be unreachable | Oct 2026 |
| 2, code done | The dish: a spatial colony with diffusion, drug penetration and contact inhibition, validated on the HeLa movie and calibrated on DLD-1 spheroids, with the web viewer. Outside users still to find | Apr 2027 |
| 3, in review | Ten drugs × ten lines, combinations and evolved resistance, uncertainty with measured coverage, a JOSS paper, conda-forge | Oct 2027 |
| 4, started | **Your own cells:** `cellsim plate` reads a plate-reader export and fits it with a measured error bar; next, fit a line's own simulator constants and predict bands, not lines | 2028 |

Details, what was kept and dropped, and how success is measured:
[`docs/PLAN.md`](docs/PLAN.md).

## For contributors

```
cellsim/
  cell/      the cell engine: cycle, p53, death, drug uptake, the dish
  dock/      Vina + Meeko + PoseBusters + fpocket, batch screen, triage, strain, off-target
  chem/      SMILES → OpenFF Sage + AM1-BCC; ADMET descriptors; profile dashboards
  uq/        Monte-Carlo, Sobol, split-conformal, target-class reliability table
  quantum/   xTB GFN2, CYP3A4 site-of-metabolism (advisory), PySCF hook
  bridge/    binding ΔG → rate-law priors with provenance and accuracy
  cache/     content-addressed SQLite cache for every expensive physics call
  fep/       alchemical ΔG scaffolds (experimental; see VALIDATION.md)
  md/        OpenMM ligand / protein MD helpers
benchmarks/  every input needed to reproduce a number in docs/VALIDATION.md
tests/       one folder per module; each file is a standalone gate
scripts/     the `cellsim` command, fetchers, benchmark runners, doctor
docs/        plan, validation, tutorials, archive of the 2026 campaign docs
web/         the site: lab3d/ (the 3D lab), lab/ (the Lab and Inside a cell), viewer/ and demo/
OLD/         the frozen 2026 C++/Metal prototype and its headless validators (see OLD/README.md)
```

Every push and pull request runs [`smoke.yml`](.github/workflows/smoke.yml):
54 gates across chem, MD, docking, FEP scaffolds, quantum, UQ, bridge,
cell and cache, about 15 minutes on a 2-core runner. A regression in any
gate blocks the merge. Numbers that take hours (FEP sampling, scale
tests) run only on request and are reported in
[`docs/VALIDATION.md`](docs/VALIDATION.md) with the commit that produced
them.

## History, briefly

Version 1.0 (March 2026) was a real-time C++/Metal 3D cell simulator
with drug kinetics. In April 2026 the project restarted around a
non-AI molecular stack, after a critique of circular validation. The
July 2026 fixes corrected the docking numbers and the FEP binding path.
In October 2026 it was refocused on the cell engine and the dish, and
the 3D lab became its front door. The original prototype is kept in
[`OLD/`](OLD/) as a biology reference; it still builds and passes its
headless benchmarks. The full record, including the original roadmap
and every benchmark post-mortem, is in
[`docs/archive/campaign1/`](docs/archive/campaign1/).

## License

MIT, see [`LICENSE`](LICENSE). Third-party attributions are in
[`THIRD_PARTY_NOTICES.md`](THIRD_PARTY_NOTICES.md).
