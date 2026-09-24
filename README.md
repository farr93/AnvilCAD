# ANVIL CAD

A parametric solid modeller built for fabrication work — tube bending, laser-cut
plate, sheet metal, and 3D printing.

It is a real CAD program, not a mesh editor. Geometry is exact B-rep
(OpenCASCADE, the same kernel FreeCAD is built on), which means circles stay
circles, fillets are real surfaces, and the DXF you send to a laser cutter
contains true arcs instead of a polygon approximation of one.

---

## Start it

Double-click **`Anvil CAD.vbs`**.

That's it. The first launch takes a few minutes while it installs the geometry
kernel (~600 MB, one time, into a local `.venv` folder — nothing is installed
system-wide and nothing outside this folder is touched). Versions are pinned in
`requirements.txt`, so every install gets the kernel this build was tested on.

**Want it on your desktop?** Double-click **`Create Desktop Shortcut.vbs`** once.
It puts an *Anvil CAD* icon on your desktop that you can then drag to the
taskbar to pin. Delete the shortcut to undo it.

If it doesn't open, run **`START.bat`** instead — same thing, but it keeps a
console window open so you can read the error.

**Requirements:** Windows 11, Python 3.11+ on your PATH, and Edge or Chrome
(Anvil needs WebGL2; Edge ships with Windows so this is almost certainly already
fine).

Anvil opens in its own window with no browser chrome. Pin `Anvil CAD.vbs` to your
taskbar and it behaves like any other installed program.

---

## What it does

### Sketching
Exact 2D geometry with a real constraint solver.

- Line, rectangle (corner / centre / 3-point), circle (centre-radius / 3-point),
  arc (centre / 3-point / tangent), ellipse, polygon, slot, spline, point
- Trim, extend, offset, fillet, chamfer, mirror, linear and circular pattern
- Constraints: coincident, horizontal, vertical, parallel, perpendicular,
  tangent, equal, concentric, symmetric, fix
- Dimensions: distance, horizontal/vertical distance, radius, diameter, angle
- Snapping to endpoints, midpoints, centres, quadrants, intersections, and along
  entity extensions — with an on-screen glyph telling you *why* it snapped
- A degrees-of-freedom readout that goes green when the sketch is fully defined,
  and highlights the offending constraints in red when they conflict

### Modelling
History-based and parametric. Every feature stays editable.

- Extrude (blind / symmetric / two-sided / through-all / to-body, with draft and
  thin-wall), revolve, sweep along a path, loft
- Holes: simple, counterbore, countersink, tapped — with the tap drill size
  computed from the thread callout
- Fillet and chamfer on real edges, shell, draft
- Boolean union / subtract / intersect
- Linear, circular and mirror patterns
- Primitives: box, cylinder, sphere, cone, torus, wedge
- Stock sections: flat bar, round bar, angle, channel, W beam, tee, square tube,
  rectangular tube, round tube, pipe, and gusset plate — pick the size off a
  list instead of looking up three numbers. 710 sizes, steel and aluminium,
  plus sheet and plate by gauge.
  Every one stays a parametric feature you can edit.

### Fabrication
The features this was actually built for.

- **Tube Bend** — enter OD, wall, centreline radius and a bend table (angle,
  rotation, run), get the bent tube plus the numbers you take to the saw: flat
  length, cut length, per-bend arc length, and springback-compensated die angle.
  It warns you when the CLR is tight enough to kink the tube — and it says which
  stock that warning is *about*. The kink rule, the ovality rule and the "above
  2 is comfortable" habit are all rules about a **wall**, so a solid bar gets the
  geometry and an explicit refusal instead: no sourced forming limit exists for a
  bent solid section, and none is invented here. The 25% outer-fibre stretch
  behind CLR/OD = 2 is a mild steel figure, and the panel says so, because you
  can pick 6061 off the same list.
