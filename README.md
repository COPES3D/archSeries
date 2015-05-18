#archSeries
##Archaeological time series tools

###Contributors so far
* David Orton, BioArCh, Department of Archaeology, University of York
* James Morris, School of Forensic and Investigative Sciences, University of Central Lancashire

###Overview
This project aims to develop tools for constructing and comparing frequency time series from archaeological data. In particular, we're looking at ways of synthesising ecofactual data from multiple sites and contexts with varying dates and dating precision, using both aoristic and simulation-based approaches (inspired to a great extent by Crema 2011). More experimentally, we're also developing functions for calibrating ecofactual data against time series of research intensity, e.g. based on volumes of processed environmental samples or numbers of excavated contexts.

The files in this repo are designed for use on a pilot dataset of contexts, environmental samples, and zooarchaeological finds supplied by MOLA, one of London's largest archaeological contractors. The data themselves are not included here for obvious reasons).

This is an ongoing project, so this README is intended primarily as a place to update progress and discuss problems. Ultimately, we intend to turn the functions in **date_functions.R** into an R package.

###Main files
1. **date_functions.R** - source file containing the functions under development.
2. **London_analysis.R** - script demonstrating the use of some of these functions to assess variations in research intensity over the course of London's 2000-year existence.
3. **fresh_vs_marine.R** - script expoloratory analysis revisiting (for London) the 'Fish Event Horizon' (FEH) phenomenon, which saw a sudden shift towards marine fish consumption in medieval England at around AD1000 (Barrett et al. 2004).
4. **CPUE_code.R** - code for a forthcoming paper exploring the FEH in London **WORK IN PROGRESS**.
5. **London_prep.R** - script for cleaning and formatting the datasets as supplied by the archaeological contractor. Obviously this is specific to our dataset and unlikely to be of general use.

###The functions
The core functions so far are `aorist` and `date.simulate`, which each estimate a chronological distribution from a data table of entities with date ranges, but using two very different approaches.

###1. aorist
Calculates the aoristic sum (*sensu* Crema 2011) from given date ranges for a set of entities, for example representing discrete archaeological contexts or samples. Optionally, these can be weighted, e.g. representing the number of items within a context or the volume of a soil sample. The function uses the data.table package for speed, but is nonetheless quite computationally intensive.

**Arguments:**

* `data` is a data table with (at least) two numeric columns called Start and End.
* `start.date` is a single numeric value for the start of the time period to be analysed (defaults to 0).
* `end.date` is a single numric value for the end end of the time period to be analysed (defaults to 2000).
* `bin.width` is a single numeric value setting the resolution of the analysis, in years (defaults to 100).
* `weight` is a numeric vector giving a weight for each context/entity (defaults to a constant of 1).

**Returns:**
A two-column data table with the aoristic sum itself (numeric) and bin labels (character). The number of rows in this data table will be `(end.date-start.date)/bin.width`.

**Also outputs (->>):**
`breaks`: a numeric vector of breaks points.
`params`: a character value summarising the arguments, for use in naming output files.

##2. date.simulate
Applies a simulation approach to estimate chronological distribution from given date ranges for a set of entities, for example representing discrete archaeological contexts or samples (this is again based on the procedure described by Crema (2011) and has advantages over `aorist` as set out in that paper). Optionally, the entities can be weighted, e.g. representing the number of items within a context or the volume of a soil sample.

The function simulates a date (year) for each entity, assuming a uniform distribution within the limits of its date range, and places the results into chronological bins of a specified resultion. This process is repeated a specified number of times, allowing the number of points falling into each chronological bin to be estimated, with confidence intervals. Weights are applied **after** sampling, such that heavily weighted entities will tend to increase confidence intervals within their date ranges (this will typically be an appropriately conservative approach in an archaeological scenario).

In future this function may be expanded to permit distributions other than uniform, although it is more likely these will end up as separate functions.

**Arguments:**

* `data` is a data table with (at least) two numeric columns called Start and End. If 'data' also has a column called taxon, this can be used with the 'species' argument to select the rows to be included in the simulation.
* `species` is a character vector indicating which rows should be included in analysis (defaults to NULL). It will be ignored if no taxon column is provided in 'data'.
* `start.date` is a single numeric value for the start of the time period to be analysed (defaults to 0).
* `end.date` is a single numric value for the end end of the time period to be analysed (defaults to 2000).
* `bin.width` is a single numeric value setting the resolution of the analysis, in years (defaults to 100).
* `rep` is the number of times the simulation will be run (defaults to 100).
* `weight` is a numeric vector giving a weight for each context/entity (defaults to a constant of 1).

**Returns:**
A long-format data table giving the sum of weight for each bin in each repeat.

**Also outputs (->>):**
`breaks`: a numeric vector of breaks points.
`params`: a character value summarising the arguments, for use in naming output files.

