# AGENTS.md

Working guidance for AI coding agents and developers in the `QuasarApp/constexpr-xxh3` repository.

## Description

constexpr-xxh3 is a C++20 constexpr implementation of the XXH3 64-bit variant
of [xxHash](https://github.com/Cyan4973/xxHash) — a single-header,
header-only library, so a hash can be computed at compile time. Three
functions are implemented: `XXH3_64bits_const`,
`XXH3_64bits_withSecret_const`, `XXH3_64bits_withSeed_const`. Used as a
submodule by `QuasarApp/Heart` (and transitively by Games/Ecliptica).

- **Language:** C++20, header-only (`constexpr-xxh3.h`)
- **Build system:** CMake ≥ 3.19 (declares the INTERFACE target `xxh3`)
- **License:** see `LICENSE`

### Repository layout

| Path | Contents |
|---|---|
| `constexpr-xxh3.h` | the entire library (single header) |
| `test/` | standalone unit test (`constexpr-xxh3-test.cc`, gtest, built via `test/Makefile`) |
| `CMakeLists.txt` | declares the INTERFACE target `xxh3` for consumers |

### Architecture

- Consumers use it via CMake (`add_subdirectory(<xxh3>)` + link `xxh3`) or by
  including the header directly with `-std=c++20`.
- Correctness contract: results must be bit-identical to the reference
  xxHash library — the unit test asserts parity.

## Developing process

### Toolchain

| Tool | Version / notes |
|---|---|
| CMake | ≥ 3.19 |
| Compiler | C++20 required |
| GoogleTest | for the standalone unit test (`test/Makefile`) |

### Build

Step by step.

**1. One-time toolchain setup.** CMake ≥ 3.19 + a C++20 compiler + GoogleTest
for the unit test. No Qt.

**2. Get the sources.** No submodules.

```sh
git clone https://github.com/QuasarApp/constexpr-xxh3.git
```

**3. Configure** (consumers normally configure their own project; the
standalone configure below verifies the CMake wiring).

```sh
cmake -B build -G Ninja -DCMAKE_BUILD_TYPE=Release
```

**4. Build.** There is nothing to compile (header-only INTERFACE library) —
the build step verifies that the CMake wiring is intact.

```sh
cmake --build build --parallel 16
```

**5. Find the result.** No binary — the deliverable is the header plus the
`xxh3` CMake target.

CMake options: none.

### Branching, labels and pull requests (workflow for AI agents)

- Work is issue-driven: one GitHub issue → one branch named
  `task_<issue-number>`, branched **strictly from `main`** — never from
  another task branch.
- Commit messages: short imperative subject + issue reference, e.g.
  `fix: <summary> (issue #<issue>)`, `docs: <summary> (issue #<issue>)`.
- Keep the build and lint green; a red CI blocks the merge.

**The exact commands.**

```sh
# 1. every task starts from a freshly fetched main
git fetch origin
git checkout -b task_<issue> origin/main

# 2. label the issue while working. Workflow labels in this repo:
#    in-progress, high/medium/low priority, question, sponsor
gh issue edit <issue> --add-label in-progress

# 3. post the initial checklist as a comment; afterwards update that same
#    comment in place, never re-post it
gh issue comment <issue> --body-file /tmp/checklist.md
gh issue comment <issue> --edit-last --body-file /tmp/checklist.md

# 4. push and open the PR — see "Creating the pull request" below
git push -u origin task_<issue>
gh pr create --base main --head task_<issue> \
  --title "<type>: <short imperative summary> (issue #<issue>)" \
  --body-file /tmp/pr_body.md

# 5. the issue goes to waiting-for-review; watch CI from the CLI
gh issue edit <issue> --remove-label in-progress --add-label waiting-for-review
gh pr checks <number>
```

This repo does not define a `waiting-for-review` label yet — create it
before first use: `gh label create waiting-for-review --color 'fbca04'`.

**Status messages — what to post and when.** An agent must keep a
human-readable trail on the issue (and on the PR once it exists). Every
message is a GitHub comment with a `##` heading; the checklist comment is
updated in place (`--edit-last`), never duplicated.

| Stage | Label(s) | Comment template |
|---|---|---|
| Start | `in-progress` | `## Task issue_XXX: <title> — initial checklist (branch task_XXX from main)` — one `- [ ]` line per sub-task |
| Each step done | — | edit the checklist comment: mark the line `- [x]` + one short evidence note (command run, result) |
| Blocked / needs input | `blocked` / `question` | `## Blocked — task_XXX` — what is blocking and exactly what is needed to continue |
| PR opened | `waiting-for-review` | `## Progress — task_XXX` — all lines `- [x]` + `**PR: <url>**` + one-line verification summary |
| Review changes requested | `waiting-for-review` | `## Review-fix cycle — PR #NNN — update` — one `- [x]` line per review point, each with the evidence |
| CI red on the PR | `in-progress` | `## CI failure — PR #NNN` — failing check, cause, fix plan; push the fix, then update the comment |

**Creating the pull request.** An agent has no browser and cannot create a
pull request from the GitHub web UI — it must use the `gh` CLI (a token with
the `repo` scope; in this organisation the agent runs as `QuasarAppCI`):

```sh
gh pr create \
  --base main \
  --head task_<issue> \
  --title "<type>: <short imperative summary> (issue #<issue>)" \
  --body-file /tmp/pr_body.md
```

- The body carries what changed and the verification actually performed
  (commands + their real output); `(issue #<n>)` in the title links the PR
  to the issue. Use `--draft` while still iterating.
- A pull request exists only when `gh pr create` prints its URL. Confirm
  with `gh pr view <number> --json number,url,mergeable` before reporting
  "PR created" — never post that status on faith.
- If work is blocked, keep the PR open, set the issue label to `blocked` and
  post the status message above.

### Documentation rules

- The README documents the three functions and the xxHash parity contract —
  keep it in sync with `constexpr-xxh3.h`.
- A changed hash function ships **with** an updated unit test — never in a
  follow-up ticket.

## Testing process

### Test infrastructure

A standalone gtest unit test in `test/` (`constexpr-xxh3-test.cc` +
`test/Makefile`) that checks the constexpr functions produce exactly the same
results as the reference xxHash library.

### Running the tests from CMake

The unit test is not wired into CMake; it runs through its own Makefile:

```sh
make -C test test-gcc     # or: make -C test test-clang
```

### Smoke tests — the most important testing step

The smoke test is: `make -C test test-gcc` completes with zero failures —
headless, seconds-fast, no Qt. Because the hash values are part of
compile-time contracts in downstream QuasarApp projects, any change to
`constexpr-xxh3.h` must re-run this test in the same PR.