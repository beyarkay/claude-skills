---
name: quickplot
description: Quick honest terminal distribution plots (sparkline histogram + box-and-whisker) of numeric data. Use when the user wants a fast in-terminal look at the distribution/spread of some values (e.g. coherence/alignment scores across models) — "histogram of these", "show me the distribution", "plot these numbers", "box plot of", "what do these look like". For throwaway terminal viz, NOT publication plots.
---

# quickplot

Turn raw numbers into a compact unicode **sparkline histogram + box-and-whisker
plot, plus a full numeric summary**, printed in the terminal. For
quick-and-dirty "what does this distribution look like" checks — real/paper
plots happen elsewhere.

Group name on the left, graphics (histogram over box-&-whisker) on the right,
all on a shared fixed x-axis; the full numeric summary is a plain list below:

```
coherence (0-100)

Gemma           ▂   ▂     ▂ ▂ ▂   ▂▂▂▂▂▄▄▂▂█▄▄▂▂▄   ▄      <- histogram (sparkline)
                ├───────────────────[====┃===]──────┤      <- box & whisker
Olmo   ▄▂▂▂▆▂▄▄▂▆▄▄▄█▄▂
       ├────[===┃==]──┤
Qwen                                     ▁▁█▇▅▅▄▇▁
                                         ├─[=┃]──┤
       └──────────┴──────────┴───────────┴──────────┴      <- shared axis (once)
       44         57.7       71.4        86.3     100

Gemma: n=30  min 55  q1 80.2  med 86.5  q3 90.8  max 100  mean 84.1  sd 10.5
Olmo:  n=30  min 44  q1 50.2  med 55  q3 59  max 63  mean 54.6  sd 5.51
Qwen:  n=30  min 86  q1 89.2  med 91  q3 93  max 96  mean 91.1  sd 2.52
```

Box & whisker reads `├` min · `[` q1 · `┃` median · `]` q3 · `┤` max. Stacking
several groups lets you eyeball-compare spreads since every box sits on the
same axis.

## The contract — why this is a script, not eyeballing

`plot.py` is the **only** thing allowed to turn the data into a picture or a
summary. It computes the histogram, the box plot, and the full seven-number
summary itself. You run it and **paste its output verbatim**.

Do NOT, separately, describe the distribution in prose ("looks roughly
normal", "Olmo is worse", "tightly clustered"), round the numbers, or report a
subset of them. The user explicitly does not want that layer — the script's
output is the answer. The plot is only a visual aid; the authoritative numbers
are the text summary line, which prints n, min, q1, median, q3, max, mean, and
std for every group. Unparseable values are counted and flagged (`⚠ N
unparseable`), never silently dropped. If they ask a follow-up about the
numbers, answer from that printed summary.

## How to run

Self-contained `uv` script (only dep: `numpy`) — nothing to install. Pass a
file or pipe via stdin:

```bash
~/.claude/skills/quickplot/plot.py data.json
echo '{"Gemma":[...],"Olmo":[...],"Qwen":[...]}' | ~/.claude/skills/quickplot/plot.py --title coherence
cat scores.csv | ~/.claude/skills/quickplot/plot.py
```

Typical flow: the data already exists (a list in the conversation, a column in
a CSV/parquet, an array in a script). Get it into one of the input formats
below — usually by writing a small JSON object to `/tmp` or piping — then run
the script and show what it prints.

## Input formats (auto-detected)

| Input                                    | Result               |
| ---------------------------------------- | -------------------- |
| JSON object `{"Gemma":[..],"Olmo":[..]}` | one panel per key    |
| JSON array `[1,2,3,...]`                 | a single panel       |
| CSV/TSV with a header row of names       | one panel per column |
| plain whitespace/newline numbers         | a single panel       |

For the common "models × metrics" case (Gemma/Olmo/Qwen each scored on
coherence + alignment), the cleanest input is a flat JSON object with one key
per group, e.g. `{"Gemma coherence":[...], "Gemma alignment":[...], ...}`. All
groups share the **same fixed x-range** so the panels are directly comparable.

## Flags

- `--width N` — plot width in characters (default 46)
- `--title TEXT` — title printed once above the panels
- `--summary-only` — print just the numeric summary lines, no plot

## Out of scope

Scatter, line, or time-series plots, and anything headed for a paper or slide
deck. This is terminal distribution-at-a-glance only.