- **Sheet flange** and **unfold** — model the folded part, flatten it, export the
  flat pattern with correct bend allowance. Unfold walks out from whichever face
  you want to stay put, develops every bend it finds — folded toward you or away,
  any angle up to a closed hem — and gives you the blank size and a bend table.
  It checks the flat pattern back against the material in the folded part, and
  tells you what is wrong instead of handing you a blank when it cannot lay the
  body out honestly. **The flat pattern says where to fold, not just where to
  cut:** the DXF and SVG carry fold lines on their own layers — `BEND-UP`,
  `BEND-DOWN`, the two tangent lines of the bend zone, and the angle as readable
  text. Three lines per bend, because the line the brake is set to is the
  *centre* of the developed bend and the tangent lines bounding it are a
  different instruction — 3.39 mm away on 3 mm material at 90°. A fold nobody
  gave a direction to goes on a plain `BEND` layer, so "we do not know" cannot
  read as "either way is fine".
- **Stock sizes** — the section tools carry a size list: DOM round tube, A500
  square and rectangular tube, A36 flat, round bar and angle, AISC channels,
  W beams and tees, NPS pipe, and 192 rows of 6061-T6 aluminium to ASTM B221.
  Pick a size and it fills in the dimensions and names the feature after the
  designation, so the tree reads like a cut list. Your cut length is never
  touched by the list — you always say how long. **Pipe is a lookup table, not
  arithmetic:** NPS 1 pipe is 33.4 mm across, not 25.4, so pick *1 in NPS Sch 40*
  off the list. Typing 1 into the OD field gets you a 1" tube, which is different
  steel. Steel and aluminium share the lists, so the designation carries the
  alloy: *2 x 2 x 0.250 (6061-T6)* is a different row from the A500 tube that
  answers to the same three numbers, and an extruded corner is square where a
  rolled one is not.
- **Measure** — distance, angle, radius, area, mass. Shows mm and inches at once,
  because you were going to convert it anyway.

### Will it hold?
The part nobody expects from a CAD program. Anvil rates the frame you just
modelled against **AISC 360-16** when it is steel and the **Aluminum Design
Manual 2020** when it is aluminium, and shows the arithmetic. Which book applies
is decided by the material on the body, not by the caller, and a steel clause
asked about an aluminium member is refused by name rather than answered.

Open **Load Path** on the rail. It sweeps every pair of parts and finds where
they actually touch — a faying face, a pin fit, a tangent line, a collision you
have not coped yet — and how long that seam is. The seam is *measured off the
solids every time*, so it is the weld you can really lay, not the one you meant
to lay. Move the tube 2 mm and the seam changes with it.

Then you tell it three things:

- **Weld it** on a contact, or point straight at the seam with the **Weld** tool
  (`Shift+W`). The leg comes in at the smallest one AISC Table J2.4 allows for
  the metal it just measured, and it says whether you are welding one side or
  both — two fillets are twice the weld and twice the capacity. A joint that is
  not square grows a **dihedral** box, already filled in with the angle measured
  off the two solids' faces rather than asked for; below 60° it also asks the
  welding **process** and **position**, because AWS Table 4.2's Z-loss spans five
  to one inside one band and there is no safe default. An **aluminium** joint
  grows a **filler** picker, because there is no default filler to fall back on —
  4043 and 5356 are not interchangeable.
- **Push here** — the load, in newtons, on the part it acts on. It reads back in
  pounds beside it.
- **Hold here** — what is bolted to the floor.

**Will it hold?** traces the load from where it goes in to where it goes to
ground, works out what arrives at every joint on the way including the lever
arm — usually most of the answer — and rates:

- **Fillet welds**, the group taken as a line, so an off-centre pull twists the
  weld instead of only shearing it. Base metal as well as filler. Steel joints
  go through AISC §J2.4, aluminium ones through ADM §J.2 with the heat-affected
  band beside them; a joint that is not square goes through AWS D1.1's four skew
  regimes and gets its real throat, which at 120° is 0.577 of the leg and not
  0.707.
- **Bolts** — shear, tension, the combined case, bearing and tear-out, with edge
  distance and spacing checked against the table.
- **Pins and clevises** — the four limit states on a pad eye, plus bending in
  the pin itself, which is usually what actually limits a clevis.
