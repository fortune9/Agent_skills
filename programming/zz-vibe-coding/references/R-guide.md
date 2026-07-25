
## Reference documentation

- for locally installed functions and packages, use the following
  methods to get their info:
  * use the mcp tool 'r-btw' if available
  * use the command like: Rscript -e 'help("geom_bar", package="ggplot2")'
  * use websites in the following order:
    1. https://rdrr.io/
    2. https://www.rdocumentation.org/
    3. https://tidyverse.org/
    4. https://cran.r-project.org/manuals.html
    5. https://cran.r-project.org/web/packages/available_packages_by_name.html


## Tests

- use `testthat` package for unit testing
- add R packages used in tests to `Suggests` field in `DESCRIPTION`
  file

