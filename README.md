# assay.R

A tool in R for targeted mass-spec assays.

## AssayR

AssayR provides a small number of functions for simple and
rapid assay of high resolution LCMS data, especially isotopically
labelled tracer experiments.

Given a table of desired compounds, in a specific format, AssayR will extract
chromatograms for those compounds, and interactively allow you to optimise peak
detection.  Peak detection is performed on the maximal trace for the whole data
set, including any isotopes you've defined, and so the approach is immune to 
missing peaks.

The output is a table of results (peak areas).  With a labelling experiment, 
there's also graphical output in the form of stacked bars to summarise the 
information (both percentage and aboslute).

To install AssayR (first install R and optionally RStudio) run the following
commands **in R-3.2.2** or later:

    install.packages("devtools")
    devtools::install_github("davidsanin/assayR", dependencies=TRUE)

If you have
not yet installed R, or have an older version, 
please visit [CRAN](https://cran.r-project.org/).  And I 
heartily recommend
[RStudio Desktop](https://www.rstudio.com/products/RStudio/#Desktop).

Please note that this package is still in development and there are known bugs (see below.)  If you feel you've uncovered another bug, please don't hestitate to report it.

## Summary of use

```{r usage, eval=FALSE}

# convert raw files to mzML if they are not already
# msconvert_all() canhelp with this
# this extract pos and neg and puts them in mzMLpos and mzMLneg
# directories

# to use the example files, load AssayRdata and grab the file paths...

library(AssayR)
library(AssayRdata)

path.to.config <- system.file(
    "extdata/configs/examples-maxC6.tsv", 
    package = "AssayR")
path.to.mzML <- system.file(
    "extdata/", 
    package = "AssayRdata")

# create a directory where our data and graphs will be output:
dir.create("~/AssayR-example")
setwd("~/AssayR-example/")

# generate TICs/EICs/XICs from config file
run.config.tics(path.to.config, path.to.mzML, desiredOutputDirectory)
# more about the config file below

# analyse the peaks (interactively)
results <- run.config.peaks(path.to.config, path.to.tics)
# this is interactive, more below.
# you could write the result to csv
# write.csv(results, "results.csv")

# rearrange and output the results, and output plots of the
# isotopes:
assay.plotter(results)
# check the current working directory for outputs

```

## Parallel processing

Samples can be run in parallel using the BiocParallel package:

```
BPPARAM = MulticoreParam(workers=20)
run.config.tics(path.to.config, path.to.mzML, desiredOutputDirectory, BPPARAM=BPPARAM)
```

## Config file

There is an example config file with this package.  To access it use a command
like this:

    system.file("extdata/configs/examples-maxC6.tsv", package = "AssayR")

The format should be a tsv, as readable by `read.delim`, and the file
should have the following columns:

* target
    name of the target channel (gets used for naming data)
* name
    name of the compound (gets used for labelling graphs)
* mz
    monoisotopic m/z value for this compound
* ppm
    m/z tolerance in ppm for this compound
* rt.min
    minimum retention time to be considered (can be modified interactively)
* rt.max
    maximum retention time to be considered (can be modified interactively)
* seconds
    width of pick-detection filter in seconds (can be modified interactively)
* threshold
    minimum required intensity (can be modified interactively)
* known alternatives
    reserved for future use
* C13
    in labelled experiment, maximum possible number of 13C in compound
* N15
    in labelled experiment, maximum possible number of 15N in compound
* H2
    in labelled experiment, maximum possible number of 2H in compound
* interactive
    values: "yes" or "done", indicates whether interactive mode is 
    required for this definition
* comment
    free text.  I use this for sorting my compounds in pathway order.

## Known bugs

* Very asymettric peaks can cause a peak start but no peak end to be detected. This causes script to crash.  Some error handler needed for this case.
* The above case can happen due to the workaround used to fix a previous bug which caused a crash when the Mexican hat filter was too wide for the region undergoing detection.  The crash still happens, but for a different reason now!

## more information

If you think something's missing, please get in contact and I'll update this
document so everybody can benefit.  Thanks!