- **Members** — the tube *between* the joints. A mast can be welded perfectly
  top and bottom and still fold in the middle; this is what catches that.
  Buckling, bending, shear, torsion and tension, and the combinations, braced
  over the measured distance between the joints holding it. A 6061 member goes
  to the ADM instead — Chapters D and E, Chapter F for a tube or a rectangular
  bar, **Chapter G for shear, Section H.2 for torsion and Section H.3 for the
  two together**, and the ADM's own interaction equation, which is linear where
  AISC's is not. Where the weld sits on an aluminium member changes the answer: a
  transverse weld more than 0.05L in from a supported end drops the *whole*
  member from 241 MPa of yield to 105.

Every joint comes back with a ratio — under 1.0 holds — and prints the code
clause and the equation with your numbers substituted, so a welder, an inspector
or an engineer can re-run it on paper. It prints with the drawing.

**Three colours, and the third one is the one to learn.** Green passed, red
failed, and **amber means something on that load path was not checked**. Amber
is not a softer green. Anvil names what it skipped instead of quietly dropping
it, and when you see it the ratio describes the rest of the frame and not that.
The named gaps you are most likely to meet:

- **Torsion twisting an open section** — an angle, a tee or a channel. Closed
  tube is answered and so is a flat bar on edge; §H3.3 scopes the open case but
  gives its buckling stress as "as determined by analysis" and prints no formula.
  This is where it bites hardest, because an open section has almost none of a
  closed tube's torsional stiffness.
- **Block shear at a bolt group** (§J4.3). Anvil takes a count, a size and a
  grade — never a layout — and block shear needs a layout. It is lettered on the
  drawing's bolt notes as well as on the report, because a bolted end connection
  is routinely what §J4.3 governs and a sheet that says nothing about it reads
  like a connection that passed.
- **Aluminium shear and torsion on some SHAPES, not on the chapters.** ADM
  Chapter G, Section H.2 and Section H.3 are answered now for tubes, pipe and
  rod. What is still refused, by name: a solid **rectangular bar** in shear or
  torsion, because Chapter G's four member sections are flat webs on two edges,
  flat webs on one, pipes and round or oval tubes, and rods — and a bar is none
  of them; an **open shape** in torsion, because Section H.2.4 prints no `Tn`
  equation at all; a solid **rod** in H.3, which is neither H.3.1's flat element
  nor H.3.2's curved one; and a member with a **longitudinal weld**, because
  Eq. G.1-2 wants the weld-affected part of the SHEAR area and what a document
  states is the weld-affected part of the gross area.
- **An aluminium member in bending with a weld on it** (ADM Chapter F). The
  manual prints a welded local-buckling stress and a welded lateral-torsional
  one and no welded `Mnp` or `Mnu` at all, so there is no plastic-moment
  equation to write down.
- **A skewed joint that is also aluminium.** Each alone is answered. Together
  they would need a skewed-throat equation from AWS D1.2, the aluminium
  welding code — which is held, was read end to end, and **does not contain
  one**. That is a permanent refusal with a clause behind it, not a missing
  book (`STANDARDS-FINDINGS.md` §H).

It refuses outright rather than guessing, too. A section you sketched instead of
picking off the stock list can be checked for pull but not for bending, because
nothing told it what shape it is. A member in compression with no unbraced
length is refused rather than answered on yielding. And a member held at both
ends by pins has no effective-length factor to take — that is a mechanism, not a
column — so the check refuses instead of assuming K = 1.0, which on something
that is really a cantilever overstates the buckling load fourfold.

So two habits make the whole thing work: **give every part a material**, and
**pick sections off the stock list**. Those are what the check reads.

Sixteen of AISC's own published worked examples run as tests, and seven of the
Aluminum Design Manual's, so the numbers land where the books land.

**And the plain version:** this is a calculator that shows its work, not a
stamped drawing. It rates what you told it, on the frame you modelled, against
the code that covers the metal. It knows nothing about fatigue, shock loading, a
cold-lapped weld, or the mystery steel in the rack. If somebody is going to stand
under it, an engineer signs it.

