# BindCraft2 on Argon

Everything a University of Iowa Argon user needs to run [BindCraft2](https://github.com/PacesaLab/BindCraft2)
(protein binder design) without building anything by hand:

- **`bc2`**: a terminal helper that installs BindCraft2, asks you plain questions to set up a design
  campaign, submits it to the queue, and shows progress and results.
- **A container image** with BindCraft2 built for Argon's GPU drivers.
- **A beginner's guide** (a web page, [open it here](https://schnickn.github.io/bindcraft2-image/)) for people who know neither BindCraft nor Argon.

## Quick start (on an Argon login node)

```bash
mkdir -p ~/bin && cd ~/bin
curl -fsSLO https://raw.githubusercontent.com/schnickn/bindcraft2-image/main/bc2
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
<https://schnickn.github.io/bindcraft2-image/>**  It explains each step in plain language, lets you
practice the commands in a simulated terminal, and helps you plan memory for your target. The command
reference is in [docs/bc2.md](docs/bc2.md).

## Why a separate image

Upstream's `containers/Dockerfile` installs the `cuda13` wheels, which need NVIDIA driver >= 580.
Argon's GPU nodes run driver 535 (CUDA 12.2), so the stock image finds no GPU there. `Dockerfile` here is
upstream's file at commit `c0ee638` plus an `ACCEL` build argument (default `cuda12`).
The GitHub Actions workflow (`.github/workflows/build.yml`) builds it for `linux/amd64` and publishes
`ghcr.io/schnickn/bindcraft2:cuda12`, which `bc2 setup` downloads. Run the workflow from the Actions tab to
rebuild, or to build `cuda13` or a different upstream commit.

## What has been tested

On Argon with driver 535: BindCraft2 on L40S, A40, A100 and V100 GPUs; 1, 2 and 3 workers sharing one GPU;
and the `bc2` flow (setup as a batch job, the question wizard, `bc2 test`, real submitted campaigns that
produced accepted designs, status, results, cancel, resume after a stop, edit, resources). Not yet tested:
antibody-type binders (VHH, scFv, Fab), mmCIF, FASTA and multi-chain targets through the wizard, and long
campaigns.

## License note

The container redistributes BindCraft2 binaries. BindCraft2's license permits redistribution for others to
run but forbids offering it as a hosted (SaaS) service. See the upstream repository for the full terms.
