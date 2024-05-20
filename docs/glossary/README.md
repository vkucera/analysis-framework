---
sort: 10
title: Glossary
---

# Glossary

## General

O<sup>2</sup> <a id="o2" />
: online-offline

DPL <a id="dpl" />
: data processing layer

Arrow <a id="arrow" />
: format in which [tables](#table) are processed in [O<sup>2</sup>](#o2)

JSON <a id="json" />
: format used for configuration of [devices](#device)

AO2D <a id="ao2d" />
: format of saving [tables](#table) in ROOT files

## Workflows

task <a id="task" />
: C++ `struct` which can process and produce [tables](#table) and produce other output (e.g. histograms).
A task is executed as a [device](#device) of a [workflow](#workflow).

configurable <a id="configurable" />
: [task](#task) parameter, whose value can be set without editing the code of the task.
Implemented as a task member of type `Configurable`.
Can be set via [JSON](#json) configuration.

process function <a id="process-function" />
: [task](#task) method, which subscribes to [tables](#table) (i.e. consumes them as input) and performs some operations with their content (e.g. fills output tables or histograms)

process function switch <a id="process-function-switch" />
: [task](#task) parameter, which allows to enable and disable execution of a given [process function](#process-function).
Can be set via [JSON](#json) configuration.

device (DPL device) <a id="device" />
: execution sub-unit of a [workflow](#workflow).
Implemented as a [task](#task).
The device name is generated from the task name, unless it is provided explicitly, using `TaskName("device-name")` as an argument of `adaptAnalysisTask`.

workflow (DPL workflow) <a id="workflow" />
: execution unit in [DPL](#dpl), consisting of one or several [devices](#device).
Implemented as a C++ `.cxx` file, compiled into a single executable binary file.
The workflow name is generated from the arguments of the `o2physics_add_dpl_workflow` function in the `CMakeLists.txt` file.
For workflows files located `PWG..` directories, a corresponding prefix is added: `o2-<component>-[<pwg>-]<name>`
: Example: `o2-analysis-hf-task-d0`

  - `<component>` is `analysis`, derived from `COMPONENT_NAME Analysis`,
  - `<pwg>` is `hf`, derived from the `PWGHF` directory name,
  - `<name>` is `task-d0`, provided in `o2physics_add_dpl_workflow(task-d0`.

workflow topology <a id="workflow-topology" />
: the connection between running [workflows](#workflow), based on their [inputs](#table) and outputs

## Table content

table <a id="table" />
: data format, storing a collection of columns for each entry (table row).
Rows represent objects of the given table type and columns represent properties of these objects.
A table definition defines a C++ type and therefore must be unique.

static column <a id="static-column" />
: [table](#table) column, which stores a value provided when it is filled.
: Example: The `Collisions` table of collisions has a static column `PosZ` which stores the value of the _z_ coordinate of the primary vertex.
The value has to be provided when a new collision row is being added in the table.

dynamic column <a id="dynamic-column" />
: [table](#table) column, which behaves as a function of other (static or expression) columns of the same table.
Its value is calculated only when the [column getter](#column-getter) is called.
Dynamic column definitions can have free parameters, which have to be provided as arguments of the getter.
Values of dynamic columns are not written in [AO2D](#ao2d) files and cannot be used for table [filtering](#filtering).
: Example: The `Tracks` table of tracks has a dynamic column `Px`, representing the _x_ component of the track momentum, defined as a function of other static columns.
It's value is calculated every time the corresponding `px` getter is called.

expression column <a id="expression-column" />
: [table](#table) column, which stores a value which is calculated by evaluating an expression when the table is written.
Values of expression columns are written in [AO2D](#ao2d) files and can be used for table [filtering](#filtering).
: Example: The `Tracks` table of tracks has an expression column `Phi`, representing the azimuth of the track momentum, defined as a function of other static columns.
All values of the `Phi` column are calculated for all tracks together (in bulk) when the `Tracks` table is written in the [AO2D](#ao2d) file.

index column <a id="index-column" />
: [table](#table) column, which stores the index of a table row
: Example: The `Tracks` table of tracks has an index column pointing to rows of the `Collisions` table, which links each track to its collision.
Calling the `collision` getter returns an iterator pointing to a given collision.
Calling the `collisionId` getter returns the value of the index itself (i.e. the position of the row in the `Collisions` table).

column getter <a id="column-getter" />
: method that returns the value stored in the column
: Example: The value of the transverse momentum of a given `track` in the `Tracks` table is stored in the `Pt` column.
This value can be obtained by calling the corresponding `pt` getter as `track.pt()`.

data model <a id="table" />
: collection of [table](#table) definitions.
Usually defined in header files in [`DataModel`](../gettingstarted/theo2physicsrepo.md#folder-structure) directories.

## Table operations

grouping <a id="grouping" />
: grouping

filtering <a id="filtering" />
: filtering

partitioning <a id="partitioning" />
: partitioning

slicing <a id="slicing" />
: slicing

spawning <a id="spawning" />
: spawning

## AliHyperloop

wagon <a id="wagon" />
: instance of a [workflow](#workflow) on AliHyperloop, defined by its dependencies, settings of devices and outputs

train <a id="train" />
: collection of wagons executed on AliHyperloop

derived data <a id="derived-data" />
: [AO2D](#ao2d) file produced as a result of processing another AO2D file (parent)

linked derived data <a id="linked-derived-data" />
: [derived data](#derived-data) containing tables with [index columns](#index-column) pointing to tables in its parent files.
Processing these columns requires access to the parent files.

self-contained derived data <a id="self-contained-derived-data" />
: [derived data](#derived-data) containing tables without [index columns](#index-column) pointing to tables in other files.
