# bc2: run BindCraft2 on Argon

New to this? The [interactive beginner's guide](https://schnickn.github.io/bindcraft2-image/) walks through every step.

`bc2` is one terminal script that sets up BindCraft2 on Argon, asks you a few questions instead of making
you edit JSON and job scripts, submits the job, and shows you the results. It is built for people running
BindCraft2 for the first time.

You need an Argon account and a login node. Nothing else: the script uses only `bash` and the system `python3`.

## Install the script (once)

On an Argon login node:

```bash
mkdir -p ~/bin && cd ~/bin
curl -fsSLO https://raw.githubusercontent.com/schnickn/bindcraft2-image/main/bc2
chmod +x bc2
echo 'setenv PATH $HOME/bin:$PATH' >> ~/.tcshrc      # most Argon accounts use tcsh; for bash use ~/.bashrc and: export PATH=$HOME/bin:$PATH
```

Log out and back in (or run `setenv PATH $HOME/bin:$PATH` once) and check with `bc2 help`.
`bc2` runs under bash by itself, so you do not need to change your login shell.

## One-time setup

```bash
bc2 setup
```

This submits a batch job that downloads the container image (about 7 GB) and the AlphaFold model weights
(about 5 GB) into `~/bc2`. It takes a while; the command prints the log to watch. When the log says
`setup finished`, run:

```bash
bc2 check      # should show only green checks
bc2 test       # optional: a tiny run to prove everything works end to end
```

If someone already shared a container file or the weights folder with you, skip the downloads:
`bc2 setup --sif /path/to/bindcraft2.sif --weights /path/to/folder/with/params_model_*.npz`.

## Design binders

```bash
bc2 new                 # a short question-and-answer session
bc2 status              # watch it
bc2 results NAME        # the best designs
```

`bc2 new` asks for:

1. **A name** for the campaign.
2. **The target.** Pick one BindCraft2 already includes, or point to your own file. For your own file,
   press Tab to complete the path; the script lists the chains it finds, checks the chain you type, and
   checks that your hotspot residue numbers exist.
3. **The binder type** (new miniprotein, nanobody, scFv, peptide, ...) and, for most types, the length range.
4. **Optional properties** such as humanization.
5. **How many accepted designs** you want and how many attempts to allow before giving up.
6. **Where to run it:** GPU type, queue and workers per GPU, with a memory estimate for your target size.

It writes everything into `~/bc2/campaigns/NAME/`, shows you the settings file it created, and offers to
submit. If you are not sure, accept the defaults.

## Everyday commands

| Command | What it does |
|---|---|
| `bc2 new [NAME]` | Guided setup of a campaign |
| `bc2 run NAME` | Submit it. Run it again after any stop and it resumes where it left off |
| `bc2 run NAME --dry-run` | Check the request with the scheduler without submitting |
| `bc2 status [NAME]` | Job state, accepted designs, attempts, last log lines (no name: all campaigns) |
| `bc2 results NAME [N]` | Top N designs, plus the `scp` command to copy them to your computer |
| `bc2 cancel NAME` | Stop a queued or running campaign (progress is kept) |
| `bc2 edit NAME` | Open `settings.json` in nano (`export EDITOR=vim` to change) and check it is valid |
| `bc2 resources NAME` | Change GPU type, queue and workers |
| `bc2 list` | All campaigns |

## Things worth knowing

- **Each campaign is its own folder.** Never point two campaigns at the same results folder.
- **Accepted designs are rare.** Most attempts fail the quality filters, so the attempts cap matters. A hard target can use many GPU hours.
- **GPU choice.** L40S, A40, A100 and V100 have all been tested. The V100 is about 60% slower. On UI-GPU you can have one job running at a time.
- **all.q** starts quickly but can be interrupted at any moment. Run `bc2 run NAME` again and the campaign continues.
- **The first run is slower and uses more memory.** BindCraft2 compiles its GPU code the first time it sees a given GPU type and size, and keeps the result in `~/bc2/xla_cache`. A first single-worker run peaked at about 29 GB of host memory, so keep the default of 8 slots. Later runs on the same GPU type are faster and lighter.
- **Memory.** Bigger targets need more GPU memory per worker; the wizard shows an estimate and how many workers fit on each card. The default of 8 CPU slots (about 32 GB of RAM) covers up to 2 workers; 3 workers uses 12 slots.
- **Where things are.** Settings are in `settings.json`, the generated job in `job.sh`, logs in `logs/`, results in `results/` inside the campaign folder. `results/3_Ranked/` holds the accepted designs.
- **A different install location:** `export BC2_HOME=/some/path` before running any `bc2` command.

## If something goes wrong

- `bc2 check` tells you what is missing.
- A campaign stuck in `waiting` just needs a free GPU: check with `qstat -g c -q UI-GPU`, or pick another GPU type with `bc2 resources NAME`.
- A job in state `ERROR` or a log that stops early: `bc2 status NAME` shows the last lines; `qstat -j JOBID` explains scheduler problems.
- If `bc2 setup` fails because the batch nodes cannot reach the internet, run `bc2 setup --here` inside `screen` instead.
