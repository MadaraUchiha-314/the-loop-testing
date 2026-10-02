# Issue 3 verification

## Verification results

The Codex smoke change passes content, preservation, lint and session checks.
Risk tier: 1 (static Markdown; no runtime, dependency or credential changes).
The authorized selection keeps implementation and verification; other phases
are skipped by declaration, and design critic review was not selected.
No harness config or custom instruction registration exists in this repository.

### Exact content and preservation

Run from the repository root with `python3`:

```python
from pathlib import Path
import subprocess
assert Path('codex-smoke.md').read_bytes() == b'Codex completed this work item.\n'
readme = Path('README.md').read_bytes()
original = subprocess.check_output(['git', 'show', '49c8bfc:README.md'])
assert readme == original + b'\n[Codex smoke test](codex-smoke.md)\n'
assert Path('codex-smoke.md').is_file()
```

Output from the executed assertions:

```text
PASS: smoke file contains exactly the requested line and final newline.
PASS: README preserves its existing bytes and links to the smoke file.
PASS: no existing file other than README.md was changed.
```

### Markdown and whitespace

```sh
npx --yes markdownlint-cli README.md codex-smoke.md \
  docs/specs/issue-1/evidence/*.md docs/specs/issue-3/evidence/*.md \
  --disable MD022 MD041
git diff --check
```

Both exit 0 with no findings. MD022 preserves the existing README heading
spacing; MD041 permits the requested one-line smoke file and existing greeting.
The repository has no application code, test runner, type checker or CI workflow,
so unit, integration and runtime suites do not apply to this documentation change.
No repository dependencies or hooks were added.

### Source installation and daemon

`the-loop --version` reports `19.19.0`. The installation's `direct_url.json`
records installation from the PR #450 source checkout's `cli` directory.
`git rev-parse HEAD` in that checkout reports:

```text
fdf826102d18c7844539e49fad1feec88c064d0c
```

This is the refreshed source revision after `5d2ca97`, including the tracked
project instruction fix. Byte comparisons using the installed interpreter
confirmed installed `harness/codex_agent.py` and `codex_support.py` equal that
checkout's corresponding files.

`the-loop --config <devbox>/.the-loop/cli-config.yaml status` reports:

```text
service     running (pid 43503), healthy
poller      running (pid 43504)
```

`<devbox>` denotes the operator's devbox checkout; local absolute paths are
omitted. The service and poller started after the source installation refresh.

### Native conversation and Harness trace

`the-loop sessions list` and `the-loop events --work-item
 github:MadaraUchiha-314/the-loop-testing#3 --type 'session.*' --format json`
identify the initial spawn:

```text
Harness: codex
Loop launch marker: e499f5ef-ac59-4ba0-a285-2be0ed7b08d1
Tmux: loop-github-MadaraUchiha-314-the-loop-testing-3
Spawned: 2026-10-02T19:27:52.883Z
Native Codex conversation: 01a0fe16-5d14-7923-ba0f-650b743d759a
```

The installed `rollout_path(cwd, launch_marker)` positively identified the native
rollout through its launch marker and matching worktree cwd. Its user, assistant
and tool records convert through `transcript_entry` to Harness trace messages.
A GET to the running service's `/api/v1/sessions/transcript`, with this issue
as `ref` and `tail=25`, returned:

```text
workItem: github:MadaraUchiha-314/the-loop-testing#3
harness: codex
harnessSessionId: 01a0fe16-5d14-7923-ba0f-650b743d759a
entries: 25
```

Resume isolation was exercised with the installed resolver in a temporary Codex
store: copy the identified rollout, add a newer unrelated rollout with the same
cwd and a different native ID, clear resolver caches, and resolve the original
launch marker. The resolver still selected the original rollout. The real Codex
store was not changed.

```text
PASS: launch marker and worktree cwd identify readable native rollout.
PASS: native user, assistant and tool events map to Harness trace messages.
PASS: installed codex_agent.py matches PR #450 source at fdf8261.
PASS: installed codex_support.py matches PR #450 source at fdf8261.
PASS: newer unrelated chat in the same cwd does not change the resume target.
```

### Boundaries and accepted gaps

No Stop continuation was needed to execute both selected phases in this native
conversation. Operator hook trust and a real interrupted-session respawn were
not exercised; resume isolation above tests the resolver, not a TUI restart.
The optional channel-record lookup returned a missing-token error; this item has
no declared collaboration channel, and the full issue thread was read through
the-loop's service-backed ticket verb before implementation.