### Getting it out
| Format | Use it for |
|---|---|
| **DXF** | Laser / waterjet / plasma. **Send this to SendCutSend.** True arcs and circles. |
| **STL, 3MF** | Your slicer. 3MF is better — it carries units, so nothing arrives at 1/25th scale. |
| **STEP, IGES** | A machinist, or another CAD package. Exact geometry, not a mesh. |
| OBJ, PLY, glTF, SVG | Rendering, viewing, documentation |

Import works too: STEP, IGES, STL, OBJ, and DXF/SVG (which come in as editable
sketch geometry you can extrude).

### Printing to paper
`Ctrl+P`. Two options:

**Quick Print** — the viewport exactly as it sits, with a title block. When you
just want a picture of it on paper.

**Drawing Sheet** — a real engineering drawing. Third-angle orthographic views
projected from the solid with hidden-line removal, so it is true vector line work
that stays sharp at any print resolution rather than a screenshot. Comes with:

- Front / Top / Right / Iso layouts, or whichever subset you pick
- Hidden edges dashed, silhouettes included (a turned part draws properly)
- Automatic scale snapped to a standard ratio — 1:2, 1:4, 1:8 — never 1:3.7194
- Title block: part name, material, mass in kg and lb, overall size, scale,
  units, date
- Sheet sizes Letter through A1, portrait or landscape
- Shop notes down the left edge

**And for a bent part it prints the bend schedule** — flat length, cut length,
and per bend the angle, rotation index, distance to start of bend, and the
springback-compensated die angle. Print it, tape it to the bender.

**For a welded frame it prints the weld schedule.** One row per joint: mark,
joint, leg, effective throat, total deposited length, extent, sides, filler,
the dihedral angle with its Z-loss, the process and position the Z-loss was read
against, and the **preheat** — the minimum preheat and interpass temperature AWS
D1.1 Table 5.11 asks for, off the thickest part at the joint. That last one is
not a number to check against; it is something to do before striking an arc, and
it prints for that reason. When the table asks for *none*, it prints that too,
because D1.1 saying no preheat is required is a different sheet from nobody
having asked. Everything the kernel worked out about the weld now reaches the
sheet the welder actually works from — it did not, for a long time, and a value
computed and not printed is from the shop floor a value nobody computed. Five
words carry the cells that are not numbers, because a blank is none of them:
**REFUSED** is a check that ran and would not answer, **NOT CHECKED** is a value
the document states that the check never saw, **SUPPRESSED** is a joint turned
off in the document, **UNSTATED** is the document not saying — which is not a
zero and not a default, and is what a leg, an extent, a side count or a filler
nobody gave comes out as — and an em dash is not applicable. The key prints
whenever one of them appears, and says none of them is a pass.

For a while it was four, and the fifth printed on the schedule anyway: the weld
key defined REFUSED, NOT CHECKED, SUPPRESSED and the em dash and left out
UNSTATED, so a sheet that used the word carried a key explaining three words it
had not used and omitting the one it had. What fixed it is worth more than the
fix. The key now takes its vocabulary off the schedule's own printed cells,
rather than from a second list somebody kept in step by hand, on all three
schedules — so a word cannot reach paper undefined.

**For a bolted frame it prints the bolt schedule.** Same idea, different
connection, and for a long time there was nothing: a bolted splice was rated in
full — shear, tension, the combined case, bearing and tear-out at the holes —
and reached no printed sheet at all. One row per bolted joint, marked B1, B2 so
it cannot be read against a W-marked weld: count, diameter, grade, whether the
threads were taken in or out of the shear plane, how many planes, the **clear**
edge distance (hole edge to part edge, which is what tears out), the spacing —
and **RATED TO**, the standard it was rated against, or one of the four words a
column that carries no measurement can take: REFUSED, NOT CHECKED, SUPPRESSED or
UNSTATED — never an em dash, because a rating is always applicable.
That last column has to be there: everything to the left of it is the drawing's
own instruction and stays true when the check refuses, so without it a refused
joint prints a row indistinguishable from a rated one. The caption says
*n OF m NOT RATED* before you read a row.

