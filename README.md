# Roejobs

Roejobs is a local job scheduling and management tool designed for running, monitoring, and controlling long-running tasks on a single machine. It provides:

- A background job server managing multiple tasks simultaneously.
- A CLI for submitting jobs from files or the command line.
- A web dashboard to monitor job status, view logs, cancel or restart jobs.
- Automatic management of working directories, output logs, and job queuing.

## Features

- Queue jobs from files or directly from the CLI.
- Optionally wait for all queued jobs to finish and report their outcome (`--wait`).
- Support for CWD=... prefixes per job or global override.
- View running, queued, succeeded, failed, or cancelled jobs in a web browser.
- Stream job stdout/stderr to log files and view them via the dashboard.
- Cancel or restart jobs directly from the web UI.
- Shutdown the server gracefully from the web UI.
- CLI automatically launches the server in the background if it is not running.

## Installation

```bash
pip install roejobs
```

This will install the CLI commands and the package itself.
Or, if you cloned this package locally:

```bash
pip install -e .
```

### Starting the server

Start the job server on the default port (5678):
```bash
roejobs-server
```

Optional arguments:

```bash
roejobs-server --port 1234           # Specify a custom port
roejobs-server --n-processes 20      # Set maximum concurrent jobs
```

- The server runs locally (127.0.0.1) and serves the web dashboard.
- Default maximum concurrent jobs: 10.

### Access the dashboard

Open your browser at: (http://127.0.0.1:5678/)

From here you can:
- Monitor all jobs
- View logs
- Cancel or restart jobs
- Shut down the server

## Submitting jobs via CLI
### 1. From a task file

Each line of a task file is a job:

```bash
# Comment lines start with #
CWD=./exp python train.py --lr 0.001 --batch-size 64
python eval.py --checkpoint ckpt.pt
```

Submit jobs:
```bash
roejobs-cli tasks.txt
```

Optional flags:
- `--override-cwd — ignore CWD=...` in job specs and use the CLI's current working directory.
- `--server` — point to a server on a custom port.
- `--wait` — after queuing, block until all submitted jobs conclude, then print their final states and exit.

### 2. Inline submission

You can also submit jobs directly from the CLI:
```bash
roejobs-cli --jobs "python quick_test.py" "CWD=./exp ./run.sh --config cfg.yaml"
```
- Syntax matches task files.
- Works with or without --override-cwd.

### 3. Lazy server start

If the server is not running, the CLI automatically launches it in the background on the correct port.

### 4. Waiting for jobs to finish

By default the CLI returns as soon as the jobs are queued. Pass `--wait` to block until every job it submitted reaches a terminal state (Succeeded, Failed, or Cancelled):

```bash
roejobs-cli --wait --jobs "python train.py --lr 0.01" "python eval.py"
```

When all jobs conclude, the CLI prints each job's final state and exits. The exit code is `0` only if **every** submitted job succeeded; if any job failed or was cancelled it exits `1`, which makes `--wait` convenient in scripts and pipelines.

There is no timeout — job durations are hard to predict — but you can stop waiting at any time with `Ctrl+C`. Interrupting only stops the CLI from waiting; the jobs keep running on the server and can still be monitored from the dashboard.

## Job spec syntax

- Lines starting with `#` are ignored.
- Optional working directory per job:
```bash
CWD=some/path command args...
```

- If no `CWD` is specified, the CLI/server uses the current directory.
- Commands can be very long; they are truncated in the dashboard for readability, with full output available in the log view.

## Web dashboard features

- Job table showing status: Queued, Running, Succeeded, Failed, Cancelled.
- Full command and logs available per job.
- Cancel or restart individual jobs.
- Shut down server with a single button (gracefully kills running jobs).
- Commands truncated visually for long parameters, with full command in a tooltip.

## Example workflow
```bash
# Start server (background auto-start optional)
roejobs-server --port 5678

# Submit jobs from a file
roejobs-cli tasks.txt

# Submit inline jobs
roejobs-cli --jobs "python train.py --lr 0.01" "CWD=./exp ./run.sh"

# Open browser to monitor jobs
firefox http://127.0.0.1:5678/
```

## Notes

- Roejobs is local-only and designed for single-machine usage.
- The web interface is unprotected, so only bind to 127.0.0.1.
- Logs for each job are stored in temporary directories and accessible from the web UI.
