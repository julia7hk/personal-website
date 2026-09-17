---
title: "CVIRL Work - SLURM Jobs and Purdue Gilbreth"
date: "2026-07-13"
description: "Accessing Gilbreth over ssh, understanding what SLURM actually is, and running srun / sbatch jobs"
dayNumber: 2
---

my lab work is deep learning on echocardiogram videos. running the model on my macbook was way too slow, so everything moved to Gilbreth.

this is a quick reference for it.

### what is gilbreth

Gilbreth:  one of Purdue RCAC's community clusters, the GPU one

you don't own the machine. you ask a scheduler for resources, and it gives them to you when there's room.

SLURM:  Simple Linux Utility for Resource Management. the scheduler. 

mental model:

- **login node** — what you ssh into. shared by everyone. for editing files, git, and submitting jobs. that's it
- **compute node** — where the gpus are. you never ssh here directly, you only get here through slurm

‼️ do not run your training script on the login node. it gets killed and you're hogging a machine ~100 other people are typing on

---

### getting in

```bash
ssh <username>@gilbreth.rcac.purdue.edu
```

wants password + 2 factor every single time, which gets old fast.

add your public key so it stops asking:

```bash
ssh-copy-id <username>@gilbreth.rcac.purdue.edu
```

then in `~/.ssh/config` on my laptop:

```
Host gilbreth
    HostName gilbreth.rcac.purdue.edu
    User <username>
    ForwardAgent yes
```

now it's just `ssh gilbreth`

**agent forwarding:**  lets the server use my ssh key to connect somewhere else, without ever copying my private key onto the server

that's the thing that makes `git clone git@github.com:...` work while i'm on gilbreth. the cluster borrows my laptop's key to talk to github.

---

### where files go

three storage spots and they are NOT interchangeable. this confused me for a while.

| location | what it's for | notes |
| --- | --- | --- |
| `/home/<username>` | your code, configs, envs | small quota (25GB for me) |
| `/scratch/gilbreth/<username>` | big intermediate outputs | huge, but **purged** periodically. not backed up |
| `/depot/<lab-account>` | shared lab data | whole lab shares one quota |

check what you've used:

```bash
myquota
```

```
Type       Location       Size    Limit    Use
================================================
home       <username>   17.1GB   25.0GB  68.6%
scratch    <username>    4.0KB  200.0TB   0.0%
depot      <lab>          3.9TB    4.0TB 100.0%
```

depot at 100% is how my inference run died halfway through 💀

find what's eating it:

```bash
du -h --max-depth=1 | sort -h
du -sh /path/to/dir/* 2>/dev/null | sort -rh | head -20
```

- `-h` human readable (G, M, K instead of blocks)
- `-s` summarize, just the total per arg
- `sort -rh` reverse sort, human-numeric aware
- `2>/dev/null` throw away permission-denied noise

easy wins:

```bash
rm -rf ~/.cache/pip
```

lesson learned: **big model outputs belong in `/scratch`, not `/depot`.** depot is shared with the whole lab, so filling it blocks everyone, not just me.

---

### environment

gilbreth uses environment modules, so nothing is on your PATH until you ask for it

```bash
module load conda
conda activate <env-name>
```

a plain venv works too:

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

what bit me: my laptop's `requirements.txt` was full of mac-only pins and would not resolve. i rewrote a much simpler one with only what the script actually imports.

‼️ `module load` has to go **inside your job script too**, not just in your terminal. the compute node starts with a fresh shell and doesn't inherit anything

---

### interactive job (srun)

for when you want to watch it run, or you're debugging

```bash
srun --cpus-per-task=16 --mem=32G --time=1:00:00 \
  -A <your-account> -p a100-40gb --gres=gpu:1 \
  python prepare_dataset.py --npz-dir ... --out-dir ...
```

what each flag means:

- `-A` / `--account` — which allocation gets billed. your lab's queue
- `-p` / `--partition` — which pool of machines. picks your gpu type (`a100-40gb`, `a10`, ...)
- `--gres=gpu:1` — **g**eneric **res**ource. this is the line that actually gets you a gpu. forget it and you get a cpu-only node and wonder why everything is slow
- `--cpus-per-task` — cpu cores. matters more than you'd think, dataloader workers live here
- `--mem` — RAM (not vram)
- `--time` — walltime limit. job gets killed at this mark, so pad it

