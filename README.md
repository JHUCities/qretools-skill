# qretools-skill

An agent skill for survey research with
[QREtools](https://github.com/JHUCities/qretools). It tells an AI agent how to do these
tasks:

- check and build a workspace of survey questions;
- make a DDI dataset from real data, or from the synthetic answers of a model;
- export the views of a dataset;
- compare two datasets.

It also gives the agent rules that protect respondents and keep the results honest.

The skill is the folder `qretools-research/`, in the Agent Skills format:

- `SKILL.md`: when to use the skill, the terms, the sequence of the work, and the rules
  for research.
- `reference/commands.md`: the `qretools` commands.
- `reference/workspace.md`: the files of a workspace, `build`, maps, and Qualtrics.
- `reference/datasets.md`: datasets, the rule for facts, provenance, views and access
  levels.
- `reference/harness.md`: synthetic answers with
  [qretools-harness](https://github.com/JHUCities/qretools-harness).
- `examples/gss-2010.md`: real answers from the General Social Survey and synthetic
  answers to the same instrument, compared. It uses
  [qretools-gss](https://github.com/JHUCities/qretools-gss). The repository holds no
  GSS data.

## Install

### Claude Code

1. Clone this repository:
   `git clone https://github.com/JHUCities/qretools-skill`.
2. Make the folder for personal skills, if it does not exist: `mkdir -p ~/.claude/skills`.
3. Link the skill into that folder:
   `ln -s "$PWD/qretools-skill/qretools-research" ~/.claude/skills/qretools-research`.

A link gets each update when you pull this repository. To use the skill in one project
only, link or copy the folder into `.claude/skills/` in that project.

### Other agents

Copy or link the folder `qretools-research/` into the skills folder of the agent. The
agent must read Agent Skills: a folder with a `SKILL.md` that has a `name` and a
`description`. If the agent does not read skills, give it `SKILL.md` as instructions.
Keep the other files where the links in `SKILL.md` find them.

## Versions

The skill is written for qretools 2.0.0-alpha.13 or later, the first version that reads
a DDI URL. When a new version of qretools changes a command, update `SKILL.md` and the
reference files.

## Licence

MIT. See [LICENSE](LICENSE).