**And BOLT NOTES beside it, which exists for block shear.** §J4.3 is refused on
every bolt group here and is routinely what governs a bolted end connection, so
its note is lettered against every mark rather than left to the capacity report.
It is long, it is cut to fit, and the cut is announced — announcing it is the
whole point of the table.

**And for a pinned joint it prints the pin schedule** — the third connection
kind, and the last one that had nothing. Marked P1, P2, so a note can be read
against neither a W nor a B: the bore, the pin diameter, the lug thickness, the
**clear** edge distance, the lug's steel, the clevis ear thickness and the gap
across it — the last two being the whole of the pin's bending lever, and the
only two numbers in the row the model does not hold. Then RATED TO, the same
column for the same reason. Most of a pin row is a measurement rather than an
instruction, which is exactly why that last column has to be there: without it
an unrated pin prints a row of UNSTATED that reads as a joint nobody bothered to
dimension.

A stated gap of zero prints **0** and an unstated one prints **UNSTATED**,
because a clevis hard up against the lug really is zero and the two must not
look alike — the shorter lever is the one that makes the pin look stronger.

Printing goes through the normal browser dialog, so *Save as PDF* is right there.
There is also a **Save SVG** button if you want the vector file to drop into
another document.

---

## Working in it

### Navigation
| Action | Input |
|---|---|
| Orbit | Middle mouse drag |
| Pan | Shift + middle mouse drag |
| Zoom | Scroll wheel (zooms toward the cursor) |
| Fit everything | `Z` |
| Named views | `1`–`7` (front / back / left / right / top / bottom / iso) |
| Ortho ⇄ perspective | `0` |

Or click a face, edge or corner of the view cube in the top-right.

### Hotkeys
| | |
|---|---|
| **Sketch** | `L` line · `R` rectangle · `C` circle · `A` arc · `P` polygon · `X` trim · `T` extend · `O` offset · `F` fillet · `D` dimension · `N` look straight at the sketch · `K` constraints |
| **Model** | `S` new sketch · `E` extrude · `V` revolve · `W` sweep · `H` hole · `Shift+F` fillet edge · `Shift+C` chamfer · `Shift+S` shell · `B` union · `M` move · `Q` rotate · `G` scale |
| **Fabricate** | `Shift+B` tube bend · `Shift+U` unfold · `Shift+W` weld a seam |
| **File** | `Ctrl+S` save · `Ctrl+O` open · `Ctrl+E` export · `Ctrl+Z` undo · `Ctrl+P` print |
| **Find anything** | `Ctrl+Shift+P` command palette · `Ctrl+F` search the rail |
| **All shortcuts** | `F1` — and you can rebind any of them |
| **History tree** | `F2` rename the selected feature |

Slot, mirror, and subtract/intersect are on the rail and in the palette; they
have no key.

### Typing dimensions
Any numeric field takes what you'd actually write:

```
12          25.4mm       1.5"        3/8"
1-1/4"      2cm          1in + 4mm   od/2
sqrt(16)    max(3, 7)
```

Bare numbers are in whatever unit the document is set to. Name a value in the
Parameters table and every other field can reference it — change `od` once and
the whole model follows.

---

## There is no tutorial in this build

Anvil used to open a guided lesson on first launch. It has been removed and will
be rebuilt: it opened itself over whatever you were doing, on a timer, and it
was the only dialog in the program you could not close. Nothing pops up on
launch now.

Until it comes back, `F1` is the way in — every shortcut, and a rebind on any of
them.

---

## How it's put together

Two halves that talk over loopback HTTP:

```
  ┌─────────────────────────────┐        ┌──────────────────────────────┐
  │  Shell  (WebGL2 + JS)       │        │  Kernel  (Python + OCCT)     │
  │  ─────────────────────      │  HTTP  │  ──────────────────────      │
  │  viewport, sketcher,        │ <────> │  B-rep geometry, booleans,   │
  │  constraint solver, UI,     │        │  fillets, sweeps, tessellate,│
  │  tools                      │        │  exporters, mass properties  │
  └─────────────────────────────┘        └──────────────────────────────┘
       no geometry kernel                      no idea what a button is
```

The shell never computes solid geometry; the kernel never decides what anything
looks like. Everything crossing the wire is millimetres and radians.

