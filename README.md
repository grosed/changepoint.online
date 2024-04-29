# changepoint_online


- [Installation](#installation)
  - [using pip](#using-pip)
- [Key Features](#key-features)
- [Examples](#examples)
  - [Simple Gaussian Change-in-mean](#simple-gaussian-change-in-mean)
  - [Change in one-parameter exponential family
    distributions](#change-in-one-parameter-exponential-family-distributions)
  - [Non-Parametric changepoint
    detection](#non-parametric-changepoint-detection)
- [License](#license)
- [GitHub Repository](#github-repository)
- [How to Cite This Work](#how-to-cite-this-work)
- [References](#references)

<span style="h1">**A Collection of Methods for Online Changepoint
Detection**</span>

The changepoint_online package provides efficient algorithms for
detecting changes in data streams based on the Focus algorithm. The
Focus algorithm solves the CUSUM likelihood-ratio test exactly in
$O(\log(n))$ time per iteration, where n represents the current
iteration. The method is equivalent to running a rolling window (MOSUM)
simultaneously for all sizes of windows or the Page-CUSUM for all
possible values of the size of change (an infinitely dense grid).

## Installation

### using pip

#### installing from PyPI

    pip install changepoint-online

#### installing from github with pip

    python -m pip install 'git+https://github.com/grosed/changepoint_online/#egg=changepoint_online&subdirectory=python/package'

## Key Features

- Contains all `Focus` exponential family algorithms as well as the
  `NPFocus` algorithm for non-parametric changepoint detection.

- It’s versatile enough to be applied in scenarios where the pre-change
  parameter is either known or unknown.

- It is possible to apply constraints to detect specific types of
  changes (such as increases or decreases in parameter values).

## Examples

### Simple Gaussian Change-in-mean

``` python
from changepoint_online import Focus, Gaussian
import numpy as np

# generating some data with a change at 50,000
np.random.seed(0)
Y = np.concatenate((np.random.normal(loc=0.0, scale=1.0, size=50000), np.random.normal(loc=2.0, scale=1.0, size=5000)))

# initialize a Focus Gaussian detector
detector = Focus(Gaussian())
threshold = 13.0

for y in Y:
    # update your detector sequentially with
    detector.update(y)
    if detector.statistic() >= threshold:
        break

print(detector.changepoint())
```

    {'stopping_time': 50013, 'changepoint': 50000}

If the pre-change location is known (in case of previous training data),
this can be specified with:

``` python
# initialize a Focus Gaussian detector (with pre-change location known)
detector = Focus(Gaussian(loc=0))
threshold = 13.0

for y in Y:
    # update your detector sequentially with
    detector.update(y)
    if detector.statistic() >= threshold:
        break

print(detector.changepoint())
```

    {'stopping_time': 50013, 'changepoint': 50000}

See `help(FamilyName)` for distribution specific parameters,
e.g. `help(Gaussian)`.

### Change in one-parameter exponential family distributions

As in the Gaussian case, we can detect:

- Poisson change-in-rate

- Gamma change-in-scale (or rate). Exponential change-in-rate
  implemented as a Gamma `shape=1`.

- Bernoulli change-in-probability

For example:

``` python
from changepoint_online import Focus, Gamma
import numpy as np 

np.random.seed(0)
Y = np.concatenate((np.random.gamma(4.0, scale=3.0, size=50000), 
                    np.random.gamma(4.0, scale=6.0, size=5000)))

# initialize a Gamma change-in-scale detector (with shape = 4)
detector = Focus(Gamma(shape=4.0))
threshold = 12.0
for y in Y:
    detector.update(y)
    if detector.statistic() >= threshold:
        break
        
print(detector.changepoint())
```

    {'stopping_time': 50012, 'changepoint': 50000}

### Non-Parametric changepoint detection

If we do not know the underlying distribution, or if the nature of the
change is unkown *a priori,* we can then use `NPFocus`.

``` python
from changepoint_online import NPFocus
import numpy as np

# Define a simple Gaussian noise function
def generate_gaussian_noise(size):
    return np.random.normal(loc=0.0, scale=1.0, size=size)

# Generate mixed data with change in gamma component
gamma_1 = np.random.gamma(4.0, scale=3.0, size=5000)
gamma_2 = np.random.gamma(4.0, scale=6.0, size=5000)
gaussian_noise = generate_gaussian_noise(10000)
Y = np.concatenate((gamma_1 + gaussian_noise[:5000], gamma_2 + gaussian_noise[5000:]))

# Create and use NPFocus detector
## One needs to provide some quantiles to track the null distribuition over
quantiles = [np.quantile(Y[:100], q) for q in [0.25, 0.5, 0.75]]
## the detector can be initialised with those quantiles
detector = NPFocus(quantiles)

stat_over_time = []

for y in Y:
    detector.update(y)
    # we can sum the statistics over to get a detection
    # see  (Romano, Eckley, and Fearnhead 2024) for more details
    if np.sum(detector.statistic()) > 25:
        break


changepoint_info = detector.changepoint()
print(changepoint_info["stopping_time"])
```

    5014

## License

Copyright (C) 2023 Gaetano Romano, Daniel Grose

This program is free software: you can redistribute it and/or modify it
under the terms of the GNU General Public License as published by the
Free Software Foundation, either version 3 of the License, or (at your
option) any later version.

This program is distributed in the hope that it will be useful, but
WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU General
Public License for more details.

You should have received a copy of the GNU General Public License along
with this program. If not, see
<https://www.gnu.org/licenses/gpl-3.0.txt>.

## GitHub Repository

Source files for the packages can be found at
<https://github.com/grosed/changepoint_online>

## How to Cite This Work

A possible BibTeX entry for this package could be:

    @software{changepoint_online,
      author       = {Daniel Grose, Gaetano Romano},
      title        = {changepoint_online: A Collection of Methods for Online Changepoint Detection.},
      month        = Apr,
      year         = 2024,
      version      = {v1.0.0},
      url          = {https://https://github.com/grosed/changepoint_online}
    }

For citing the methodologies:

- Gaussian FOCuS: (Romano et al. 2023)

- Other Exponential Family detectors: (Ward et al. 2024)

- NPFocus: (Romano, Eckley, and Fearnhead 2024)

See references below.

## References

<div id="refs" class="references csl-bib-body hanging-indent"
entry-spacing="0">

<div id="ref-romano2024" class="csl-entry">

Romano, Gaetano, Idris A. Eckley, and Paul Fearnhead. 2024. “A
Log-Linear Nonparametric Online Changepoint Detection Algorithm Based on
Functional Pruning.” *IEEE Transactions on Signal Processing* 72:
594–606. <https://doi.org/10.1109/tsp.2023.3343550>.

</div>

<div id="ref-romano2023fast" class="csl-entry">

Romano, Gaetano, Idris A Eckley, Paul Fearnhead, and Guillem Rigaill.
2023. “Fast Online Changepoint Detection via Functional Pruning CUSUM
Statistics.” *Journal of Machine Learning Research* 24 (81): 1–36.
<https://www.jmlr.org/papers/v24/21-1230.html>.

</div>

<div id="ref-ward2024constant" class="csl-entry">

Ward, Kes, Gaetano Romano, Idris Eckley, and Paul Fearnhead. 2024. “A
Constant-Per-Iteration Likelihood Ratio Test for Online Changepoint
Detection for Exponential Family Models.” *Statistics and Computing* 34
(3): 1–11.

</div>

</div>
