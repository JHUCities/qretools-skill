# Maps, datasets and views

The authority is the
[README of the command](https://github.com/JHUCities/qretools/blob/main/packages/cli/README.md),
sections "Read a data file", "Read an answers file", "The provenance" and "Export a
dataset".

## The words

| Word | What it means |
|---|---|
| instrument | A survey: its questions and their order. `build` writes its DDI to `dist/instruments/<name>.json`. |
| release | A tag of a workspace, for example `v3`. An instrument reads another workspace at a tag. |
| tag reference | `owner/repo@tag:name`: the instrument `instruments/<name>.yaml` in a release. A workspace in a folder is `owner/repo/folder@tag:name`. |
| map | A column map. It gives the DDI variable of each column of a data file. |
| dataset | A DDI document that holds the answers and their provenance. |
| provenance | Who answered, how the questions were asked, the dates, and the digest of each input file. |
| view | What `dataset export` writes: CSV values, or the provenance only. |
| access level | A level in the `access` list of `workspace.yaml`, from the most open to the most closed. |

## The map of a data file

A map is JSON. Its file name is `maps/<instrument>-ddi-<source>.columns.json`.
`<source>` is a short name of letters and digits. The `source` field of the map must
give the same name, or the command refuses the map. The source `qsf` is a Qualtrics map.

A map holds no answers. Commit it in `maps/`. The main fields are:

- `source`: the short name.
- `ddi`: `sha256:` and the digest of the DDI that the map was made from. If the DDI is
  different, `responses` gives a warning.
- `answered_by` (necessary): who answered, as a term of the DDI vocabulary
  DataSourceType, for example `ResearchData.Published`.
- `citation`: `title`, and optional `publisher` and `url`, of the data file.
- `respondent`: the header of the column that identifies each respondent.
- `values`: translations from a value in the file to a code of the variable. `null`
  means no answer.
- `columns`: for each column, its header, the URN of its variable and, if necessary, its
  own `values`, `filtered: true` or `censored`.
- `metadata`: columns that no respondent is asked, for example the mode, the form or a
  weight.
- `weighting`: the advice of the publisher about the weights, its source, and the date
  that it was read.
- `interview_date`: the column that gives the date of each interview.

To make a map:

1. Run `build`.
2. Write the map, or run the tool of the workspace that writes it.
3. Get the digest of the DDI with `shasum -a 256 dist/instruments/<name>.json`.
4. Write `sha256:` and the digest in `ddi`.
5. Commit the map.

When the DDI of the instrument changes, its digest changes. Then make the map again.

## Make a dataset from a data file

**Caution: write the CSV and the dataset outside the repository. They hold answers.**

1. Convert the data file to CSV with one header row.
2. Keep only the rows of one instrument, for example of one year.
3. Run `npx -y qretools@<version> responses <data.csv> --map <map> --ddi <name.json> -o <outside>/<dataset>.json`.
4. If the map has no mode column, add `--asked-by <term>`.
5. Read the findings on stderr. A finding never shows a value.

## Make a dataset from an answers file

An answers file is CSV with the header `respondent,variable,row,value`. Each variable is
a URN of the DDI, or `start` or `end`. The `row` cell is empty. For no answer, write no
line. Do not give the output of `dataset export --csv` as an answers file. It has the
URNs of the dataset in place of `start` and `end`.

- For model answers, give `--synthetic --runtime <name>`. The dataset then says that the
  questions were asked in a simulation, and that "synthetic" answered.
- For real answers, give `--answered-by <term> --asked-by <term>`.

If the command finds an error, it writes no dataset.

## The provenance

A dataset records:

- the instrument and its version;
- the dates of the collection, from the earliest start and the latest end of the
  interviews (no option gives them);
- how the respondents were asked (DDI vocabulary ModeOfCollection);
- who answered (DDI vocabulary DataSourceType);
- for a map with weights, a `Weighting` that holds the advice of the publisher;
- the SHA-256 digest of all values (`DataFingerprint`);
- the SHA-256 digest and size of the source file, the map, the DDI, and each `--file`;
- the variables with a value (`present`) and, for a map, the variables that the map does
  not read (`not-in-source`).

## Views

Give `dataset export` the DDI that the dataset was made from.

1. Run `dataset export <dataset.json> --provenance --ddi <name.json> -o <file>`. You can
   publish this view.
2. Run `dataset export <dataset.json> --csv --ddi <name.json> -o <outside>/<file.csv>`.
   Do not publish this view.

The CSV view is long: `respondent,variable,row,value`, one value on each line, sorted by
respondent and then by the URN of the variable. It does not hold `respondent`, `start`
or `end`.

**Caution: open a CSV view in R, Python or the import of a spreadsheet. Do not open it
with a double click. A text answer can hold a formula, and a spreadsheet can run it.**

## Access levels

The dataset holds all the values. `dataset export` filters only the view.

- If the DDI declares access levels, `--access <level>` is necessary. The view keeps the
  variables at that level and at each more open level. A note on stderr names each
  variable that the command removed.
- If the DDI declares no access levels, the command refuses `--access`.
- `respondent`, `start`, `end` and the metadata columns are at the most closed level,
  unless the map gives a metadata column its own `access`. Below the most closed level,
  the CSV view numbers the respondents from 1, and the provenance view has no dates.

**Caution: the command does not remove a computed value. A value computed from a
restricted answer has no level.**

## Compare two datasets

1. Export the provenance view of each dataset.
2. In each provenance view, find the `OtherMaterial` whose `TypeOfMaterial` is
   `instrument DDI`. Its `UserID` (type `SHA-256`) is the digest of the DDI.
3. If the digests are different, do not compare the datasets. Tell the researcher.
4. Export the CSV view of each dataset with the same `--ddi` file. If the DDI is not the
   DDI of a dataset, `dataset export` refuses (`dataset-ddi`).
5. Compare the values of each variable URN in the two views.

Do not join the two views on `respondent`. The respondents of two datasets are different
people. Below the most closed level, the view also replaces each identifier with a
number.
