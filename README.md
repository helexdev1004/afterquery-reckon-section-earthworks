# Compute cut, fill and topsoil at road cross-sections

A harbor-format benchmark task, authored for the AfterQuery Experts programme as data
annotation for AI evaluation. Task slug `afterquery/reckon-section-earthworks` — **Operations / Highway Earthworks**.

An AI agent gets a container, this brief and nothing else. It works alone for
4 hours. Then a sealed verifier it never sees scores the result `0` or `1`.

## What it asks for

Cut, fill and topsoil at forty cross-sections

The full brief is in [`instruction.md`](instruction.md).

| | |
| --- | --- |
| **Deliverable** | `/app/areas.csv` |
| **Agent budget** | 4 hours (14400s) |
| **Expert estimate** | 6 hours |
| **Machine** | 1 CPU, 2048 MB RAM, 0 GPU |
| **Verifier budget** | 300s |
| **Tags** | `earthworks`, `cross-sections`, `survey-data`, `computational-geometry`, `python` |

## Why it is hard

Taking cut, fill and topsoil strip off levelled cross-sections is everyday work for a highways quantity surveyor or an earthworks estimator, done from exactly what ships here: the surveyors' field books, the designer's template and formation levels, the land schedule and the office's note. The data is synthetic and was made truth first: the ground of 5 schemes was laid down as a surface in the designer's own frame, 321 cross-sections and 10838 levels of it, and only then was each field book written out in its surveyor's units and with his hand. The sealed figures are exact by construction and the records are static, so nothing is computed at marking time.

<details>
<summary>Full note</summary>

The note gives every rule the marking uses: how a section is booked, how the template sits and where its side slopes stop, what counts as cut and as fill, and how deep each kind of land is stripped. Applied to the field books as booked it gets 0 of 40 sections, because four surveyors booked in a unit the note does not use - offsets in feet, levels in feet, levels in millimetres, chainages in kilometres. Each is glaring, each is only a unit, and leaving any one alone costs eight sections against an allowance of 4. What decides the task is a hand. The note books an offset positive to the right of the centreline looking up chainage; 2 of the 5 surveyors booked theirs positive to the left, one of them a surveyor with an odd unit and one the surveyor who otherwise kept to the note. Nothing says so, and the note's own reading fails without a sign: a section is booked as far one way as the other, the template sits on it either way, the side slopes find ground, and every figure comes out - for the mirror image of the ground, under a template that is wider on one side and a land schedule that is not the same either side. It costs 16 of the 40. No first-order figure shows it. The schemes cross no ditch, bank or hedge; the ground falls across the road whichever way it falls; the two ends of a section are statistically alike; and the five schemes are different ground, so no book can be laid against another. What the land leaves in the levels is second order: ploughed ground is rougher from one staff position to the next than pasture. Measured as how far a booked level stands from the straight line between its two neighbours, the middle value on arable land is at least 3.1 times that on pasture in every book read in its true hand, and at most 1.0 times in the other. The fall of the ground and its long undulations are far larger than either roughness, and the odd gateway or track is rough on any land, so the plain spread of levels by kind of land says nothing; and the land schedule is a file the work needs anyway, for the topsoil. tests/audit_task.py measures all of this against the sealed figures and refuses to pass otherwise: of the 48 combinations of readings the proper one gets 40, the best other 32 against a bar of 36, every book taken as left-handed 16, and the best single row given for every section 2.

</details>

## How it is verified

The verifier reads /app/areas.csv as text with the csv module and compares each wanted section's three areas with the sealed figures in /tests/sealed.json, each within 0.2 square metres; at most 4 of the 40 rows may be wrong. Nothing the agent wrote is imported or executed, the sealed figures exist only in the verifier image, /tests is mode 700 there and the agent's container never mounts it. The 0.2 is rounding room: the sealed figures are those the reference gets from the books as booked, feet and all, and every wanted section was chosen so that the ground at each edge of the formation is at least a quarter of a metre clear of formation level in either hand - so the choice of cut or fill slope is never in doubt - and so that the wrong hand moves at least one of its areas by a square metre and a half or more.

<details>
<summary>Full note</summary>

Nothing in the agent's container attests any area, no total has to balance, and a wrong sheet is as self-consistent as the right one. The reward is binary and written to /logs/verifier/reward.txt on every path; an empty container scores 0 and the reference scores 1. The generator is seeded and shipped.

</details>

<details>
<summary>Reference approach (spoiler)</summary>

solution/take_off.py, with solution/template.py, reads only /app and writes /app/areas.csv with the standard library alone. It finds each field book's units by setting its chainages, offsets and levels against the formation levels and the reach of the land schedule. For each scheme it measures how far every booked level stands from the line between its neighbours, groups that by the kind of land the point stands on - once with offsets as booked and once mirrored - and keeps the hand under which arable comes out rougher than pasture. It then lays the template on each wanted section: a level formation, a cut or a fill slope from each edge according to the ground there, run out to where it first meets the ground; integrates ground above template and template above ground exactly between those two points, piece by piece between booked points and crossings; and adds the strip depth of each kind of land over the same width. Run in the task's own container it gets 40 of 40.

</details>

## Layout

```
instruction.md          the brief - the only thing the agent is told
task.toml               artifacts, timeouts, resources, metadata
environment/            Dockerfile and data for the agent's container
solution/solve.sh       reference solution; scores 1 by construction
tests/                  verifier image, grading logic and ground truth
```

## Running it

With [harbor](https://github.com/laude-institute/harbor) installed and Docker running:

```bash
uv tool install harbor

harbor run -p . -a oracle -e docker   # reference solution - must score 1
harbor run -p . -a nop    -e docker   # nothing happens    - must score 0
```

Those two runs are the calibration the bundle has to satisfy: the oracle run substitutes
`solution/solve.sh` for the agent and must score `1`, the nop run does nothing and must
score `0`. Anything in between means the verifier is measuring the wrong thing.

## Reading this

`tests/` and `solution/` hold the answers. To try the task honestly, read only
`instruction.md` and `environment/`.

---

Part of a set of 38 tasks indexed at
[afterquery-data-annotation](https://github.com/helexdev1004/afterquery-data-annotation).
