
# Benchmarking: Startup cost of the 'methods' package
Henrik Bengtsson on 2014-05-17

You might have noticed that `R` attaches the 'methods' package by default whereas `Rscript` does not.  Compare
```
% R --silent -e "search()"
> search()
[1] ".GlobalEnv"        "package:stats"     "package:graphics" 
[4] "package:grDevices" "package:utils"     "package:datasets" 
[7] "package:methods"   "Autoloads"         "package:base"     
> 
> 
```
with
```
% Rscript -e "search()"
[1] ".GlobalEnv"        "package:stats"     "package:graphics" 
[4] "package:grDevices" "package:utils"     "package:datasets" 
[7] "Autoloads"         "package:base"     
```

The reason for this is that there is a substantial overhead in loading the 'methods' package.  From `help("Rscript")` we can read

> The default for Rscript omits methods as it takes about 60% of the startup time.


Here is how you can measure this yourself.  Let's start by defining function `Rscript()` for calling `Rscript` from within R.  It takes the expression to be avaulated as argument `expr` and the packages to be attached when loaded as argument `pkgs`.
```r
> Rscript <- function(expr, pkgs = NULL) {
+     args <- c("-e", shQuote(paste(expr, collapse = "\n")))
+     args <- c("--vanilla", args)
+     if (length(pkgs) > 0) 
+         args <- c(sprintf("--default-packages=%s", paste(pkgs, collapse = ",")), args)
+     system2(file.path(R.home("bin"), "Rscript"), args = args, stdout = TRUE)
+ }
```

With this function we can now run Rscript with an R expression as:
```r
> Rscript("cat('Hello world!\n')", pkgs = "base")
[1] "Hello world!"
```
which will run `Rscript` with only the 'base' package loaded.


Finally, let's benchmark `Rscript` with and without the 'methods' package being attached:
```r
> library("microbenchmark")
> without_methods <- function() Rscript("cat('Hello world!\n')", pkgs = "base")
> with_methods <- function() Rscript("cat('Hello world!\n')", pkgs = c("base", "methods"))
> stats <- microbenchmark(with_methods(), without_methods(), times = 10L)
> stats
Unit: milliseconds
              expr      min       lq   median       uq      max neval
    with_methods() 301.9947 302.0017 302.0088 302.0267 303.0191    10
 without_methods() 201.9699 201.9845 202.0155 202.3587 202.9888    10
```
This shows that attaching 'methods' increases the startup time by 49%.

Although the increase is only a fraction of a second in absolute terms, it adds an unnecessary overhead if the 'methods' package is not needed.  This may be noticable if the script itself is short and designed to return as quick as possible.


