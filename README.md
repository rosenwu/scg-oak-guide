# SCG + Oak

With thanks to [Stanford’s official SCG guide](https://login.scg.stanford.edu/), [Chuner’s GitHub resources](https://github.com/chunerguo/resources), and James’s GitHub guide.

**SCG** is Stanford’s shared research computing system: you connect to its computers to run analyses. **Oak** is Stanford’s large research storage system, where your lab keeps data and results.

**A few terms before you start**

| Term | In plain language |
| --- | --- |
| Terminal | An app where you type commands to navigate files and run programs. |
| Command line / shell | The text interface and program that interpret your commands. |
| SSH | A secure way to connect your Terminal to another computer, such as SCG. |
| Directory / path | A folder / its address, such as `/home/rosenwu`. |
| Script | A saved file of code or commands that you can run again. |
| Slurm / batch job | SCG’s system for scheduling work / a task submitted to run when resources are available. |
| tmux | A tool that keeps a terminal session on SCG available after you disconnect. |
| Codex / Claude Code | AI coding assistants you use in a terminal to explain, write, or edit code; they can also run commands. |
| ProxyJump | An SSH setting that connects through an intermediate computer to reach another computer. |

**Example account:** `rosenwu` · **SCG home:** `/home/rosenwu` · **Oak:** `/oak/stanford/groups/longaker/rosenwu`

Replace `rosenwu` with your SUNetID. The Oak path `/oak/stanford/groups/longaker/rosenwu` is an illustrative folder; substitute your actual Oak directory. Other project and file names are examples.

**Laptop** = your Terminal. **SCG** = after SSH login.

## 1. Request access

Email [srcc-support@stanford.edu](mailto:srcc-support@stanford.edu) and **CC your PIs**. Include your name, SUNetID, lab/PI, and the lab storage you need.

## 2. Log in from Terminal

**Laptop • macOS/Linux Terminal**

```bash
ssh rosenwu@login.scg.stanford.edu
```

Run `exit` to log out. [SCG connection instructions](https://login.scg.stanford.edu/tutorials/connecting/).

## 3. Make a login shortcut

**Laptop • one-time setup, before logging into SCG.** SSH means **Secure Shell**. Open your local SSH configuration:

```bash
mkdir -p ~/.ssh       # mkdir = make directory; -p creates missing parents and allows an existing folder
chmod 700 ~/.ssh      # chmod = change permissions; 700 gives only you access to this folder
nano ~/.ssh/config   # nano = a text editor; open or create your SSH configuration file
```

Add this block, or edit an existing `Host scg` block instead of duplicating it:

```text
Host scg
    HostName login04.scg.stanford.edu
    User rosenwu
    ServerAliveInterval 60
    ServerAliveCountMax 3
```

| Setting | Meaning |
| --- | --- |
| `Host scg` | The shortcut name you choose; use it in `ssh scg`. |
| `HostName login04.scg.stanford.edu` | The actual server address. Keep this for the login04 SCG node. |
| `User rosenwu` | Your SUNetID for login. |
| `ServerAliveInterval 60` | After 60 seconds without server data, send a connection check. |
| `ServerAliveCountMax 3` | Disconnect after three unanswered checks. |

`nano` is a text editor; `alias` gives a command a shorter name. `chmod` means **change mode** (permissions): `700` restricts the SSH folder to you; `600` lets only you read and write the configuration file.

In nano, save with **Ctrl+O**, Enter, then exit with **Ctrl+X**. Then:

```bash
chmod 600 ~/.ssh/config  # Only you can read and write this file
ssh scg                  # SSH = Secure Shell; connect using your saved shortcut
```

A fixed node keeps your tmux sessions easy to find. If it is unavailable, use `ssh rosenwu@login.scg.stanford.edu`; tmux sessions remain on the original node.

**Optional: type only `scg`.** Put `alias scg='ssh scg'` in your laptop's `~/.zshrc` for zsh, or `~/.bashrc` for interactive bash. Open a new terminal; on macOS login bash, use `~/.bash_profile` if it does not source `.bashrc`. Run `echo "$SHELL"` if unsure which shell you use. Node behavior: [SCG primer](https://login.scg.stanford.edu/scg_primer/).

## 4. Find your way around

**SCG**

```bash
pwd                         # Print current directory
ls                          # List names
ls -lh                      # Show sizes and details
ls -lah                     # Also show hidden files
cd ~                        # Go home
cd /oak/stanford/groups/longaker/rosenwu
cd ..                       # Go up one level (space required)
cd -                        # Return to previous directory
mkdir -p project_example/results
du -sh project_example      # Estimate folder size; can be slow
less notes.txt              # Read an existing text file; q quits
```

**Go straight to your Oak folder • on SCG:**

```bash
cd ~/oak
```

`cd` means change directory, `~` means your SCG home folder, and `oak` is the shortcut created in section 5. Once that shortcut exists, this command takes you to your Oak folder from anywhere on SCG. Run `ls` to see its contents. If you get “No such file or directory,” set up the shortcut in section 5 first.

**Explore the folder structure with `tree` • on SCG:**

```bash
tree -L 2 .           # tree = show a folder diagram; -L 2 limits it to two levels; . = here
tree -d -L 2 ~/oak    # -d = directories only; show folders within your Oak shortcut
tree -a -L 2 .        # -a = also show hidden files and folders
tree -L 2 -I '*.bam|*.fastq.gz' .  # -I = hide names matching these patterns
```



| Command | Full wording or plain-language meaning |
| --- | --- |
| `pwd` | Print working directory — show your current folder. |
| `ls` | List — show files and folders. |
| `cd` | Change directory — move to another folder. |
| `mkdir` | Make directory — create a folder. |
| `du` | Disk usage — report space used by files or folders. |
| `cp` / `mv` | Copy / move (also used to rename). |
| `ln -s` | Link, symbolic — create a shortcut to another path. |
| `rsync` | Remote synchronization — copy or update files locally or between computers. |
| `ps` | Process status — list running processes. |
| `tail` | Show the end of a file; `-f` follows new output. |
| `tmux` | Terminal multiplexer — keep multiple terminal views in a session. |
| `sbatch` / `squeue` / `sacct` / `scancel` | Slurm commands to submit a batch script / view the queue / view job accounting / cancel a job. |

**Options depend on the command.** For `ls -lah`: `-l` = long listing, `-a` = all entries including hidden files, `-h` = readable sizes. For `mkdir -p`, `-p` creates missing parent folders. For `du -sh`, `-s` summarizes and `-h` shows readable sizes. `~` = home; `..` = parent folder. Some command names are names rather than acronyms.

**Use Tab to complete folder names.** For a folder named `project_example` in your current directory, type `cd pro`, then press **Tab** to fill in the rest. Press **Enter** to enter the folder. If several names start with `pro`, type more letters and press Tab again; in many shells, pressing Tab twice shows the matching names. Use ↑ to recall previous commands. Quote paths containing spaces: `cd "folder with spaces"`. `~` means your home on the machine where the command runs. Large recursive scans can burden shared storage; avoid scanning the whole lab unnecessarily.

## 5. Link SCG home to Oak

**SCG • run once.** Check the target exists, then create the shortcut only if `~/oak` is unused:

```bash
if [ ! -d /oak/stanford/groups/longaker/rosenwu ]; then
    echo "Oak path missing or inaccessible; check permissions."
elif [ -e "$HOME/oak" ] || [ -L "$HOME/oak" ]; then
    echo "~/oak already exists; inspect it before changing anything."
    ls -ld "$HOME/oak"
else
    ln -s /oak/stanford/groups/longaker/rosenwu "$HOME/oak"
fi
```

After verifying the link:

```bash
ls -ld ~/oak
cd ~/oak
pwd -P
```

A symlink is a shortcut, not a copy or backup. Editing or deleting files inside `~/oak` changes the files on Oak. It does not grant new permissions.

## 6. Copy large files safely

**SCG • folder-to-folder copy.** Replace these illustrative source/destination folders first. Keep the destination outside the source. Pause writers so the source is stable.

```bash
src="$HOME/oak/project_example/raw"
dst="$HOME/oak/project_example/raw_copy"
ls -ld "$src"
mkdir -p "$dst"

# Preview first; -n means no files are copied.
rsync -rltnv --partial --progress "$src/" "$dst/"

# After checking the preview, copy.
rsync -rltv --partial --progress "$src/" "$dst/"
```

`-rlt` copies directories recursively, preserves symlinks as links, and preserves timestamps. It avoids forcing source ownership or permissions onto shared Oak storage. A trailing slash on the source copies its **contents**. `--partial` retains incomplete files; rerun the same command after interruption. Existing destination files can be replaced; use a new destination for the safest first copy. Do not add `--delete` for an ordinary copy.

**Only selected files • SCG:**

```bash
rsync -rltv --partial --progress \
  "$src/sample_A.fastq.gz" "$src/sample_B.fastq.gz" "$dst/"
```

**Verify before removing any original:** check the transfer finished without errors, then compare content checksums (this reads all files and can be slow):

```bash
rsync -rlnci "$src/" "$dst/"
```

For a stable source, exit status 0 and no itemized differences indicate matching source contents at the destination. Extra destination files are not reported by this check. Symlinks remain links, not independent copies of their targets. Retain originals until verification and your lab's retention requirements are satisfied.

**Laptop → Oak • run on your laptop after setting up `ssh scg`:**

```bash
rsync -rltvn --partial --progress ./mydata/ \
  scg:/oak/stanford/groups/longaker/rosenwu/incoming/
```

`mydata` and `incoming` are example folders. Ensure the remote destination is permitted; inspect the preview, then remove `n` from `-rltvn` to copy. This transfer depends on your laptop staying awake and connected. For very large transfers across systems, consider Globus. SCG's `/labs` and `/projects` permission recipe differs from this Oak example; see [SCG data movement](https://login.scg.stanford.edu/tutorials/data_movement/).

**Small copies and moves • SCG:**

```bash
cp -i notes.txt notes_backup.txt      # Copy; ask before overwriting
mkdir -p archive
mv -i sample_A.txt sample_B.txt archive/   # Move both; ask before overwriting
```

`mv` removes the old location after a successful move. Within one filesystem it is usually a quick rename; across filesystems it becomes a copy-and-remove operation. For big cross-filesystem moves, prefer rsync, verify, then deliberately remove originals later. `cp` has no built-in transfer resume.

## 7. Codex CLI and Claude Code

**Check access before installing.** Not everyone has the same eligibility or an activated account. Check Stanford’s [Claude for Education](https://uit.stanford.edu/service/claude) and [ChatGPT Edu](https://uit.stanford.edu/service/openai-chatgpt-edu) pages for account requests and current access details. Both list free Standard access for active students, faculty, postdocs, and staff, with a separate affiliate request route. Use the Help request link on either page to ask IT to confirm your affiliation and access to Codex or Claude Code.

**For residents:** You may need to contact Virginia Ford at [vmford@stanford.edu](mailto:vmford@stanford.edu) for help setting up a postdoc affiliation. I'm happy to help if you have more questions.


**Recommended starting point: your laptop, inside a folder of approved code.** These tools can read files, send context to external services, edit code, and run commands. Use only Stanford/lab-approved accounts and data. Do not expose PHI, restricted datasets, credentials, or private keys. A personal subscription is not evidence of institutional approval.


**Codex • supported macOS/Linux environment:**

```bash
curl -fsSL https://chatgpt.com/codex/install.sh | sh
codex --version
mkdir -p ~/code/scg-sandbox
cd ~/code/scg-sandbox
codex
```

Use the sign-in flow for your approved account. For an approved remote/headless setup, `codex login --device-auth` provides a browser code flow when enabled by your account/workspace. Keep codes private. Sources: [Codex installation](https://learn.chatgpt.com/docs/codex/cli) and [authentication](https://learn.chatgpt.com/docs/auth).

**Claude Code • supported macOS/Linux environment:**

```bash
curl -fsSL https://claude.ai/install.sh | bash
export PATH="$HOME/.local/bin:$PATH"
claude --version
mkdir -p ~/code/scg-sandbox
cd ~/code/scg-sandbox
claude
```

Follow the displayed login flow; open any authentication link on your laptop if remote. If needed, add the PATH line once to your shell startup file. Native Claude installations auto-update. Check [Anthropic's setup requirements](https://code.claude.com/docs/en/setup) before installing; older Linux environments may be incompatible. A GLIBC error calls for an admin-supported environment, not replacing cluster libraries.

**Keep cluster work scheduled.** tmux and AI tools do not allocate compute resources. Use Slurm for analysis, intensive builds, and commands launched by an agent. A modest interactive allocation example is:

```bash
srun --partition=interactive --account=default \
  --nodes=1 --ntasks=1 --cpus-per-task=2 --mem=8G \
  --time=01:00:00 --pty bash
```

Wait for the allocation before running work; type `exit` to release it. Availability, limits, software and outbound network access depend on SCG configuration. Longer workloads belong in appropriate batch jobs. Sources: [SCG quick start](https://login.scg.stanford.edu/quick_start/) and [account selection](https://login.scg.stanford.edu/faqs/account/).

## 8. Keep a terminal with tmux

tmux keeps your server terminal running after SSH disconnects. Reconnect to the **same login node** to find your session. It does not allocate compute resources or extend a Slurm time limit.

**Laptop:** `ssh scg`. **Then on SCG:**

```bash
tmux new -s lab              # Start a session
```

Detach: **Ctrl+B**, release, then **D**. Later:

```bash
tmux ls                     # List sessions on this node
tmux attach -t lab          # Return and see the terminal output
```

**Check your job from inside tmux.** A tmux session can exist after a job finishes; check the job itself. If your current pane is busy, press **Ctrl+B**, then **C** to open another window for these checks.

```bash
squeue -u "$USER"            # Slurm jobs: R = running, PD = pending
```

For the batch example in section 9, replace the ID and project path below:

```bash
job_id=12345678              # Use the ID returned by sbatch
sacct -j "$job_id" --format=JobID,State,ExitCode,Elapsed
cd /oak/stanford/groups/longaker/rosenwu/my_project
ls -lh logs/                # See available logs and their sizes
tail -n 30 "logs/analysis-${job_id}.out"  # Recent output
tail -n 30 "logs/analysis-${job_id}.err"  # Recent errors
tail -f "logs/analysis-${job_id}.out"    # Watch new output live
```

**Ctrl+C** stops `tail -f`, not the batch job. Logs may not exist while a job is pending; a quiet log alone does not mean it stopped. Use `sacct` after a job leaves the queue: `COMPLETED` means it finished; review the exit code and outputs.

**For a command started directly in a tmux pane:** reattach to view its output. From another window on the same node, list your processes:

```bash
ps -u "$USER" -o pid,etime,stat,args
```

This lists processes on the current node only. For work running through Slurm, use the Slurm checks above. Terminal output is not automatically saved to a file; batch jobs use the logs configured in their submission script.

**Useful keys:** Ctrl+B then `n` / `p` switches windows; `[` opens scroll mode (Ctrl+C exits); `?` lists shortcuts. Use `exit` to close a finished shell.

More shortcuts: [Chuner Guo](https://github.com/chunerguo/resources/blob/main/cheatsheets/tmux_cheatsheet.md) · [Michael Lihs](https://gist.github.com/michaellihs/b6d46fa460fa5e429ea7ee5ff8794b96) · [Andy Acer](https://github.com/andyacer/tmux-cheatsheet). Some reference shortcuts require custom configuration.

## 9. Submit a batch job with Slurm

A batch job is a saved script that Slurm queues and runs with reserved resources. After `sbatch` confirms submission, you can disconnect: **tmux is not required for a submitted job to continue**. Submitting with `sbatch` and choosing the partition named `batch` are separate decisions. SCG also supports `sbatch` on the `interactive` partition with `--account=default`; the example below explicitly uses the billed `batch` partition.

**SCG • find your authorized account first:**

```bash
scgwhoami
```

Choose the PI/project account approved for your work from “Available SLURM Accounts.” Your Oak directory name does not establish the billing account. `YOUR_LAB_ACCOUNT` below is a required placeholder; replace it before submitting. Confirm lab billing authorization and size resources for the actual task.

**Use your existing project and code.** No separate batch working folder is required. Put a small submission file beside the code you already intend to run:

```text
/oak/stanford/groups/longaker/rosenwu/my_project/
├── analysis.py        # Your existing analysis code
├── run_analysis.sbatch # Slurm launcher you create once and reuse
└── logs/              # Slurm output and errors
```

`my_project` and `analysis.py` are examples: replace them with your actual folder and script. Your code must be accessible on SCG/Oak, not only on your laptop. The submission file requests resources and invokes that code; you do not need to paste your analysis into it.

**SCG • open your existing project, create the log folder, and edit the launcher:**

```bash
cd /oak/stanford/groups/longaker/rosenwu/my_project
mkdir -p logs
nano run_analysis.sbatch
```

Paste this into `run_analysis.sbatch`. Replace the account, project path, environment setup, and script name; choose suitable resource requests:

```bash
#!/bin/bash
#SBATCH --job-name=analysis
#SBATCH --partition=batch
#SBATCH --account=YOUR_LAB_ACCOUNT
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=1
#SBATCH --mem=8G
#SBATCH --time=01:00:00
#SBATCH --output=logs/%x-%j.out
#SBATCH --error=logs/%x-%j.err

set -euo pipefail
cd /oak/stanford/groups/longaker/rosenwu/my_project
echo "Job: $SLURM_JOB_ID"
echo "Host: $(hostname)"
echo "Started: $(date)"

# REQUIRED: load your analysis environment here, as you normally do.
# For example, replace the placeholder with a module from `module avail`:
module load YOUR_PYTHON_MODULE

# This is where your existing analysis script is called:
python -u analysis.py
echo "Finished: $(date)"
```

Save with Ctrl+O, Enter, then Ctrl+X. `--mem=8G` requests 8 GB total for the node; `--time` is the wall-clock limit. `%x` becomes the job name and `%j` the job ID. Keep all `#SBATCH` lines before executable commands; shell variables such as `$HOME` are not expanded in those directives. Create `logs/` **before submission**, because Slurm opens logs before your script runs.

The `cd` line sets the working directory used by your code, including relative input/output paths. Replace `module load YOUR_PYTHON_MODULE` with your actual module or environment activation commands. For an R script, replace the Python line with `Rscript analysis.R`; for a shell pipeline, use `bash pipeline.sh`.

**Submit from that same project folder:**

```bash
bash -n run_analysis.sbatch       # Check shell syntax; does not submit
sbatch run_analysis.sbatch        # Submit once; save the returned job ID
squeue -u "$USER"          # See queued/running jobs
```

Use `sbatch`, not `bash run_analysis.sbatch`: running it with bash bypasses resource allocation. A response such as “Submitted batch job 12345678” means accepted into the queue, not completed. `PD` means pending; `R` means running. The queue reason can indicate priority or resource availability.

**Inspect logs and completion.** Replace `12345678` with the ID actually returned:

```bash
job_id=12345678
scontrol show job "$job_id"
tail -n 50 "logs/analysis-${job_id}.out"
tail -n 50 "logs/analysis-${job_id}.err"
sacct -j "$job_id" --format=JobID,JobName,State,ExitCode,Elapsed,MaxRSS
```

Logs may not exist while pending. After the job leaves `squeue`, use `sacct`; accounting may take a moment to update. `COMPLETED` with `0:0` indicates the script exited successfully, but you should still check scientific outputs. `FAILED`, `TIMEOUT`, or `OUT_OF_MEMORY` call for reviewing logs and resource requests. MaxRSS may appear on the `.batch` or another job-step row.

**Cancel only when intended:** `scancel "$job_id"` stops that queued/running job. Ctrl+C on your laptop does not cancel a job already submitted with `sbatch`.

**Reuse the launcher:** edit the script name, arguments or resource requests for another run. If you already have a shell script with the required `#SBATCH` header, submit it directly; another wrapper is unnecessary. Requesting more CPUs does not automatically parallelize a program; configure its threads to match. Optional email notifications require both `#SBATCH --mail-user=YOUR_STANFORD_EMAIL` and `#SBATCH --mail-type=END,FAIL,TIME_LIMIT` above `set -euo pipefail`; replace the email placeholder. This template was syntax-checked locally, not submitted to SCG.

Sources: [SCG Slurm tutorial](https://login.scg.stanford.edu/tutorials/job_scripts/), [SCG account selection](https://login.scg.stanford.edu/faqs/account/), [SCG partition examples](https://login.scg.stanford.edu/scg_primer/), and [Slurm sbatch reference](https://slurm.schedmd.com/sbatch.html).

## 10. More command lines incoming

## 11. VS Code setup

Coming soon.
