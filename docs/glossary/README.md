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
: a [task](#task) parameter, whose value can be set without editing the code of the task.
Implemented as a task member of type `Configurable`.
Can be set via [JSON](#json) configuration.

device (DPL device) <a id="device" />
: execution sub-unit of a [workflow](#workflow).
Implemented as a [task](#task).
The device name is generated from the task name, unless it is provided explicitly, using `TaskName("device-name")` as an argument of `adaptAnalysisTask`.

workflow (DPL workflow) <a id="workflow" />
: execution unit in [DPL](#dpl), consisting of one or several [devices](#device), implemented as a C++ `.cxx` file, compiled into a single executable binary file

workflow topology <a id="workflow-topology" />
: the connection between running [workflows](#workflow), based on their [inputs](#table) and outputs

## Table content

table <a id="table" />
: data format, storing a collection of columns for each entry (table row).
Rows represent objects of the given table type and columns represent properties of these objects.
A table definition defines a C++ type and therefore must be unique.

static column <a id="static-column" />
: [table](#table) column, which stores a value provided when it is filled.

dynamic column <a id="dynamic-column" />
: [table](#table) column, which behaves as a function of other (static or expression) column of the same table.
Its value is calculated only when the [column getter](#column-getter) is called.

expression column <a id="expression-column" />
: [table](#table) column, which stores a value which is calculated by evaluating an expression when the table is written

index column <a id="index-column" />
: [table](#table) column, which stores the index of a row in a table

column getter <a id="column-getter" />
: method that return the value stored in the column

data model <a id="table" />
: collection of [table](#table) definitions.
Usually defined in header files in [`DataModel`](../gettingstarted/theo2physicsrepo.md#folder-structure) directories.

## Table operations

grouping <a id="grouping" />
: grouping

filtering <a id="filtering" />
: filtering

paritioning <a id="paritioning" />
: paritioning

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
