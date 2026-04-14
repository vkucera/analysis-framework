---
sort: 1
title: Dependency finder
---

# Dependency finder

Dependency finder is a tool to explore input/output dependencies between O2Physics workflows and tables.

## Usage overview

The [`find_dependencies.py`](https://github.com/AliceO2Group/O2Physics/blob/master/Scripts/find_dependencies.py) script is available in the O2Physics repository and must be executed inside the O2Physics environment:

```bash
$O2PHYSICS_ROOT/share/scripts/find_dependencies.py <options>
```

The full list of options is displayed when the option `-h` is provided:

```text
usage: find_dependencies.py [-h] [-t TABLE [TABLE ...]] [-w WORKFLOW [WORKFLOW ...]] [-T TABLE_REV [TABLE_REV ...]] [-W WORKFLOW_REV [WORKFLOW_REV ...]] [-c] [-g {pdf,svg,png}]
                            [-x EXCLUDE [EXCLUDE ...]] [-l LEVELS]

Find dependencies required to produce a given table or to run a given workflow.

optional arguments:
  -h, --help            show this help message and exit
  -t TABLE [TABLE ...]  table(s) for normal (backward) search (i.e. find producers)
  -w WORKFLOW [WORKFLOW ...]
                        workflow(s) for normal (backward) search (i.e. find inputs)
  -T TABLE_REV [TABLE_REV ...]
                        table(s) for reverse (forward) search (i.e. find consumers)
  -W WORKFLOW_REV [WORKFLOW_REV ...]
                        workflow(s) for reverse (forward) search (i.e. find outputs)
  -c                    be case-sensitive with table names
  -g {pdf,svg,png}      make a topology graph in a given format
  -x EXCLUDE [EXCLUDE ...]
                        name patterns of tables and workflows to exclude
  -l LEVELS             maximum number of workflow tree levels (default = 0, include all if < 0)
```

## Modes

Modes define the direction of search through the dependency tree.

Supported options are `-t`, `-w`, `-T`, `-W`.
Options can be used together and each option takes an arbitrary number of arguments.

### Backward mode

The backward mode searches for **parents** of the given object in the dependency tree.

#### Table producers (`-t`)

Gives a list of workflows that produce a given table.

If the origin prefix is not provided, it is assumed to be `AOD`.

Example: `-t BC_001`

```text
Table: AOD/BC_001

AOD/BC_001 <- ['o2-analysis-bc-converter']
```

Examples of use:

- Find a helper task (e.g. a converter) that produces a missing input table.
- Find a producer workflow to inspect how a table is filled.

#### Workflow inputs (subscriptions) (`-w`)

Gives a list of tables consumed by a given workflow.

Example: `-w o2-analysis-bc-converter`

```text
Workflow: o2-analysis-bc-converter

o2-analysis-bc-converter <- ['AOD/BC']
```

Examples of use:

- Find dependencies of a Hyperloop wagon (i.e. the list of directly required workflows) (with the `-l 1` option).
- Resolve unclear table name aliases in subscriptions.

### Forward mode

The forward mode searches for **children** of the given object in the dependency tree.

#### Table consumers (`-T`)

Gives a list of workflows that consume a given table.

If the origin prefix is not provided, it is assumed to be `AOD`.

Example: `-T BC`

```text
Table: AOD/BC

AOD/BC -> ['o2-analysis-bc-converter']
```

Examples of use:

- Find workflows affected by the modification of a table.
- Find a workflow which consumes a table that is not supposed to be required.

#### Workflow outputs (`-W`)

Gives a list of tables produced by a given workflow.

Example: `-W o2-analysis-bc-converter`

```text
Workflow: o2-analysis-bc-converter

o2-analysis-bc-converter -> ['AOD/BC_001']
```

Examples of use:

- List all output tables of a workflow that contains multiple structs.
- See the table descriptions of tables whose description names are different from their type names.
- See the table descriptions of tables table whose description names are generated in macros.

## Dependency levels (`-l`)

If provided with an integer number, dependencies are searched recursively up to the provided number of workflow levels.
The levels are indicated with indentation in the output.

If the provided number is negative, all levels are considered.

If not provided, only direct dependencies (`-l 0`) are considered.

Example: `-w o2-analysis-timestamp -l 1`

```text
Workflow: o2-analysis-timestamp

o2-analysis-timestamp <- ['AOD/BC_001']
  AOD/BC_001 <- ['o2-analysis-bc-converter']
    o2-analysis-bc-converter <- ['AOD/BC']
```

## Exclude (`-x`)

Workflows and tables can be excluded from the search using the `-x` option which supports regular expressions.

Whitelisting can be achieved using the negative lookahead `(?!...)`.

Example: `-T TRACKSELECTION -x 'o2-analysis(?!-em-)'` finds workflows that consume the `TRACKSELECTION` table and their name contains `o2-analysis-em-`.

```text
Table: AOD/TRACKSELECTION

AOD/TRACKSELECTION -> ['o2-analysis-em-omega-meson-emc', 'o2-analysis-em-efficiency-ee', 'o2-analysis-em-mc-templates', 'o2-analysis-em-heavy-neutral-meson']
```

## Graphical output (`-g`)

The dependency tree can be visualised in a graph. (Requires [Graphviz](https://graphviz.org/) installed.)

Example: `-t TRACKSELECTION -l 1 -x converter onthefly alice3 run2 service -g png`

```text
Table: AOD/TRACKSELECTION

AOD/TRACKSELECTION <- ['o2-analysis-trackselection']

Workflow dependency tree:

o2-analysis-trackselection <- ['AOD/TRACK', 'AOD/TRACKDCA', 'AOD/TRACKEXTRA_002']
  AOD/TRACK <- ['o2-analysis-track-propagation']
    o2-analysis-track-propagation <- ['AOD/BC_001', 'AOD/COLLISION_001', 'AOD/MCPARTICLE_001', 'AOD/MCTRACKLABEL', 'AOD/TIMESTAMPS', 'AOD/TRACKCOV_IU', 'AOD/TRACKEXTRA_002', 'AOD/TRACK_IU']
  AOD/TRACKDCA <- ['o2-analysis-trackextension', 'o2-analysis-track-propagation']
    o2-analysis-trackextension <- ['AOD/BC_001', 'AOD/COLLISION_001', 'AOD/TIMESTAMPS', 'AOD/TRACK', 'AOD/TRACKEXTRA_002']

Making dot file in: TRACKSELECTION.gv
Making graph in: TRACKSELECTION.png
```

The output graph `TRACKSELECTION.png` is shown below:

<div align="center">
<img src="TRACKSELECTION.png" width="1000px" alt="TRACKSELECTION">
</div>
