---
name: qretools-research
description: Use this skill for survey research with QREtools. Use it when you check or build a workspace of survey questions and instruments. Use it when you put a survey in Qualtrics or make its Qualtrics map. Use it when you make a DDI dataset from a data file, a Qualtrics export or the synthetic answers of a language model (qretools-harness). Use it when you export a view of a dataset, or compare real answers with synthetic answers.
---

# Survey research with QREtools

QREtools keeps survey questions as YAML in a git repository and writes them as
DDI-Lifecycle 4.0. Its command, `qretools`, is on npm.

## The terms

Each term has one meaning in all the files of this skill.

| Term | What it means |
|---|---|
| workspace | A git repository that holds questions and instruments. |
| instrument | The questions of a survey and their order. |
| field (a verb) | To give a survey to its respondents. |
| release | What `qretools build` writes for one commit. A tag, for example `v15`, names it. |
| DDI URL | The `https://` URL of the DDI of an instrument in a published release. |
| source file | A file of answers from outside QREtools: a data file or a Qualtrics export. |
| data file | A source file that is not a Qualtrics export, for example the GSS data. |
| Qualtrics export | The CSV file of answers that Qualtrics exports for a survey. |
| map | `maps/<instrument>-ddi-<source>.columns.json`. It gives the DDI variable of each column of a source file. |
| data map | The map of a data file. |
| Qualtrics map | The map of a Qualtrics export, `maps/<instrument>-ddi-qsf.columns.json`. |
| answers file | A CSV file of answers in the form of QREtools, with the times of each interview. |
| real answers | The answers of persons. |
| synthetic answers | The answers of a language model, from a run. |
| dataset | A DDI document that holds the answers and their provenance. |
| view | What `qretools dataset export` writes: a CSV view or a provenance view. |
| CSV view | The values of a dataset, as CSV. |
| provenance view | The provenance of a dataset, with no values. |
| harness | `qretools-harness`. It asks a language model the questions, in a simulation. |
| run | One `qretools-harness respond` command. It makes one dataset from the respondents that finish. |
| respondent | One interview. In a run, each respondent has its own seed and its own conversation. |

## The sequence

1. Run `check` on the workspace. See [reference/workspace.md](reference/workspace.md).
2. Run `build` on the workspace. See [reference/workspace.md](reference/workspace.md).
3. Make a dataset with `responses`, from a source file and its map, or from an answers
   file. See [reference/datasets.md](reference/datasets.md).
4. For synthetic answers, use the harness. See [reference/harness.md](reference/harness.md).
5. Write views with `dataset export`. See [reference/datasets.md](reference/datasets.md).
6. Check the datasets. See [examples/gss-2010.md](examples/gss-2010.md).
7. Compare the datasets. See [examples/gss-2010.md](examples/gss-2010.md).

Run each command with an exact version: `npx -y qretools@<version> <command>`. A command
that reads a DDI URL needs 2.0.0-alpha.13 or later. For other commands, use the version
that the workspace pins in its `mise.toml`. If this skill and the output of `--help` do
not agree, the output of `--help` is correct. The options of each command are in
[reference/commands.md](reference/commands.md).

## Rules for research

Obey these rules. They protect respondents. They also make sure that the results do not
mislead a reader.

1. **Do not commit or publish real answers.** This rule applies to the source file, a CSV
   file that you make from it, a dataset and a CSV view. Write them outside the
   repository. Do not write them into `dist/` or into a folder that CI publishes.
2. **Publish only a provenance view.** Make sure that it holds no `ItemSet`.
3. **Treat a dataset as data at the most closed access level.** Do not filter it
   yourself. If the DDI declares access levels, give `--access <level>` to each
   `dataset export`. If the DDI declares no access levels, the command refuses `--access`.
4. **Keep every parameter of a run.** Do not delete `traces.jsonl` or the copy of the
   config. Check them before you share them. They can hold local paths.
5. **Say that synthetic answers are synthetic.** Make a dataset of synthetic answers only
   with `--synthetic`. Never give `--answered-by` for synthetic answers.
6. **Do not choose a weight.** The researcher chooses it. If the map records the advice of
   the publisher, show the advice to the researcher. Then ask the researcher which weight
   to use.
7. **Compare only datasets of the same DDI.** Compare two datasets only if they record the
   same DDI digest. Compare them by variable URN, not by respondent.
8. **Do not state figures from a licensed data file.** Do not put them in a commit, a pull
   request or a file that you publish. Do not print the values of the data file in a
   shared log.
9. **Do not guess a fact.** If `responses` asks for an option, ask the researcher for the
   value.

If a rule stops a step, stop. Ask the researcher what to do.

## Not in QREtools yet

- QREtools has no `qretools compare` command. Until it has one, do the checks and the
  comparison in [examples/gss-2010.md](examples/gss-2010.md) yourself.
- QREtools has no waves and no sweeps.
