# BindCraft2 on Argon

Everything a University of Iowa Argon user needs to run [BindCraft2](https://github.com/PacesaLab/BindCraft2)
(protein binder design) without building anything by hand:

- **`bc2`**: a terminal helper that installs BindCraft2, asks you plain questions to set up a design
  campaign, submits it to the queue, and shows progress and results.
- **A container image** with BindCraft2 built for Argon's GPU drivers.
- **A beginner's guide** (a web page, [open it here](https://schnickn.github.io/bindcraft2-for-argon/)) for people who know neither BindCraft nor Argon.

## Quick start (on an Argon login node)

```bash
mkdir -p ~/bin && cd ~/bin
curl -fsSLO https://raw.githubusercontent.com/schnickn/bindcraft2-for-argon/main/bc2
chmod +x bc2
echo 'setenv PATH $HOME/bin:$PATH' >> ~/.tcshrc     # bash users: ~/.bashrc and  export PATH=$HOME/bin:$PATH
```

Log in again, then:

```bash
bc2 setup     # one time: downloads the container and AlphaFold weights (a batch job)
bc2 new       # guided questions -> a ready-to-run campaign
bc2 status    # watch it
bc2 results NAME
```

**New to BindCraft or Argon? Start with the interactive guide:
<https://schnickn.github.io/bindcraft2-for-argon/>**  It explains each step in plain language, lets you
practice the commands in a simulated terminal, and helps you plan memory for your target. The command
reference is in [docs/bc2.md](docs/bc2.md).

## Why a separate image

Upstream's `containers/Dockerfile` installs the `cuda13` wheels, which need NVIDIA driver >= 580.
Argon's GPU nodes run driver 535 (CUDA 12.2), so the stock image finds no GPU there. `Dockerfile` here is
upstream's file (unchanged between `c0ee638` and v1.0.4) plus an `ACCEL` build argument (default `cuda12`) and a `biotite<2` pin: biotite 2.0, which upstream does not pin against, breaks the VHH modality.
The GitHub Actions workflow (`.github/workflows/build.yml`) builds it for `linux/amd64` and publishes
`ghcr.io/schnickn/bindcraft2:cuda12-<upstream commit>`. The plain `:cuda12` tag, which `bc2 setup` downloads,
only moves when you run the "Promote image" workflow, which points it at an already-tested tag without
rebuilding (so students get exactly the image that was tested). Run the build workflow from the Actions tab
to build `cuda13` or a different upstream commit or release tag. The Dockerfile also pins `biotite<2`, because
biotite 2.0 breaks the VHH modality and BindCraft2 does not pin it.

## What has been tested

On Argon with driver 535, BindCraft2 1.0.4 on L40S GPUs: 1, 2 and 3 workers sharing one GPU, one job on 4 GPUs
with 2 workers each (8 workers, requested through `bc2 new`), nanobody (VHH) campaigns with 1 and 2 workers, and short runs of scFv, Fab and
ARP binders. BindCraft2 1.0.3 was also run on A40, A100 and V100 GPUs. The `bc2` flow (setup as a batch job, the
question wizard, `bc2 test`, real submitted campaigns that produced accepted designs, status, results, cancel,
resume after a stop, edit, resources) was verified on a real install.

Not yet tested: A40, A100 and V100 with 1.0.4, mmCIF, FASTA and multi-chain targets through the wizard, long
campaigns, and any antibody run long enough to judge design quality. `bc2` can request up to 4 GPUs of one type
on a node; A100 and V100 nodes usually allow only 1, and the scheduler refuses requests it cannot meet.

## License note

The container redistributes BindCraft2 binaries. BindCraft2's license permits redistribution for others to
run but forbids offering it as a hosted (SaaS) service. See the upstream repository for the full terms.
