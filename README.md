
<!-- README.md is generated from README.Rmd. Please edit that file -->
The MonteCarlo Package
======================

The MonteCarlo package helps you quickly and easily create simulation studies and summarize their results in LaTeX tables. LaTeX is a tool that turns plain text into professional-looking documents. The following example demonstrates how to use the MonteCarlo package and briefly discusses more advanced features. For more information about the MonteCarlo package, see the [package vignettes](https://github.com/FunWithR/MonteCarlo/tree/master/vignettes).

The MonteCarlo package contains the following main functions:

| Function      | Description |
| ----------- | ----------- |
| `MonteCarlo()`| Runs a simulation study for a user-defined parameter grid. This function handles the generation of loops over these parameter grids and parallelizes the computation on a user-specified number of CPU units. |
| `MakeTable(` | Creates LaTeX tables from the output of `MonteCarlo()`. This function stacks high-dimensional output arrays into tables with a user-specified ordering of rows and columns.|

To run a simulation study, you need to create a function that performs the following nested tasks:

* Generates a sample
* Calculates the desired statistics from the sample

The created function is then passed to the `MonteCarlo()` function. No additional programming is required. This approach gives you full control and flexibility in designing your MonteCarlo experiment. It also makes the `MonteCarlo()` function versatile. Finally, it's intuitive because you can formulate your experiment as if you're only interested in a single result.

Example
---------------

Suppose you want to evaluate the performance of a standard t-test for the hypothesis that the mean is equal to zero. You want to see how the size and power of the test change with the sample size (*n*), the distance from the null hypothesis (*loc* for location) and the standard deviation of the distribution (*scale*). The sample is generated from a normal distribution.

To conduct this analysis, begin by loading the MonteCarlo package with the following command:

``` r
library(MonteCarlo)
```

Then define the following function:

``` r
#########################################
##      Example: t-test

# Define a function that generates data and applies the method of interest.

ttest<-function(n,loc,scale){
  
  # generate sample:
    sample<-rnorm(n, loc, scale)
  
  # calculate test statistic:
    stat<-sqrt(n)*mean(sample)/sd(sample)
  
  # get test decision:
    decision<-abs(stat)>1.96
  
  # return result:
    return(list("decision"=decision))
}
```

As discussed in the previous section, the *ttest()* function is formulated to generate a single test decision. The function arguments are the parameters you're interested in. The *ttest()* function perform the following steps:

1.  Draws a sample of *n* observations from a normal distribution with mean *loc* and standard deviation *scale*.
2.  Calculates the t-statistic.
3.  Determines the test decision.
4.  Returns the desired result in a list.

You then define the combinations of parameters you're interested in and collect them in a list. The elements of the list must have the same names as the parameters for which you want to supply grids.

``` r
# Define a parameter grid:

  n_grid<-c(50,100,250,500)
  loc_grid<-seq(0,1,0.2)
  scale_grid<-c(1,2)

# Collect parameter grids in a list:
  param_list=list("n"=n_grid, "loc"=loc_grid, "scale"=scale_grid)
```

To run the simulation, pass the function *ttest()* and the parameter grid (*param\_list*) to the `MonteCarlo()` function with the desired number of MonteCarlo repetitions (*nrep=1000*):

``` r
# Run the simulation:

  MC_result<-MonteCarlo(func=ttest, nrep=1000, param_list=param_list)
```

No further coding is required. The `MonteCarlo()` function handles the mechanics of the MonteCarlo experiment.

Call the *summary()* function to get a short description of the simulation:

``` r
  summary(MC_result)
```

The function produces the following summary:

    ## Simulation of function: 
    ## 
    ## function(n,loc,scale){
    ##   
    ##   # generate sample:
    ##     sample<-rnorm(n, loc, scale)
    ##   
    ##   # calculate test statistic:
    ##     stat<-sqrt(n)*mean(sample)/sd(sample)
    ##   
    ##   # get test decision:
    ##     decision<-abs(stat)>1.96
    ##   
    ##   # return result:
    ##     return(list("decision"=decision))
    ## }
    ## 
    ## Required time: 12.33 secs for nrep = 1000  repetitions on 1 CPU
    ## 
    ## Parameter grid: 
    ## 
    ##      n : 50 100 250 500 
    ##    loc : 0 0.2 0.4 0.6 0.8 1 
    ##  scale : 1 2 
    ## 
    ##  
    ## 1 output arrays of dimensions: 4 6 2 1000

The summary indicates the simulation results are stored in an array with dimensions of `c(4,6,2,1000)`, where the MonteCarlo repetitions are collected in the last dimension of the array.

To summarize the results in a reasonable way and include them as a table in a paper or report, you have to represent them in a matrix. The `MakeTable()` function stacks the submatrices collected in the array into the rows and columns of a matrix and prints the result in the form of code to generate a LaTeX table.

To determine the order of the rows and columns in the LaTeX table, supply the function arguments *rows* and *cols*. These arguments are the vectors of the parameter names in the order you want them to appear in the table. In this example, they're sorted from the inside to the outside.

``` r
# Genrate a table:

  MakeTable(output=MC_result, rows="n", cols=c("loc","scale"), digits=2, include_meta=FALSE)
```

    ## \begin{table}[h]
    ## \centering
    ## \resizebox{ 1 \textwidth}{!}{%
    ## \begin{tabular}{ rrrrrrrrrrrrrrr }
    ## \hline\hline\\\\
    ##  scale && \multicolumn{ 6 }{c}{ 1 } &  & \multicolumn{ 6 }{c}{ 2 } \\ 
    ## n/loc &  & 0 & 0.2 & 0.4 & 0.6 & 0.8 & 1 &  & 0 & 0.2 & 0.4 & 0.6 & 0.8 & 1 \\ 
    ##  &  &  &  &  &  &  &  &  &  &  &  &  &  &  \\ 
    ## 50 &  & 0.06 & 0.30 & 0.83 & 0.98 & 1.00 & 1.00 &  & 0.05 & 0.11 & 0.28 & 0.55 & 0.79 & 0.95 \\ 
    ## 100 &  & 0.05 & 0.51 & 0.98 & 1.00 & 1.00 & 1.00 &  & 0.05 & 0.16 & 0.54 & 0.84 & 0.97 & 1.00 \\ 
    ## 250 &  & 0.06 & 0.89 & 1.00 & 1.00 & 1.00 & 1.00 &  & 0.04 & 0.34 & 0.91 & 1.00 & 1.00 & 1.00 \\ 
    ## 500 &  & 0.06 & 1.00 & 1.00 & 1.00 & 1.00 & 1.00 &  & 0.04 & 0.58 & 0.99 & 1.00 & 1.00 & 1.00 \\ 
    ## \\
    ## \\
    ## \hline\hline
    ## \end{tabular}%
    ## }
    ## \caption{ decision  }
    ## \end{table}

To change the order of the rows and columns, modify the *rows* and *cols* arguments:

``` r
# Generate a table:

  MakeTable(output=MC_result, rows=c("n","scale"), cols="loc", digits=2, include_meta=FALSE)
```

    ## \begin{table}[h]
    ## \centering
    ## \resizebox{ 1 \textwidth}{!}{%
    ## \begin{tabular}{ rrrrrrrrr }
    ## \hline\hline\\\\
    ## scale & n/loc &  & 0 & 0.2 & 0.4 & 0.6 & 0.8 & 1 \\ 
    ##  &  &  &  &  &  &  &  &  \\ 
    ## \multirow{ 4 }{*}{ 1 } & 50 &  & 0.06 & 0.30 & 0.83 & 0.98 & 1.00 & 1.00 \\ 
    ##  & 100 &  & 0.05 & 0.51 & 0.98 & 1.00 & 1.00 & 1.00 \\ 
    ##  & 250 &  & 0.06 & 0.89 & 1.00 & 1.00 & 1.00 & 1.00 \\ 
    ##  & 500 &  & 0.06 & 1.00 & 1.00 & 1.00 & 1.00 & 1.00 \\ 
    ##  &  &  &  &  &  &  &  &  \\ 
    ## \multirow{ 4 }{*}{ 2 } & 50 &  & 0.05 & 0.11 & 0.28 & 0.55 & 0.79 & 0.95 \\ 
    ##  & 100 &  & 0.05 & 0.16 & 0.54 & 0.84 & 0.97 & 1.00 \\ 
    ##  & 250 &  & 0.04 & 0.34 & 0.91 & 1.00 & 1.00 & 1.00 \\ 
    ##  & 500 &  & 0.04 & 0.58 & 0.99 & 1.00 & 1.00 & 1.00 \\ 
    ## \\
    ## \\
    ## \hline\hline
    ## \end{tabular}%
    ## }
    ## \caption{ decision  }
    ## \end{table}

Now you can copy the code and add it to our paper, report, or presentation. This automated process lets you focus on the design of your experiment and the interpretation of the results.

Additional package functionality
=================================

The MonteCarlo package provides several arguments that modify the behavior of the `MonteCarlo()` function and the appearance of tables generated by the `MakeTable()` function. The tables in the next section provide information about the available function arguments.

The MonteCarlo() Function
-------------------------

The following table lists additional arguments for the `MonteCarlo()` function. The most important arguments are *export\_also* which is needed to export datasets if they're required in a parallezied simulation, and *time\_n\_test* which produces an estimate of the time required for the desired simulation.

<table style="width:83%;">
<colgroup>
<col width="18%" />
<col width="65%" />
</colgroup>
<thead>
<tr class="header">
<th align="center">Argument</th>
<th align="left">Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="center">func</td>
<td align="left">The function to be evaluated.</td>
</tr>
<tr class="even">
<td align="center">nrep</td>
<td align="left">An integer that specifies the desired number of MonteCarlo repetitions.</td>
</tr>
<tr class="odd">
<td align="center">param_list</td>
<td align="left">A list of components named after the parameters of the function. Each component is a vector containing the desired grid values for that parameter.</td>
</tr>
<tr class="even">
<td align="center">ncpus</td>
  <td align="left">An integer specifying the number of CPUs to use. Default is <i>ncpus=1</i>. If <i>ncpus</i> is greater than 1, the simulation is parallized automatically using the <i>ncpus</i> CPU units.</td>
</tr>
<tr class="odd">
<td align="center">max_grid</td>
<td align="left">An integer that specifies for which grid size to throw an error, if the size becomes too large. Default is <i>max_grid=1000</i>.</td>
</tr>
<tr class="even">
<td align="center">time_n_test</td>
<td align="left">A boolean that specifies whether to estimate the required simulation time. This argument is useful for large simulations or slow functions. Default is time_n_test=FALSE.</td>
</tr>
<tr class="odd">
<td align="center">save_res</td>
  <td align="left">A boolean that specifies whether to save the results of <i>time_n_test</i> to the current directory. Default is <i>save_res=FALSE</i>.</td>
</tr>
<tr class="even">
<td align="center">raw</td>
<td align="left">A boolean that specifies whether to average the output over the *nrep* repetitions. Default is <i>raw=TRUE</i>.</td>
</tr>
<tr class="odd">
<td align="center">export\_also</td>
<td align="left">A list of additional objects to export to the cluster. Use this argument to export data or bypass the automatic export of functions. Default is <i>export\_also=NULL</i>.</td>
</tr>
</tbody>
</table>

The MakeTable() function
---------------------------

The following table lists additional arguments for the `MakeTable()` function. The most important arguments are *collapse* and *transform*, which let you determine how the results of a simulation are aggregated and presented. This capability makes the MonteCarlo package much more versatile. You can find additional examples that demonstrate how to use these arguments in the help file and in the [package vignettes](https://github.com/FunWithR/MonteCarlo/tree/master/vignettes).

<table style="width:83%;">
<colgroup>
<col width="18%" />
<col width="65%" />
</colgroup>
<thead>
<tr class="header">
<th align="center">Argument</th>
<th align="left">Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="center">output</td>
  <td align="left">A list of class MonteCarlo generated by the <i>MonteCarlo()</i> function</td>
</tr>
<tr class="even">
<td align="center">rows</td>
<td align="left">A vector of parameter names to be stacked in the rows of the table. The parameters are ordered from the inside to the outside.</td>
</tr>
<tr class="odd">
<td align="center">cols</td>
<td align="left">A vector of parameter names to be stacked in the columns of the table. The parameters are ordered from the inside to the outside.</td>
</tr>
<tr class="even">
<td align="center">digits</td>
  <td align="left">The number of digits displayed in table. Default is <i>digits=4</i>.</td>
</tr>
<tr class="odd">
<td align="center">collapse</td>
<td align="left">An optional list of functions to apply to the respective components of the output when collapsing the results in a table. The list must be the same length as the output. Default is <i>mean</i>, which calculates the average. Another example could be <i>sd()</i> for standard deviation.</td>
</tr>
<tr class="even">
<td align="center">transform</td>
<td align="left">An optional argument that transforms the output table. For example, you could transform the output from mean squared error (MSE) to root mean squared error (RMSE). If you supply a function, it's applied to all tables. Alternatively, you can supply a list of functions of the same length as the output. For tables that are supposed to remain unchanged, set the list element to <i>NULL</i>.</td>
</tr>
<tr class="odd">
<td align="center">include_meta</td>
<td align="left">A boolean that determines whether the metadata provided by the *summary()* function is included in comments below the table. Default is <i>include_meta==TRUE</i>.</td>
</tr>
<tr class="even">
<td align="center">width_mult</td>
<td align="left">The scaling factor for the width of the output table. Default is <i>width_mult=1</i>.</td>
</tr>
<tr class="odd">
<td align="center">partial_grid</td>
<td align="left">An optional list of elements named after the parameters for which only a subset of the grid values should be included in the table. Each component of the list is a vector that specifies the grid values of interest.</td>
</tr>
</tbody>
</table>
