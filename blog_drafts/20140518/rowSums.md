
# Benchmarking: Subsetted operations in matrixStats
Henrik Bengtsson on 2014-05-17

## Background
```r
> library("microbenchmark")
> set.seed(42)
> x <- matrix(rnorm(1e+05), ncol = 1000)
> rows <- 20 + 1:50
> xS <- x[rows, , drop = FALSE]
> gc()
          used (Mb) gc trigger (Mb) max used (Mb)
Ncells  606597 32.4     984024 52.6   984024 52.6
Vcells 1001688  7.7    1757946 13.5  1757945 13.5
> stats <- microbenchmark(A = rowSums(x[rows, , drop = FALSE]), B = rowSums(xS), times = 1000L)
```



|expr |     min|       lq|  median|       uq|       max| neval|
|:----|-------:|--------:|-------:|--------:|---------:|-----:|
|A    | 477.727| 549.1355| 583.781| 718.8995| 87803.786|  1000|
|B    | 117.027| 120.8760| 129.345| 156.8690|   485.041|  1000|

![](figures/matrixStats,subsetting,benchmark.png)


## Subsetted operators in matrixStats


[Git]: http://git-scm.com/
[Subversion]: http://subversion.apache.org/
[R-Forge]: http://r-forge.r-project.org/