```
AnvilCAD/
  Anvil CAD.vbs      launch (silent)
  START.bat          launch (with console, for troubleshooting)
  Create Desktop Shortcut.vbs
  requirements.txt   pinned kernel versions — what .venv gets built from
  CLAUDE.md          the one-page rule sheet for anyone (or anything) working here
  ROADMAP.md         where the build stands against the goal — read this first
  QUEUE.md           the dispatch board; newest dated STATUS block at the top
  DONE-WHEN.md       what is left, why each one is left, and the 23 rules
  PHASES.md          order, branch names, agent conventions
  STANDARDS-FINDINGS.md
                     values read off AWS D1.1:2025, AWS D1.2-97, the ADM 2020
                     and AISC 360, with table numbers and printed pages
  index.html         app shell; every module mounts into it
  css/app.css        design system
  assets/            the application icon, and the script that draws it
  js/
    app.js           boot, wiring, layout
    core/            math, mesh decode, document store, event bus
    gl/              WebGL2 renderer, camera, picking, shaders
    sketch/          entities, constraint solver, region finder, snapping
    io/              kernel client, file save/open/export
    ui/              icons, DOM helpers, dialogs, tool rail, inspector, tree,
                     status bar, view cube, hotkeys, print
    tools/           tool state machines — sketch, model, measure,
                     load path, weld
  server/
    launch.py        builds .venv on first run, then starts app.py
    app.py           HTTP server + routing
    kernel/          shapes, profiles, features, tessellate, exporters,
                     analysis, bend, drawing, statics, structure
                     profiles.py holds the stock size tables — the only copy
                     structure.py + statics.py are the capacity check
  docs/
    ARCHITECTURE.md  the contract every module is written against
    EDITION-360-22.md, RCSC-348-20.md, AUDIT-*.md
                     what the newer editions and the bolt spec change,
                     read chapter by chapter against the kernel
    sample-*.svg     drawing sheets, checked in so a change to the
                     projector is visible in a diff
  demo/              the demo-reel capture rig. Not part of the application —
                     delete it and Anvil is byte-for-byte unchanged
  test/
    tests.html       browser unit tests
    smoke.html       module load & renderer smoke check
    occ_smoke.py     kernel smoke test
    regress.py       bugs that have been fixed once and must not come back
    *_check.py       the capacity suites — welds, bolts, members, statics,
                     load path, structure, whole assembly, the aluminium
                     (ADM) path, tube bending, body ids, and the stylesheet's
                     contrast ratios
    run-*.mjs        the browser suites again, driven headless
```

### Running the tests

```bash
node test\run-all.mjs
```

