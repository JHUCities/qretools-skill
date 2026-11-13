# The workspace, its maps and Qualtrics

The authority is the
[README of the command](https://github.com/JHUCities/qretools/blob/main/packages/cli/README.md),
sections "Build", "Make the QSF column map" and "Export from Qualtrics", and the README of
[qretools-gss](https://github.com/JHUCities/qretools-gss), section "Maps and datasets".

## The files that you commit

A workspace is one folder: `workspace.yaml`, `questions/<folder>/`, the shared folders
(for example `scales/`), `missing.yaml`, `select_all.yaml` and `instruments/`. For data,
it also has these folders:

| Folder | What it holds |
|---|---|
| `instruments/` | One YAML file for each instrument. |
| `maps/` | The column maps, for example `gss_2010-ddi-qsf.columns.json` and `gss_2010-ddi-gss.columns.json`. |
| `tools/` | The scripts of the workspace that write maps. |

The rules for `maps/`:

- The folder holds only maps.
- A person writes a map by hand, or `qsf-map` writes it, or a script in `tools/` writes it.
- A map holds no answers, so you can commit it.
- `build` does not make a map. It reads and checks each map.
- Do not edit a map that a script writes. Change the script, then run it again.

## Build a release

1. Commit your changes. `build` reads the last commit, not your working files.
2. Run `npx -y qretools@<version> check .`.
3. Correct each finding, then commit again.
4. Run `npx -y qretools@<version> build . -o dist`.

**Caution: do not make a shallow clone. `build` needs the full history, because the
versions come from it. In CI, use `fetch-depth: 0`.**

`build` writes these files:

| File | What it holds |
|---|---|
| `dist/ddi.json` | The DDI of the questions of the workspace. |
| `dist/instruments/<name>.json` | The DDI of an instrument. |
| `dist/instruments/<name>.qsf` | The Qualtrics survey of the instrument. If Qualtrics cannot run the instrument, `build` gives a warning and writes no survey. |
| `dist/instruments/<map file>` | A copy of each map in `maps/`. |

`build` checks each map in `maps/`:

- If the map names no instrument of the workspace, `build` fails.
- If the `source` field is missing, or is not the source in the file name, `build` fails.
- If the file is not a column map, `build` fails.
- If the map was made from a different DDI, `build` gives a warning (`build-map-ddi`)
  and copies the map. The warning names the digest of that DDI.

A map from a different DDI can still be correct. `responses` checks each value when it
reads the data.

A tag names a release. A workspace can publish each release. For example, qretools-gss
publishes the release `v15` at `https://jhucities.github.io/qretools-gss/v15/`. The DDI
URL of its instrument `gss_2010` is
`https://jhucities.github.io/qretools-gss/v15/instruments/gss_2010.json`.

## Write a data map

The map of a data file is JSON. Its file name is
`maps/<instrument>-ddi-<source>.columns.json`. `<source>` is a short name of letters and
digits, and the `source` field gives the same name. These are the main fields:

- `source`: the short name.
- `ddi`: `sha256:` and the digest of the DDI that the map was made from.
- `answered_by` (necessary): who answered, as a term of the DDI vocabulary
  DataSourceType, for example `ResearchData.Published`.
- `citation`: the `title`, and optional `publisher` and `url`, of the data file.
- `respondent`: the header of the column that identifies each respondent.
- `values`: translations from a value in the file to a code of the variable. `null`
  means no answer.
- `columns`: for each column, its header, the URN of its variable and, if necessary, its
  own `values`, `filtered: true` or `censored`. `filtered` means that the file does not
  hold the values of the variable in its form: the command reads no value from the
  column. `censored` lists values that mean a range, for example a top code ("8 or
  more"): the command writes no value for them.
- `metadata`: the columns that no respondent is asked, for example the mode, the form or
  a weight.
- `weighting`: the advice of the publisher about the weights, its source, and the date
  that it was read.
- `interview_date`: the column that gives the date of each interview.

To write a data map:

1. Run `build`.
2. Write the map, or run the script of the workspace that writes it.
3. Get the digest of the DDI: `shasum -a 256 dist/instruments/<name>.json`.
4. Write `sha256:` and the digest in `ddi`. A script can do this step.
5. Commit the map.

`qsf-map` writes the digest in a Qualtrics map itself.

## Put a survey in Qualtrics

1. Find the release that holds the instrument, for example `v15`.
2. Record the tag of the release.
3. Download `instruments/<name>.qsf` from the release.
4. In Qualtrics, import the file under Projects → Create project → Survey.
5. Field the survey.

The survey holds the URN of its instrument and the digest of its DDI, in `qretools_*`
fields. Each response in an export holds these fields.

## Make the Qualtrics map

Make the map after you field the survey.

1. Run `npx -y qretools@<release version> qsf-map <owner>/<repo>@<tag>:<name> -o maps`.
   Use the tag that you recorded. The map records the release.

   **Caution: use the qretools version that built the release. The message of the tag
   names it. A tag reference makes the DDI again, and it stops if the digest is different.**
2. Read each `qsf-unmapped` warning. The map has no column for the item that it names.
3. Commit `maps/<name>-ddi-qsf.columns.json`.

A workspace can have a script for this step. For example, qretools-gss has
`tools/qsf-columns.sh <instrument> <tag>`.

If you do not know the release, make the map from an export and a DDI:

1. Get an export of the survey. See "Export from Qualtrics".
2. Run `npx -y qretools@<version> qsf-map --export <export.csv> --ddi <DDI file or URL> -o maps`.
3. Read each `qsf-map-matched` finding. It tells how the command matched a question of
   the export to a question of the DDI.
4. If a match is wrong, correct the map by hand.
5. Read each `qsf-map-absent` warning.
6. Commit the map.

This command reads only the header rows of the export and the `qretools_*` fields. It
does not read an answer.

**Caution: do not make the Qualtrics map of a fielded survey again. The map names the
release of the survey that you fielded.**

## Export from Qualtrics

1. Open Data & Analysis.
2. Open Export & Import, then select Export Data.
3. Select the CSV tab.
4. Select "Export values".
5. **Caution: do step 6. If you do not, all the options of a select-all question go into
   one column, and the map cannot read it.**
6. Select "Split multi-value fields into columns".
7. Download the file.

The labels are those of Qualtrics in November 2026.

**Caution: the export holds answers. Keep it outside the repository.**

## Change an instrument

The DDI of an instrument gets its version from the history of `main`. Thus, the new DDI
exists only after the merge.

1. Merge the change to the instrument. `build` then gives a `build-map-ddi` warning for
   each map that was made from the old DDI.
2. Make each data map again from the new DDI. For example, run `tools/columns.py` in
   qretools-gss.
3. Commit the new data map in a second pull request.

Do not make the Qualtrics map again:

- If you fielded the survey, its map names the release that you fielded. Keep it.
- If you did not field the survey, no Qualtrics map exists yet. Make it after you field
  the survey.