a preprocessing step that was crawling on the login node finished almost immediately once it had real cores.

---

### batch job (sbatch)

the normal way. write a `.sh`, submit it, close your laptop, come back later.

`run_job.sh`:

```bash
#!/bin/bash
#SBATCH --account=<your-account>
#SBATCH --partition=a100-40gb
#SBATCH --gres=gpu:1
#SBATCH --cpus-per-task=16
#SBATCH --mem=64G
#SBATCH --time=8:00:00
#SBATCH --job-name=finetune
#SBATCH --output=logs/%x_%j.out
#SBATCH --error=logs/%x_%j.err

module load conda
conda activate <env-name>

cd $SLURM_SUBMIT_DIR
python finetune.py --epochs 100 --patience 15
```

- `#SBATCH` lines are comments to bash and directives to slurm. they must be at the top, before any real command
- `%x` = job name, `%j` = job id. so logs don't overwrite each other
- `$SLURM_SUBMIT_DIR` = the directory you ran `sbatch` from

submit:

```bash
sbatch run_job.sh
```

it prints a job id and returns immediately. that's the whole point.

chain a few at once:

```bash
sbatch job_a.sh && sbatch job_b.sh && sbatch job_c.sh
```

all three queue up and run whenever gpus free up. i submitted three training variants before bed and read all three logs in the morning 🎉

---

### one-off jobs without writing a file

`--wrap` takes a command string instead of a script:

```bash
sbatch --account=<your-account> --partition=a10 --gres=gpu:1 \
  --cpus-per-task=4 --mem=165G --time=4:00:00 \
  --wrap="python $(pwd)/extract_landmarks.py --inference-root /path/to/outputs"
```

use `$(pwd)` so the path is absolute. the compute node does not start where you are.

---

### checking on jobs

```bash
squeue -u $USER          # my queue
scancel <jobid>          # kill one
scancel -u $USER         # kill all mine
```

`ST` column is the one to read:

- `R` running
- `PD` pending — waiting for a free gpu. totally normal, not an error
- `CG` completing

my first few "why isn't it running" panics were just `PD`. someone else had the gpu. nothing to fix, just wait.

after it finishes:

```bash
sacct -j <jobid> --format=JobID,JobName,State,Elapsed,MaxRSS
seff <jobid>             # efficiency summary, did i over-request
```

---

### long jobs that aren't slurm jobs

moving a couple TB between filesystems isn't a compute job, but it takes hours and dies the second your ssh drops.

`screen` keeps the session alive on the login node:

```bash
screen                   # start
# ctrl-a then d          -> detach, walk away
screen -rx               # reattach
screen -X -S <id> quit   # kill a stuck one
```

related habit i picked up the hard way: **put a `--resume` flag in any script that loops over a big file list.** mine died at ~1000/1172 outputs on a full disk, and without `--resume` that's the entire run again.

---

### getting data off the cluster

first instinct was rsync:

```bash
rsync -avzP <username>@gilbreth.rcac.purdue.edu:/path/on/cluster /local/destination/
```

- `-a` archive (keeps permissions, timestamps, recurses)
- `-v` verbose
- `-z` compress in transit
- `-P` progress bar + resume partial files

totally fine for a few GB. for 1.2TB it quoted me **36 hours** over my laptop's connection, so i killed it.

**Globus** instead:

- install Globus Connect on the machine holding the drive, register it as a collection
- register the external drive as its own collection
- queue the transfer in the Globus web UI

it's server-to-server, so you can close the laptop and it keeps going. restarts itself on failure. both folders landed over the weekend.

rule i'd give myself: **under ~50GB use rsync, above that use Globus.**

---

### stuff that cost me the most time

- ran a heavy script on the login node before i understood what a login node was
- forgot `--gres=gpu:1` and quietly ran on cpu
- `module load` only in my shell, not in the job script, so the job couldn't find python packages
- relative paths inside a job script — job starts elsewhere, paths break, job fails 3 seconds in after queueing for an hour
- writing multi-TB outputs to shared `/depot` instead of `/scratch` and filling the lab's quota
- no `--resume`, so a crash at 90% meant starting over

basically every single one is "the compute node is a different computer than the one you typed on." once that sank in most of it stopped happening.
