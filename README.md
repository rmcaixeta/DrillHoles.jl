# DrillHoles

[![Build Status][build-img]][build-url] [![Coverage][codecov-img]][codecov-url]

`DrillHoles.jl` is a package written in Julia for drill holes desurvey and compositing.

**This fork is based in the version `0.1.4` of the [`JuliaEarth/DrillHoles.jl`](https://github.com/JuliaEarth/DrillHoles.jl), before some major refactoring. This version is currently (Apr 24) preferred for the following situations:**

- **The coordinates after desurvey matches commercial softwares**
- **More options of compositing**
- **Warnings available for some issues during desurvey**
- **Big datasets will run faster in this version**

**Known bugs on the other hand (corrected on the official version):**

- **Merging multiple tables may generate some issues in categorical variables in some specific conditions**

## Installation

First, it is necessary to install Julia. Installation instructions for Windows, Linux and macOS are available [here](https://julialang.org/downloads/platform/).

To install `DrillHoles.jl` package: open a terminal, type `julia` to open the REPL and then install the package with the following command. Additionally, the `GeoStats.jl` package is also installed to run the example later.

```julia
using Pkg; Pkg.add("DrillHoles"); Pkg.add("GeoStats")
```

## Drill hole tables

Before using the package, it is necessary to have a collar table, a survey table and at least one interval table (such as assay and lithology). They can be passed as CSV file or `DataFrame`. Examples of data tables are shown below:

<p align="center">
  <img src="imgs/tables_example.png">
</p>

## Usage example

```julia
using DrillHoles

# Inform drill hole tables and main columns
collar = Collar(file="C:/collar.csv", holeid=:HOLEID, x=:X, y=:Y, z=:Z)
survey = Survey(file="C:/survey.csv", holeid=:HOLEID, at=:AT, azm=:AZM, dip=:DIP)
assay  = Interval(file="C:/assay.csv", holeid=:HOLEID, from=:FROM, to=:TO)
litho  = Interval(file="C:/litho.csv", holeid=:HOLEID, from=:FROM, to=:TO)

# Desurvey drill hole tables
dh = drillhole(collar, survey, [assay, litho])

# The DrillHole object created have 4 components
dh.table  # the desurveyed drill hole table
dh.trace  # the trace file with coordinates at surveyed depths
dh.pars   # the main column names
dh.warns  # the table of warning and errors identified during desurvey

# Composite drillhole using :equalcomp mode, which seeks to create composites
# with the exact `interval` length;  borders are discarded if have length below
# `mincomp`. Max composite length = `interval`
comps = composite(dh, interval=1.0, zone=:LITHO, mode=:equalcomp)

# Composite drillhole using :equalcomp mode; composite lengths are defined
# seeking to include all possible intervals with length above `mincomp`.
# Max composite length = 1.5*`interval`
comps = composite(dh, interval=1.0, zone=:LITHO, mode=:nodiscard)

# To use the drill hole file as PointSet into GeoStats.jl framework
using GeoStats
pointset = georef(comps.table, (:X,:Y,:Z))
```


[build-img]: https://img.shields.io/github/actions/workflow/status/rmcaixeta/DrillHoles.jl/CI.yml?branch=master
[build-url]: https://github.com/rmcaixeta/DrillHoles.jl/actions

[codecov-img]: https://codecov.io/gh/rmcaixeta/DrillHoles.jl/branch/master/graph/badge.svg
[codecov-url]: https://codecov.io/gh/rmcaixeta/DrillHoles.jl
