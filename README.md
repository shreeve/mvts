# MVTS 9.10

The MUMPS Validation Test Suite, version 9.10, by MUMPS Systems Laboratory
(`MVTS V9.10;15/6/96`, `COPYRIGHT MUMPS SYSTEMS LABORATORY 1978-1996`): a
conformance suite for implementations of the M (MUMPS) language, written in
M itself. It runs inside the implementation under test and checks it
against the language standard (ANSI X11.1, later ISO/IEC 11756), test by
test.

The suite is available online from several sources, and has been for more
than two decades; for example, public test repositories for M
implementations on GitHub carry it in full. Its routines bear the notice
`COPYRIGHT MUMPS SYSTEMS LABORATORY`, but no license terms are known, and
no holder has been heard from in that time. It is used here on the working
assumption that its long, open public availability makes such use
acceptable.

## Files

| File | What it is |
|---|---|
| `mvts-9.10.rsa` | the 713 routines, in routine-save format: two header lines, then each routine's name, its lines and an empty line |
| `mvts-9.10.gsa` | the globals, in global-save format: two header lines, then each node's reference and value |
| `mvts-9.10-v4pat9.diff` | the one change from the text as published (below) |
| `NOTICE` | who owns what: the routines, and everything else (CC0) |
| `mvts-9.10.zip` | the files above, in one archive |

```
424c1c0d1c313b643b97145ffca132f31681a69d5abd06b379e6d8ade77ef1d9  mvts-9.10.rsa
e045933daf62e92230a15798ad4f82b2fea43a2c65dc33f812243d8868bd2706  mvts-9.10.gsa
```

## A brief history

MUMPS Systems Laboratory maintained the suite from 1978 to 1996, adding a
part for each edition of the standard. Version 9.10 has four parts, each
started by a driver routine, and a part for transactions:

| Part | Driver | Covers |
|---|---|---|
| Part-77 | `VV1` | the 1977 standard: commands, expressions, functions, special variables, I/O |
| Part-84 | `VV2` | the 1984 additions |
| Part-90 | `VV3` | the 1990 additions: NEW, `$ORDER`, extrinsic functions, more indirection |
| Part-94 | `VV4` | the 1995 additions: MERGE, `$QUERY`, `$NAME`, structured system variables and more |
| transactions | `VV4TP` | TSTART, TCOMMIT, TRESTART and TROLLBACK, with several jobs |

Over the years the Laboratory corrected some tests and withdrew others; the
routines say so beside each one ("test corrected in V7.3", "withdrawn in
15/2/1994 on X11.1-1990, MSL"). Version 9.10 withdraws 93 tests.

### The text in these files

`mvts-9.10.rsa` is the Laboratory's text as published, with one exception.
Copies of the suite circulated for years, and some had tests edited to
match a particular implementation's answers; those edits are not in this
file. MVTS carries its own integrity check: the routines `VINT9` and
`VINT9B` hold a checksum for each of 711 routines (the sum of the character
codes of its lines, with the line start as one space). In this file, 710
of the 711 match exactly.

The exception is `V4PAT9`, test IV-858. As published, its line reads
`S ^VCOMP=1.5(3.4(1."A",1."B",1."C"),2.(3.9"AB",3.5AN))`: the pattern match
is missing its `X?`, so the line is not valid M and the test cannot run. A
fix dated 20 June 2002 (`mvts-9.10-v4pat9.diff`) adds the two characters,
and every known copy carries it. They are exactly the routine's difference
from its checksum: `X` (88) plus `?` (63) is 151, the difference between
72422 and the table's 72271.

### The globals

The Laboratory ships no globals. MVTS keeps its settings in one global,
`^VENVIRON`, which it builds itself the first time a driver runs: it runs
its integrity check (`^VINT9`), then a 32-item questionnaire (`^VENVIRON`)
about the site, the principal input and output devices, the secondary and
tertiary devices the I/O tests write to, and where the summary report goes.
It records the answers, the time the questionnaire ended
(`^VENVIRON("COMPLETE")`) and the integrity check's result
(`^VENVIRON("INTEGRITY")`).

`mvts-9.10.gsa` is what MVTS records with generic answers:

- the eight descriptive items (customer, date, site, implementation,
  host, target and two points of contact) are each their own node's name,
  as placeholders; MVTS only prints them;
- the principal input and output are device `0`;
- the secondary and tertiary devices are the files `10` and `11`, turned
  on;
- the summary report (the VSR) goes to the principal device and to the
  file `vsr.log`;
- `COMPLETE` is a fixed `67849,0`, so the file is the same each time it
  is made;
- `INTEGRITY` is `NOT OK`, as the check records it with V4PAT9's fix.

Change the device answers to whatever the implementation under test needs;
the rest can stay.

## Using it to validate an implementation

1. Load `mvts-9.10.rsa` into the implementation's routines and
   `mvts-9.10.gsa` into a fresh database, with the implementation's own
   routine and global restore tools.
2. Run the drivers in order: `D ^VV1`, `D ^VV2`, `D ^VV3`, `D ^VV4`, and
   `D ^VV4TP` for transactions.
3. Read the results. Each test prints one line with a five-digit test
   number, its ID and its description:

   ```
      PASS  40338 IV-338  a local variable has maximum total length
   ** FAIL  30952 III-0952  NEW all
              COMPUTED ="..."
              CORRECT  ="..."
   ```

   A failure shows what the implementation computed and what the test
   expected. Each part ends with a banner such as
   `*** Standard MUMPS Validation Test Suite Version 9.10, Part-94 END ***`.

Things to know when running it:

- **It is interactive.** It asks the operator to press keys, to confirm
  the questionnaire, and to judge a few visual tests (form feeds, screen
  positioning) as pass or fail. A harness can answer every prompt.
- **An error ends only the running test.** MVTS sets no error trap for
  its tests; it keeps the entry to resume from in `^NEXT`, so a harness
  can restart the run from there after an error.
- **One block needs a second terminal.** The multi-job tests in `V1MJA`
  expect a second terminal turned on; a run without one skips that block.
- **Some results vary from run to run on any implementation.** The
  `$RANDOM` tests (I-738, I-742 to I-748) are statistical and fail now and
  then even with an ideal generator; I,III-324 times a two-second HANG in
  whole seconds; and the transaction tests that interleave jobs depend on
  the order the jobs run in.
- **A few expected answers cannot be met.** Several transaction tests
  (IV-898, IV-900 to IV-902, IV-904, IV-905, IV-908) expect to see changes
  a rolled-back transaction made, which the standard forbids, and IV-923
  expects a record that a restarted transaction cannot leave; no conforming
  implementation passes them.

A complete run of VV1 to VV4 reports about 4,250 results. An
implementation's own limits (for example a maximum number of subscripts
below what IV-767 and IV-768 use) can leave a test without a result.
