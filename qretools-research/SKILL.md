---
name: qretools-research
description: Use when you do survey research with QREtools - you check or build a workspace of survey questions and instruments, put a survey in Qualtrics or make its column map, make a DDI dataset from a data file, a Qualtrics export or the simulated answers of a language model (qretools-harness), export a view of a dataset, or compare real answers with synthetic answers.
---

# Survey research with QREtools

QREtools keeps survey questions as YAML in a git repository and writes them as
DDI-Lifecycle 4.0. Its command, `qretools`, is on npm.

## The terms

| Term | Meaning |
|---|---|
| workspace | A git repository that holds questions and instruments. |
| instrument | A survey: its questions and their order. |
| release | What `qretools build` writes for one commit. A tag, for example `v15`, names it. |
| DDI URL | The `https://` URL of the DDI of an instrument in a published release. |
| map | `maps/<instrument>-ddi-<source>.columns.json`. It gives the DDI variable of each column of a source file. |
| source file | A file of answers from outside QREtools, for example a Qualtrics export or a published data file. |
| answers file | A CSV file of answers in the form of QREtools, with the times of each interview. |
| dataset | A DDI document that holds the answers and their provenance. |
| view | What `qretools dataset export` writes: the values as CSV, or the provenance only. |
| harness | `qretools-harness`. It asks a language model the questions, in a simulation. |
| run | One `qretools-harness respond` command. It makes one dataset from the respondents that finish. |
| respondent | One interview. In a run, each respondent has its own seed and its own conversation. |

## The sequence

1. `check` and `build` the workspace. See [reference/workspace.md](reference/workspace.md).
2. Make a dataset with `responses`, from a source file and its map, or from an answers
   file. See [reference/datasets.md](reference/datasets.md).
3. For synthetic answers, use the harness. See [reference/harness.md](reference/harness.md).
4. Write views with `dataset export`. See [reference/datasets.md](reference/datasets.md).
5. Check the datasets, then compare them. See [examples/gss-2010.md](examples/gss-2010.md).

Run each command with an exact version: `npx -y qretools@<version> <command>`. A command
that reads a DDI URL needs 2.0.0-alpha.13 or later. For other commands, use the version
that the workspace pins in its `mise.toml`. If this skill and the output of `--help` do
not agree, the output of `--help` is correct. The options of each command are in
[reference/commands.md](reference/commands.md).

## Rules for research

Obey these rules. They protect respondents and keep the results honest.

1. **Do not commit or publish real answers.** This applies to the source file, a CSV
   that you make from it, a dataset, and a CSV view. Write them outside the repository.
   Do not write them into `dist/` or into a folder that CI publishes.
2. **Publish only a provenance view.** Make sure that it holds no `ItemSet`.
3. **Treat a dataset file as data at the most closed access level.** Do not filter it
   yourself. If the DDI declares access levels, give `--access <level>` to each
   `dataset export`. If it declares none, the command refuses `--access`.
4. **Keep every run parameter.** Do not delete `traces.jsonl` or the copy of the config.
   Check them before you share them: they can hold local paths.
5. **Say that synthetic answers are synthetic.** Make a dataset of model answers only
   with `--synthetic`. Never give `--answered-by` for model answers.
6. **Do not choose a weight.** The researcher chooses it. Show the researcher the advice
   of the publisher, if the map records it, and ask.
7. **Compare like with like.** Compare two datasets only if they record the same DDI
   digest. Compare them by variable URN, not by respondent.
8. **Do not state figures from a licensed data file** in a commit, a pull request or a
   file that you publish. Do not print its values in a shared log.
9. **Do not guess a fact.** If `responses` asks for an option, ask the researcher for
   the value.

If a rule stops a step, stop and ask the researcher.

## Not in QREtools yet

- `qretools compare`. Until it exists, do the checks and the comparison in
  [examples/gss-2010.md](examples/gss-2010.md) yourself.
- Waves and sweeps.
