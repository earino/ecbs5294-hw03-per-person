# Homework 3 — Waste per person: two files, three layers

**ECBS5294 — Working with Data · due Saturday 31 October 2026, 23:59 (slot on Moodle)**

Plan for about **2 to 3 hours**. This is an estimate. If you are far past it and still stuck, post on
the Moodle forum. That helps us improve the homework. Asking for help does not lower your grade.

You do this homework alone. Every query, every count and every sentence is yours.

## Start here

| | |
|---|---|
| **The question** | How much municipal waste does each of the 27 EU members generate per person, in 2024? |
| **The files** | Eurostat's waste file (Lab 6's) and the World Bank's population pages (Lab 5's), in `data/raw/`. The setup cells turn both into tables for you. |
| **What you write** | Four questions: did every record arrive, did every value survive, the gold that answers the question, and a colleague's answer you check. Every query is yours, from an empty cell; under each question, a count. |
| **What you hand in** | `notebooks/report.ipynb`, run from top to bottom with its outputs saved, on Moodle. Nothing else. |
| **First step** | Clone the project, open the notebook, run the four setup cells. |

## Get the project

In your terminal (Git Bash on Windows, Terminal on macOS), in your course folder:

```bash
git clone https://github.com/earino/ecbs5294-hw03-per-person.git
cd ecbs5294-hw03-per-person
uv sync
```

Open **this folder** in VS Code (*File → Open Folder…*; trust the authors if it asks). Open
`notebooks/report.ipynb`, pick the `.venv` kernel, and run the first four cells. The first prints the folder it works
in: it must be this project's folder. The next two make the tables; the fourth holds the checks. You never write
those cells.

Views live in memory. After you restart the kernel, run the cells again from the top.

## The brief

> **Waste** is Eurostat's municipal waste generated (`wst_oper = 'GEN'`), in **thousand tonnes** (`unit = 'THS_T'`).
>
> **The members** are the 27 rows of the view `members`. Each has two codes: `eurostat_geo` in Eurostat's file
> (`EL` for Greece) and `worldbank_id` in the World Bank's (`GRC`).
>
> **Waste per person** is a member's total times 1,000,000 (thousand tonnes into kilograms), divided by its
> population in the same year: **kilograms per person**. Where a member has no total, it has no number: `NULL`. It
> need not equal Eurostat's own figure per person, because the two sources count population differently; you do not
> need to find out why.

## The questions

1. **Did every record arrive?** *(Block 5)* (a) Count the rows of `population`, beside the `total` the pages state.
   (b) Count the members that have a population for 2024.
2. **Did every value survive?** *(Block 6)* (a) Write `waste_silver`: the totals in thousand tonnes, the number made a
   number with `CAST`. (b) Count the numbers before your step and after it. (c) The numbers, year by year. (d) The
   members with no number for 2024.
3. **Gold: waste per person, 2024.** *(Block 6)* First, predict how many rows it will have, and how many numbers.
   Then write `per_person_2024`, which joins your silver and the population through `members`, and show it, highest
   first. Count its rows, and the rows with a number.
4. **Can you trust this answer?** A colleague's version of question 3 runs with no error, and its report gets one
   number wrong. Count, find what it lost, say why, and repair it.

Under each question: your queries, **your count**, the supplied check (questions 2 to 4; each check says `PASS` or
`FAIL`, and under a `FAIL` a line says what to look at), and **one sentence**. Then three lines of reflection on the
course, and one line on AI use.

## Rules

- **Never edit `data/raw/`.** Every number comes from a query.
- **The brief decides.** If a question and the files seem to disagree, follow the brief.
- **AI is allowed.** You must be able to explain every line of SQL you wrote. The assistant cannot see your data: run
  the counts yourself.

## Hints, if stuck

1. **Question 1.** (a) `SELECT COUNT(*) FROM population` is one half; the other is `total`, in `population_metadata`,
   `WHERE page = 1`. (b) `FROM members AS m JOIN population AS p ON p.iso3 = m.worldbank_id`, then a `WHERE` for the
   year, and `COUNT(*)`.
2. **Question 2.** (a) Block 6's cell 2 is the same step on another file. Your `FROM` is `silver_split`; your `WHERE` has
   the unit and the waste operation and nothing else (every place, every year); your number is
   `CAST(number_text AS DOUBLE) AS thousand_tonnes`. (c) `GROUP BY year`, and `ORDER BY` the count of numbers.
   (d) Question 1(b)'s join, on Eurostat's code this time, with `IS NULL` in the `WHERE`.
3. **Question 3.** Look at Greece in the cell before your query. `waste_silver` uses Eurostat's code and `population`
   the World Bank's; `members` holds both, so the query starts `FROM members AS m` and has two `JOIN`s. `COUNT(*)`
   counts every row; `COUNT(kg_per_person)` skips a `NULL`.
4. **Question 4.** (a) The claim is about the colleague's table: question 3's count, on their table. (b) Block 3's
   cell 9: a `LEFT JOIN` keeps every member, and the one with no partner has `NULL` in the colleague's columns.
   (c) What does each table call that member? (d) One line of the colleague's query changes: the join to `population`.

## Grading

The rubric is on the course site. Correct answers 45 · your counts and the checks 25 · the sentences 10 · the
reflection 15 · AI use 5. Late: one day at −10%; nothing after Sunday 23:59.

**Before you upload:** *Restart*, then *Run All*, then save. Leave anything unfinished as comments (`#` in front of each
line), so every cell runs.

## If you got lost: how to reset

**First, the gentle way:** save the notebook, *Restart* the kernel, run the cells from the top. This fixes most "lost"
notebooks, and loses nothing.

Only if that does not help: both of these **destroy work**. Read before you run them.

**Discard uncommitted changes (destructive)**: throw away edits and new files, and keep your commits.

```bash
git restore --staged --worktree .    # every tracked file back to the last commit, staged or not
git clean -fd                        # and remove new, untracked files
```

> ⚠️ This permanently deletes uncommitted changes, staged or not, and any new untracked files.

**Full reset to the starter (destructive)**: back to exactly what you cloned. This throws away your commits too.

```bash
git reset --hard origin/main
git clean -fdx
```

> ⚠️ This discards your local commits and uncommitted changes. The `-x` also removes ignored files, including the
> `.venv/` environment, so the folder matches a fresh clone. `uv sync` rebuilds the environment in a minute.
