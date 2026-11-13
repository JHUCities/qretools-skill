# qretools-skill

This repository holds an agent skill for survey research with
[QREtools](https://github.com/JHUCities/qretools). The skill tells an AI agent how to do
these tasks:

- check and build a workspace of survey questions;
- make a DDI dataset from real answers, or from the synthetic answers of a language model;
- export the views of a dataset;
- compare two datasets.

The skill also gives the agent rules that protect respondents. The rules also make sure
that the results do not mislead a reader.

The skill is the folder `qretools-research/`, in the Agent Skills format:

- `SKILL.md`: when to use the skill, the terms, the sequence of the work, and the rules
  for research.
- `reference/commands.md`: the `qretools` commands.
- `reference/workspace.md`: the files of a workspace, `build`, maps, and Qualtrics.
- `reference/datasets.md`: datasets, the facts that a dataset records, the provenance,
  views and access levels.
- `reference/harness.md`: synthetic answers with
  [qretools-harness](https://github.com/JHUCities/qretools-harness).
- `examples/gss-2010.md`: a comparison of real answers from the General Social Survey
  (GSS) with synthetic answers to the same instrument. The example uses
  [qretools-gss](https://github.com/JHUCities/qretools-gss). This repository holds no
  GSS data.

## Install

### Claude Code

1. Clone this repository:
   `git clone https://github.com/JHUCities/qretools-skill`.
2. If the folder for personal skills does not exist, make it: `mkdir -p ~/.claude/skills`.
3. Link the skill into that folder:
   `ln -s "$PWD/qretools-skill/qretools-research" ~/.claude/skills/qretools-research`.

If you use a link, the skill changes each time that you pull this repository. To use the
skill in one project only, link or copy the folder into `.claude/skills/` in that project.

### Other agents

Copy or link the folder `qretools-research/` into the skills folder of the agent. The
agent must read the Agent Skills format: a folder with a `SKILL.md` that has a `name` and
a `description`. If the agent does not read skills, give it `SKILL.md` as instructions.
Keep the other files at the paths that the links in `SKILL.md` give.

## Versions

The skill is for qretools 2.0.0-alpha.13 or later. That version is the first version
that reads a DDI URL. When a new version of qretools changes a command, update `SKILL.md`
and the reference files.

## Licence

MIT. See [LICENSE](LICENSE).
