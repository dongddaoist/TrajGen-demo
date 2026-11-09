# trajgen-demo — atomistic electrolyte frames from a formulation

Milestone-M1 demo of **traj-gen**, the two-stage generative model of MD configurations for
Li-ion battery electrolytes simulated with **MU-PFF** (SES AI's polarizable force field).
Choose a formulation; Stage 1 places every molecule's centre of mass (box from a predicted
density); Stage 2 grows all atoms around those centres and applies a time-free correction.
You get an all-atom frame (`.pdb/.gro/.xyz`), a topology assembled from the 69-species
MU-PFF library, an MD-input folder in the corpus layout, and a report. **No molecular
dynamics is run.**

INTERNAL: checkpoints, registries and force-field files are SES AI internal material.

## Install (one command)

Needs micromamba, mamba or conda. From this folder:

    ./install.sh            # CPU
    ./install.sh --gpu      # CUDA build of pytorch (NVIDIA driver required)

This creates the `trajgen-demo` environment, installs the package and runs
`trajgen-demo check` (bundle and code hashes, library versions, device).

**Other machines and clusters (CPU or GPU)** — offline conda-pack, Apptainer + Slurm,
Docker + Kubernetes, the UI over an SSH tunnel, batch files: see [`LAUNCH.md`](LAUNCH.md).
Without installing anything into Python: `./trajgen-demo <command>` runs from this folder.

## Use

    conda run -n trajgen-demo trajgen-demo app                 # web UI at http://127.0.0.1:7860
    conda run -n trajgen-demo trajgen-demo examples            # corpus formulations with MD numbers
    conda run -n trajgen-demo trajgen-demo run --example F1024 --compare-md --out runs/f1024
    conda run -n trajgen-demo trajgen-demo run --compose EC=280 EMC=420 LI=60 PF6=60 --out runs/mix
    conda run -n trajgen-demo trajgen-demo run --salt LI:PF6:1.0 --solvents EC=3 EMC=7 --basis wt --out runs/r
    conda run -n trajgen-demo trajgen-demo check --smoke       # tiny end-to-end pass + timing estimate
    conda run -n trajgen-demo trajgen-demo batch formulations.json --out runs/b1   # many formulations, resumable

Species names are the library's: `trajgen-demo species`. A recipe is solved in real numbers
and rounded; the report states the realised concentration.

## Timing

Measured on corvex (one H200): Stage 2 takes ~6 s per 10k-atom frame for the flow plus
seconds for the correction; Stage 1 generates ~4,500 ns/day. On a CPU expect minutes per
frame; `trajgen-demo check --smoke` prints an estimate for your machine (scaled from a
token run, labelled as an estimate) and every report records the measured time.

## What the numbers mean (milestone M1)

On 50 held-out corpus systems with true centres of mass, Stage 2 gives Li–O coordination
3.93 vs MD 3.92, conformer populations within JS 0.009 of MD, no overlaps, and packing
+15 kJ/mol per molecule above an identically minimised MD frame. From the composition
alone (Stage 1 + Stage 2, what this demo runs), Li–O coordination is ~10 % low (3.49 vs
3.83: Stage 1 over-pairs Li⁺ with PF₆⁻) and packing +32 kJ/mol per molecule; conformers,
contacts and overlaps stay at the true-COM level.

Caveats, printed in every report: energies use MU-PFF without the induced-dipole term;
the correction quenches bond/angle vibration; frames are independent samples; species
outside the 69 are not supported in this version; sizes and temperatures far from the
corpus (~10k atoms, its temperature set) are untested and flagged.

## Outputs of a run

| file | content |
|---|---|
| `frame_000.pdb/.gro`, `frames.xyz` | all-atom frame(s); molecules whole, COMs in the box; `.itp` atom order |
| `topology/system.top` + `*.itp/*.atp` | bonded terms and charges; `[ defaults ]` verbatim from the corpus |
| `topology/*.json`, `*_nb_params.json` | the MU-PFF nonbonded parameters (exp-6, C6, charge transfer, polarisability, Thole) |
| `md_input/` | the corpus `transport_params/` layout + `transport_config.json`, frame 0 as the start box (compatibility with the MD protocol not yet verified) |
| `stage1_coms.npz` | Stage 1 centres of mass (fractional) and box |
| `report.md`, `report.json` | composition, state, every metric, settings, timing, caveats |

## Provenance

`bundle/MANIFEST.json` — every bundle file's sha256, the corvex inputs and code stamp.
`vendor/VENDOR.json` — the traj-gen source files used, byte-identical to the repository.
`BUILD.json` — which bundle and which code this folder was built from.
