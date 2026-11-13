---
name: qretools-research
description: Use when you do survey research with QREtools - you check or build a workspace of survey questions and instruments, make a DDI dataset from a published data file or from the simulated answers of a language model (qretools-harness), export a view of a dataset, or compare real answers with synthetic answers.
---

# Survey research with QREtools

QREtools keeps survey questions as YAML in a git repository (a workspace) and writes
them as DDI-Lifecycle 4.0. Its command, `qretools`, is on npm. The work goes in this
sequence:

1. `check` the workspace.
2. `build` the DDI of the workspace and of each instrument.
3. `responses` makes a dataset from answers and the DDI of one instrument.
4. `dataset export` writes a view of the dataset: CSV values, or the provenance only.

Run each command with an exact version: `npx -y qretools@<version> <command>`. Use the
version that the workspace pins in its `mise.toml`. If it pins none, use the newest
version on npm (`npm view qretools version`). If this skill and the output of
`--help` do not agree, the output of `--help` is correct.

## Rules for research

Obey these rules. They protect respondents and keep the results honest.

1. **Do not commit or publish real answers.** This applies to the data file, a CSV
   that you make from it, a dataset, and a CSV view of a dataset. Write them outside
   the repository. Do not write them into `dist/` or into a folder that CI publishes.
2. **Publish only a provenance view.** It holds no value and no respondent.
3. **Treat a dataset file as data at the most closed access level.** Do not filter it
   yourself. If the DDI declares access levels, give `--access <level>` to each
   `dataset export`. If the DDI declares none, the command refuses `--access`.
4. **Keep every run parameter.** The harness writes them in `traces.jsonl` and keeps a
   copy of the config. Do not delete these files. Check them before you share them:
   they can hold local paths.
5. **Say that synthetic answers are synthetic.** Make a dataset of model answers only
   with `--synthetic`. Never give `--answered-by` for model answers.
6. **Do not choose a weight.** The researcher chooses it. The tools never choose one.
   A data-file map can record the publisher's advice in its `weighting` field. Show the
   researcher that advice, and ask.
7. **Compare like with like.** Compare two datasets only if they record the same DDI
   digest. Compare them by variable URN. Do not compare them by respondent.
8. **Do not state figures from a licensed data file** in a commit, a pull request or
   another file that you publish. Do not print values from it in a shared log.

If a rule stops a step, stop and ask the researcher.

## Facts come from data, then from an option

`responses` gets each fact of a collection (who answered, how the questions were asked)
from the source first. If the source does not give a fact, an option must give it. If
no option gives it, the command stops and names the option. If an option gives a fact
that the source gives, the command also stops. Do not guess a value to make the
command continue. Ask the researcher.

## Forms that need qretools 2.0.0-alpha.13

The procedures in this skill use a local DDI file (`dist/instruments/<name>.json`).
That works in each version. A tag reference, `owner/repo@tag:name`, names an instrument
in a release in place of the file. The release must record the digest of the DDI.

| Form | First version |
|---|---|
| `qsf-map <tag reference>` | 2.0.0-alpha.12 |
| `responses <file> --map <map> --ddi <tag reference>` | 2.0.0-alpha.12 |
| `next <tag reference>` | 2.0.0-alpha.13 |
| `responses <answers.csv> --ddi <tag reference>` (no map, for example `--synthetic`) | 2.0.0-alpha.13 |
| `dataset export <dataset> --ddi <tag reference>` | 2.0.0-alpha.13 |
| `instrument = "<tag reference>"` in a harness config | a harness whose README says that `instrument` accepts a tag reference (it needs qretools 2.0.0-alpha.13) |

Version 2.0.0-alpha.13 is not released yet. Until it is, use a local DDI file in these
forms.

## References

- [reference/commands.md](reference/commands.md): the commands, their options and their
  exit codes.
- [reference/datasets.md](reference/datasets.md): column maps, datasets, provenance,
  views and access levels.
- [reference/harness.md](reference/harness.md): synthetic answers with qretools-harness.
- [examples/gss-2010.md](examples/gss-2010.md): real GSS 2010 answers and synthetic
  answers to the same instrument, compared.