##3. dummy.simulate
Simulates a 'dummy' chronological distribution within specified date limits by sampling from within a distribution defined by an input vector. Designed for use with date.simulate, particularly within wrapper functions like freq.simulate. The idea here is to simulate chronological distributions based on the same number of entities (with the same weights) as a date.simulate call, but unconstrained by the known date ranges.

**Arguments:**

* `probs` is a vector of relative probabilities from which to sample. The length of this vector should match the desired number of bins in the output. For a uniform dummy set, pass a uniform numeric vector whose length matches the desired number of bins - e.g. `rep(1, 100)` where 100 bins are required. Alternatively, use this to specify a custom prior distribution, e.g. the output of an `aorist` call or the median values for each bin from a `date.simulate` call.
*[currently designed for a data.table (the output of an aorist call) consisting of
a numeric column ('aoristic.sum') to be used as relative probabilities, and a character column ('labels') containing bin labels - this needs to be worked on (see point 4 below)]*
* `weight` is a numeric vector (or data frame/data.table) representing (weighted) instances to be simulated. If given an additional character column called taxon, this can be used to select rows for analysis using the 'species' argument.
* `species` is a character vector indicating which rows should be included in analysis (defaults to NULL). It will be ignored if no taxon column is provided in 'data'.
* `start.date` is a single numeric value for the start of the time period to be analysed (defaults to 0).
* `end.date` is a single numric value for the end end of the time period to be analysed (defaults to 2000).
* `rep` is the number of times the simulation will be run (defaults to 100).

**Returns:**
A long-format data table giving the sum of weight for each bin in each repeat.

##4. freq.simulate
Performs both 'real' and dummy simulation (using `date.simulate` and `dummy simulate` respectively) on a set of entities with date ranges and optionally weights so that the two can be compared to detect deviation from a null hypothesis. The dummy set is generated using the same number of entities and the same weights as for the 'real' set. Both the full simulation results and a summary dataset are returned, and optionally also saved as .csv files.

Optionally, the function can also calculate the rate of change (ROC) between each bin and the next in each simulation (for both real and dummy sets). This allows one to test hypotheses concerning increases or decreases at certain points in the time series.

**Arguments:**

* `data` is a data.frame or data.table with columns including Start, End, and Frag. If additional factor columns are provided, these can be used to filter the data using the `filter.field` and `filter.values` arguments, below.
* `probs` is an optional vector of relative probabilities to be passed to dummy.simulate. Defaults to 1, resulting in a uniform dummy set. 
* `filter.field` is a character vector of length one, denoting the name of a column that will be used to filter the data. Defaults to 'Species' but ignored unless 'filter.values' is set.
* `filter.values` is a character vector indicating which rows should be included in analysis (defaults to NULL, in which case all values are included).
* `quant.list` is a numeric vector of quantiles to be included in the summary output.
* `ROC` is a logical value indicating whether rates-of-change should be calculated and appended to the output (defaults to FALSE). Nb. setting to TRUE will make the function much slower.
* `start.date` is a single numeric value for the start of the time period to be analysed (defaults to 0).
* `end.date` is a single numeric value for the end end of the time period to be analysed (defaults to 2000).
* `rep` is the number of times the simulation will be run (defaults to 100).

**Returns:**
A list of length two:

1. A long-format data table giving the sum of weight for each bin in each repeat, for both the 'real' and 'dummy' simulations.
2. A data table giving the specified quantiles of the full simulation results for each bin, again for both the 'real' and 'dummy' simulations.

**Also outputs:**
.csv files for both full and summary results
*[to be made optional]*

##Required packages

* data.table
* reshape2

##References

* Barrett, J.H., A.M. Locker & C.M. Roberts (2004) The origins of intensive marine fishing in medieval Europe: the English evidence. *Proceedings of the Royal Society of London* B, **271**, 2417-2421.
* Crema, E. (2012) Modelling temporal uncertainty in archaeological analysis. *Journal of Archaeological Method and Theory*, **19**, 440-461. 

##Current issues to work on

1. Make aorist progress reporter optional.
2. Add progress reporters for other functions?
3. date.simulate: add option to simulate by item rather than by context.
4. Revise the filtering mechanism in date.simulate and dummy.simulate to use less biology-specific terms and to allow filtering for multiple values.
5. Clear up bugs with passing a single vector to dummy.simulate - perhaps by separating the `probs` argument into two arguments, one for the probabilities and one for the labels. Does the vector of relative probabilities actually have to be the same length as the number of bins for the output?
6. New function(s) to generate model distributions to feed into dummy.simulate?
7. Set default for probs in freq.simulate, so that it's uniform unless specified otherwise.
8. Output files/variables from freq.simulate - set arguments to specify.
9. Fix problem with zero values in probs for dummy.simulate.
10. Change 'Frag' to 'count' in freq.simulate.
11. Make plotting functions.
12. Make aorist more intelligible!
