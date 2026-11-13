# Synthetic answers with qretools-harness

The authority is the
[README of qretools-harness](https://github.com/JHUCities/qretools-harness/blob/main/README.md).

The harness asks a language model the questions of an instrument. The model answers as
one or more respondents, in a simulation. The harness then runs `qretools responses
--synthetic` to make the dataset. It makes only synthetic datasets. For real answers,
use `qretools responses` with a map (see [datasets.md](datasets.md)).

The two programs do different work:

- qretools decides the next question (`next`) and writes all the DDI.
- The harness calls the model. It does not read the routing.

## Install

You need uv, Node 22 or later with `npx`, and a model server with an
OpenAI-compatible API, for example llama.cpp.

```sh
uv tool install git+https://github.com/JHUCities/qretools-harness
```

The harness runs the one qretools version that it pins. Its README names that version.

## Run a model through an instrument

**Caution: put the run folder outside each workspace. The folder must not exist, or it
must be empty.**

1. Run `qretools build` in the workspace.
2. Write a config. Start from `examples/config.toml` in the harness repository.
3. Run `qretools-harness check-config config.toml`.
4. Correct each problem that `check-config` gives.
5. Start the model server. For llama.cpp, give one slot (`-np 1`). llama.cpp gives the
   same answers again only with one slot.
6. Run `qretools-harness respond config.toml -n <respondents> -o <run folder>`.
7. Read the exit code. If it is 1, read the reasons that the harness gives.

## The config

The config is TOML. These keys are the most important:

- `instrument` (before the first table): the path of the DDI of the instrument.
- `[model]`: `endpoint`, `name` and `timeout`.
- `[generation]`: settings that the harness sends to the server. Give `seed`.
- `[run]`: `system` (the system prompt), `missing`, `no_answer` and `draw`. Give
  `no_answer`.
- `[inputs]`: a value for each input that the route needs.
- `[prompts]`: more text for one question, keyed by the URN of the question.

Do not write an API key in the config. The run folder keeps a copy of it.

The harness refuses an instrument that has a roster.

## What a run writes

| File | What it holds |
|---|---|
| `answers.csv` | One line for each value, with the URN of its variable. |
| `traces.jsonl` | The settings of the run, then one line for each event. |
| `config.toml` | A copy of the config. |
| `dataset.json` | The dataset. It records the digests of `traces.jsonl` and `config.toml`. |

The first line of `traces.jsonl` records each setting that can change the result:

- the command line and the seed of each respondent;
- the config;
- the path of the instrument and its digest;
- the versions of qretools and of the harness;
- what the server says about itself.

The dataset holds only the respondents that finished.

**Caution: do not publish `answers.csv` or `traces.jsonl` without a check. The traces
can hold local paths of the server's computer.** To publish, export the provenance view:

```sh
npx -y qretools@<version> dataset export <run folder>/dataset.json --provenance --ddi <name.json> -o provenance.json
```

If the DDI declares access levels, also give `--access <level>`.
