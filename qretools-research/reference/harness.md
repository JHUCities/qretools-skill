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

The harness runs the one qretools version that it pins. Its README names that version. A
DDI URL in the config needs a harness that pins qretools 2.0.0-alpha.13 or later.

## The words

- A **run** is one `qretools-harness respond` command. It interviews the model one time
  for each respondent. It makes one dataset from the respondents that finish.
- A **respondent** is one interview. The model answers as one person, with its own seed
  and its own conversation.

## Run a model through an instrument

**Caution: put the run folder outside each workspace. The folder must not exist, or it
must be empty.**

1. Get the DDI of the instrument: the DDI URL of a release, or a DDI file.
2. Write a config. Start from `examples/gss_2010/config.toml` in the harness repository.
   Set `instrument` to the URL or to the path of the file.

   **Caution: a DDI URL and `examples/gss_2010/` need a harness that pins qretools
   2.0.0-alpha.13 or later. Read the version in the README of the harness. With an
   earlier harness, download the DDI (`curl -fsSL -o <file> <DDI URL>`), give the path
   of the file, and start from `examples/config.toml`.**
3. Run `qretools-harness check-config config.toml`.
4. Correct each problem that `check-config` gives.
5. Start the model server. For llama.cpp, give one slot (`-np 1`). llama.cpp gives the
   same answers again only with one slot.
6. Run `qretools-harness respond config.toml -n <respondents> -o <run folder>`.
7. Read the exit code. If it is 1, read the reasons that the harness gives.

## The config

The config is TOML. These keys are the most important:

- `instrument` (before the first table): the DDI URL of the instrument, or the path of
  its DDI file. A relative path starts from the folder of the config.
- `[model]`: `endpoint`, `name` and `timeout`.
- `[generation]`: settings that the harness sends to the server. Give `seed`.
- `[run]`: `system` (the system prompt), `missing`, `no_answer` and `draw`. Give
  `no_answer`.
- `[inputs]`: a value for each input that the route needs.
- `[prompts]`: more text for one question, keyed by the URN of the question.

Do not write an API key in the config. The run folder keeps a copy of it.

The harness refuses an instrument that has a roster.

## What `respond` does

For each respondent *i*, the harness does these steps:

1. It gives the respondent the seed `generation.seed + i - 1`.
2. It records the start time.
3. It asks `qretools next` for the next step. The reply is one of three kinds:
   - `input`: the harness gives the value from `[inputs]`, or draws one if `run.draw`
     names the input. If the config gives neither, it sends "not known" (`null`).
   - `ask`: the harness sends the question to the model and records the answer.
   - `end`: the harness records the end time.
4. If the harness or qretools does not accept an answer, the harness asks again and
   gives the reason. After the third answer that it does not accept, it records the
   `run.no_answer` code.
5. If qretools does not accept the `no_answer` code, that respondent stops.

Then the harness writes the run folder and runs `qretools responses --synthetic`. That
command makes the dataset.

If the model server or qretools fails during an interview, that respondent stops, and
the run goes on. If no respondent finishes, the harness writes no dataset.

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
- where the instrument came from, and its digest;
- the versions of qretools and of the harness;
- what the server says about itself.

The dataset holds only the respondents that finished.

**Caution: do not publish `answers.csv` or `traces.jsonl` without a check. The traces
can hold local paths of the server's computer.** To publish, export the provenance view:

```sh
npx -y qretools@<version> dataset export <run folder>/dataset.json --provenance --ddi <DDI> -o provenance.json
```

If the DDI declares access levels, also give `--access <level>`.