One command. It discovers every runner in `test/` — Python, and node, some of
which drive a headless browser — says how many it found, starts a kernel for
the one that needs it, prints a count per runner, and exits with the number of
runners that failed. A runner that could not start is reported as NOT RUN
rather than scored as a pass, so a suite that quietly stops running cannot hide
behind a green total. The per-runner counts live in that output and nowhere in
this file, because every count written into prose here has gone stale within
days. A list of the runners' names goes stale the same way, the day one is
added, so there is none here either. A runner is any `.py` in `test\` whose
name does not start with `_`, and any `run-*.mjs` but `run-all.mjs` itself —
the rule `discover()` in `test\run-all.mjs` applies — and this lists the ones
git carries:

```bash
git ls-files "test/*.py" "test/run-*.mjs" ":(exclude)test/_*" ":(exclude)test/run-all.mjs"
```

Any Python runner can also be run on its own and needs nothing but the `.venv`
the first launch built:

```bash
.venv\Scripts\python.exe test\regress.py
```

`adm_check.py` is the aluminium one: it holds the kernel against the Aluminum
Design Manual 2020 rather than AISC 360, because Chapter J is written for steel
and an aluminium weld is a different book. That book has now been read — see
`STANDARDS-FINDINGS.md`, which carries every transcribed value with its table
number and printed page. The aluminium member chapters are pinned against
**seven** of the manual's own worked examples, in the manual's own kip and inch
units because a check in millimetres cannot tell a transcription error from a
unit conversion: Examples 12, 14, 18, 19 and 20 in `member_check.py` for
Chapters D, E and F, and Examples 4, 5 and 20 in `adm_check.py` for Chapter G,
Section H.2 and Section H.3.

A node runner runs on its own the same way. The ones that drive a real browser,
headless, **need `playwright-core` first** — they borrow it from the demo rig's
`node_modules`, which is not checked in, so on a fresh clone they die on the
import before running a single check; `git grep -l playwright-core -- test`
names them. Once, and then never again:

```bash
npm --prefix demo install
```

They drive the Chrome already on the machine rather than downloading one, so if
Anvil runs here, they will.

```bash
node test\run-browser-tests.mjs
```

`run-refresh-tripwires.mjs` is the one to run before and after a visual
refresh: it fails if a colour is hard-coded outside the token block, an icon
goes missing, or a rail button loses either half of the pair of ids it carries.
It no longer fails on a rail button being *renamed* — the tutorial's lesson
selectors were what a rename used to break, and they have gone. The demo reel
still clicks rail buttons by id and nothing scans it, so a rename now shows up
there and nowhere else.

`run-flange-layout.mjs` is the per-edge flange table. It has a page of its own
because it is the only suite that needs the real `css/app.css` — `tests.html`
loads no stylesheet, and linking one in would restyle every other suite's
report. It opens the real dialog at four window widths and sweeps the container
width to prove which breakpoint actually fires.

`run-region-tests.mjs` is the region picker — choosing which enclosed region a
sketch builds from. It has a page of its own, `test/region-tests.html`, because
it stands up a sketch session and arms and disarms the keyboard layer several
times over; sharing a page would make the order the suites run in part of what
is being tested.

One of them, `run-check-smoke.mjs`, drives the real application end to end —
draws steel, welds it, loads it, and reads the ratio back off the screen — so it
is the only test that needs a kernel of its own running:

```bash
.venv\Scripts\python.exe server\launch.py --no-browser --port 8730
node test\run-check-smoke.mjs
```

**It used to flake, and it does not any more.** One check — *"the ratio reaches
the screen"* — failed about four runs in fourteen, always paired with *"the seam
is on it too"*. It was never the async render it looked like. The app opened its
welcome dialog on a 350 ms timer, the capacity panel closed itself before running
the check, and for 38 ms there was no dialog on screen and then 53 ms with only
the welcome — so `document.querySelector('.modal')` read the welcome card instead
of the answer. A welcome dialog contains neither a verdict nor the seam length,
so both checks failed together, which is what identified it.

The fix is test-side: every `.modal` read is scoped to a card that names itself
in its head. It was proved by forcing the losing interleaving rather than by
re-running until green — **20 of 20 green with the welcome forced to land first,
against 3 of 3 red for the old file under the same forcing**, plus 31 and 18
consecutive green unforced. The welcome dialog has since been removed from the
app altogether, so the second dialog no longer exists; the scoping stays anyway,
because a selector that can match two dialogs is a defect whether or not a second
one exists today.

**A run of greens is still not a measurement.** If this or anything else fails
and then passes on a re-run, that is something to diagnose, not something the
re-run fixed. Nothing in `test/` is currently known to flake but the
intermittent death the board carries as F224.

If it reports a page-load 404, the kernel it is talking to is older than the
routing table in `server/app.py` — restart it rather than chasing the test.

Or start Anvil normally and open `http://127.0.0.1:<port>/test/tests.html` and
`/test/smoke.html` in the window, which is the same browser suite with a
readable page instead of a summary line.

---

## Notes

**Nothing leaves your machine.** The kernel binds to `127.0.0.1` only. There is
no telemetry, no account, no network access beyond the one-time kernel install.

**Your work is autosaved** every 45 seconds and when the window loses focus.
Projects save as `.anvil` (plain JSON — readable, diffable, and not a format you
can get locked out of).

**Before you send a part out:** check the material and thickness on the vendor's
side, and confirm they got the units right. Anvil writes `$INSUNITS` into the DXF
header, but not every shop's software reads it.
