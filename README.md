[![Build Status](https://travis-ci.org/swager/grf.svg?branch=master)](https://travis-ci.org/swager/grf)
![CRAN Downloads overall](http://cranlogs.r-pkg.org/badges/grand-total/grf)

# grf: generalized random forests

This repository is in an 'beta' state, and is actively under development. We expect to make continual improvements to performance and usability.

### Authors

This package is written and maintained by Julie Tibshirani (jtibs@cs.stanford.edu), Susan Athey, and Stefan Wager.

The repository first started as a fork of the [ranger](https://github.com/imbs-hl/ranger) repository -- we owe a great deal of thanks to the ranger authors for their useful and free package.

### Installation

The latest release of the package can be installed through CRAN:

```R
install.packages("grf")
```

Any published release can also be installed from source:

```R
install.packages("https://raw.github.com/swager/grf/master/releases/grf_0.9.3.tar.gz", repos = NULL, type = "source")
```

Note that to install from source, a compiler that implements C++11 is required (clang 3.3 or higher, or g++ 4.8 or higher). If installing on Windows, the RTools toolchain is also required.

### Usage Examples

```R
library(grf)

# Generate data.
n = 2000; p = 10
X = matrix(rnorm(n*p), n, p)
X.test = matrix(0, 101, p)
X.test[,1] = seq(-2, 2, length.out = 101)

# Perform treatment effect estimation.
W = rbinom(n, 1, 0.5)
Y = pmax(X[,1], 0) * W + X[,2] + pmin(X[,3], 0) + rnorm(n)
tau.forest = causal_forest(X, Y, W)
tau.hat = predict(tau.forest, X.test)
plot(X.test[,1], tau.hat$predictions, ylim = range(tau.hat$predictions, 0, 2), xlab = "x", ylab = "tau", type = "l")
lines(X.test[,1], pmax(0, X.test[,1]), col = 2, lty = 2)

# Estimate the conditional average treatment effect on the full sample (CATE).
estimate_average_effect(tau.forest, target.sample = "all")

# Estimate the conditional average treatment effect on the treated sample (CATT).
# Here, we don't expect much difference between the CATE and the CATT, since
# treatment assignment was randomized.
estimate_average_effect(tau.forest, target.sample = "treated")

# Add confidence intervals for heterogeneous treatment effects; growing more trees is now recommended.
tau.forest = causal_forest(X, Y, W, num.trees = 4000)
tau.hat = predict(tau.forest, X.test, estimate.variance = TRUE)
sigma.hat = sqrt(tau.hat$variance.estimates)
plot(X.test[,1], tau.hat$predictions, ylim = range(tau.hat$predictions + 1.96 * sigma.hat, tau.hat$predictions - 1.96 * sigma.hat, 0, 2), xlab = "x", ylab = "tau", type = "l")
lines(X.test[,1], tau.hat$predictions + 1.96 * sigma.hat, col = 1, lty = 2)
lines(X.test[,1], tau.hat$predictions - 1.96 * sigma.hat, col = 1, lty = 2)
lines(X.test[,1], pmax(0, X.test[,1]), col = 2, lty = 1)
```

For examples on how to use other types of forest, including those for [quantile regression](documentation/quantile_examples.md) and causal effect estimation using instrumental variables, please see the `documentation` directory.

### Developing

In addition to providing out-of-the-box forests for quantile regression and causal effect estimation, grf provides a framework for creating forests tailored to new statistical tasks. If you'd like to develop using grf, please consult the [development guide](DEVELOPING.md).

### References

Susan Athey, Julie Tibshirani and Stefan Wager.
<b>Generalized Random Forests</b>, 2016.
[<a href="https://arxiv.org/abs/1610.01271">arxiv</a>]
