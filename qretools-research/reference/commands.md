# The qretools commands

This file lists the commands that research needs. The authority is the command's own
help (`npx -y qretools@<version> --help`) and the
[README of the command](https://github.com/JHUCities/qretools/blob/main/packages/cli/README.md).

## Install and run

The command needs Node 22 or later. `build` and the reading of another repository also
need git on your `PATH`.

```sh
npx -y qretools@<version> check .
```

Give an exact version in each command. The `-y` flag stops `npx` from asking before it
installs the package.

## Exit codes

| Code | Meaning |
|---|---|
| 0 | The command did its work. |
| 1 | The command found a problem, or refused the input. |
| 2 | The command line is not correct. `responses` also gives 2 when an option for a fact is missing, or when an option gives a fact that the source gives (`responses-facts`). |

## The workspace

A workspace is one folder: `workspace.yaml`, `questions/<folder>/`, the shared folders
(for example `scales/`), `missing.yaml`, `select_all.yaml`, `instruments/` and, for
data, `maps/`.

| Command | What it does |
|---|---|
| `check [dir] [--strict]` | Writes one finding on each line, as `path:line:col: level: message [code]`. It exits with 1 if a field is empty or wrong. With `--strict`, a warning also gives 1. |
| `build [dir] [-o outdir]` | Writes the DDI of the last commit to `<outdir>/ddi.json` (default `dir/dist`). It writes each instrument to `<outdir>/instruments/<name>.json`, and copies each map in `maps/` beside it. It ignores changes that are not committed. It refuses a shallow clone, because the versions come from the git history. |
| `instrument check <file>` | Writes the findings of one instrument. |
| `instrument export <file> [-o file]` | Writes the DDI of one instrument. It records the commit of each tag that the instrument reads, under `locked:` in `workspace.yaml`. |

Use `build` for research. A dataset refers to variables by URN, and the URN holds the
version that `build` reads from the git history.

## Datasets

| Command | What it does |
|---|---|
| `responses <data.csv> --map <map> [--ddi <DDI>] [--asked-by <term>] [-o <dataset.json>]` | Makes a dataset from a data file (CSV with one header row) and its column map. |
| `responses <export.csv> --map <name>-ddi-qsf.columns.json [--ddi <DDI>] [--asked-by <term>] [--answered-by <term>] [-o <dataset.json>]` | Makes a dataset from a Qualtrics export and its Qualtrics map. |
| `responses <answers.csv> --ddi <DDI> --synthetic --runtime <name> [--runtime-version <v>] [--file <path>]... [-o <dataset.json>]` | Makes a dataset from the answers of a model. The harness runs this command. |
| `responses <answers.csv> --ddi <DDI> --answered-by <term> --asked-by <term> [--file <path>]... [-o <dataset.json>]` | Makes a dataset from real answers that are already in the form of an answers file. |
| `dataset export <dataset.json> --csv --ddi <DDI> [--access <level>] [-o <file.csv>]` | Writes the values as long CSV. |
| `dataset export <dataset.json> --provenance --ddi <DDI> [--access <level>] [-o <file.json>]` | Writes the dataset without its values. |
| `qsf-map <DDI or tag reference> -o <dir>` | Writes the column map of a Qualtrics survey. See [workspace.md](workspace.md). |
| `qsf-map --export <export.csv> --ddi <DDI> -o <dir>` | Writes the same map from an export of a fielded survey. |

Rules:

- Without `-o`, `responses` and `dataset export` write to stdout.
- `responses` refuses to replace a file that holds a different dataset. If the file
  holds the same dataset, it writes nothing and says `unchanged`.
- The name of the dataset is the file name of `-o` without `.json`. Use only letters,
  digits, `-` and `_`.
- `responses` refuses `--file` with `--map`.
- `dataset export` compares the SHA-256 digest of the `--ddi` file with the digest that
  the dataset records. If they are different, it writes nothing (`dataset-ddi`).

`<DDI>` is a DDI file or, from 2.0.0-alpha.13, a DDI URL. See [datasets.md](datasets.md)
for the DDI URL, the provenance and access levels.

## One step of an interview

`next <DDI> [state.json]` reads the DDI of an instrument and a state, and writes
the next step as JSON. With no state file, or with `-`, it reads the state from stdin.
`next --schema` writes the JSON Schemas of the state and of the reply.

- The state holds `inputs` by their names in the instrument, and `answers`. Each answer
  has the URN of its variable and a `value`.
- The reply is `ask` (a question), `input` (a value that the route needs), or `end`.
- QREtools decides the routing. A program that calls `next` does not evaluate a
  condition.

The harness uses `next`. You do not need it to make a dataset from a data file.
