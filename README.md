![WhyNot Logo](docs/source/_static/WhyNot_fullcolor.svg)

[![Build Status](https://travis-ci.com/zykls/whynot.svg?token=ERpRX6SmHRsKJ8dNb4QV&branch=master)](https://travis-ci.com/zykls/whynot)
[![Documentation Status](https://readthedocs.com/projects/whynot-docs/badge/?version=latest)](https://whynot-docs.readthedocs-hosted.com/en/latest/?badge=latest)

**WhyNot** is a Python package that provides an experimental sandbox for causal
inference. The package facilitates development, testing, and benchmarking causal
inference and decision making in challenging dynamic environments.

For more detailed information, check out the [documentation](https://whynot-docs.readthedocs-hosted.com/en/latest/).

## Table of Contents
1. [Basic installation instructions](#basic-installation-instructions)
2. [Quick start examples](#quick-start-examples)
3. [Simulators in WhyNot](#simulators-in-whynot)
4. [Using estimators in R](#using-estimators-in-r)
5. [Frequently asked questions](#frequently-asked-questions)

WhyNot is still under active development! If you find bugs or have feature
requests, please file a 
[Github issue](https://github.com/zykls/whynot/issues). We welcome all kinds of issues, especially those related to correctness, documentation, performance, and new features.

## Basic installation instructions
1. (Optionally) create a virtual environment
```
python3 -m venv whynot-env
source whynot-env/bin/activate
```
2. Install via pip
```
pip install whynot
```

You can also install WhyNot directly from source.
```
git clone https://github.com/zykls/whynot.git
cd whynot
pip install -r requirements.txt
```

## Quick start examples

### Average Treatment Effect Experiment
Every simulator in WhyNot comes equipped with a set of experiments probing
different aspects of causal inference. In this section, we show how to run
experiments probing average treatment effect estimation on the World 3
simulator. World3 is a dynamical systems model that studies the interplay
between natural resource constraints, population growth, and industrial
development.

First, we examine all of the experiments available for World3.
```py
import whynot as wn
experiments = wn.world3.get_experiments()
print([experiment.name for experiment in experiments])
#['world3_rct', 'world3_pollution_confounding', 'world3_pollution_unobserved_confounding', 'world3_pollution_mediation']
```
These experiments generate datasets both in the setting of a pure randomized
control trial (`world3_rct`), as well as with (unobserved) confounding and
mediation. We will run a randomized control experiment. The description property
offers specific details about the experiment.
```py
rct = wn.world3.PollutionRCT
rct.description
#'Study effect of intervening in 1975 to decrease pollution generation on total population in 2050.'
```
We can run the experiment using the experiment `run` function and specifying a
desired sample size `num_samples`. The experiment then returns a causal
`Dataset` consisting of the covariates for each unit, the treatment assignment,
the outcome, and the ground truth causal effect for each unit.
```py
dset = rct.run(num_samples=500, show_progress=True)
(X, W, Y) = dset.covariates, dset.treatments, dset.outcomes
```

Using the dataset generated by WhyNot, we can then compare the true sample
average treatment effect with causal estimates. Since we ran a randomized
control trial, we compare the difference in means with the true effect.
```py
import numpy as np
true_sate = dset.sate
(X, W, Y) = dset.covariates, dset.treatments, dset.outcomes
estimated_ate = np.mean(Y[W == 1.]) -  np.mean(Y[W  == 0.])
relative_error = np.abs((estimated_ate - true_sate) / true_sate)
print("Relative Error in causal estimate: {}".format(relative_error))
#'Relative Error in causal estimate: 0.53'
```

### Using Causal Estimators

After generating the dataset, WhyNot enables you to run a large collection of
causal estimators on the data for benchmarking and comparison. The main function
to do this is the `causal_suite` which, given the causal dataset, runs all of
the estimators on the dataset and returns an `InferenceResult` for each estimator
containing its estimated treatment effects and uncertainty estimates like
confidence intervals. 

```py
import whynot as wn

# Generate the dataset
rct = wn.world3.PollutionRCT
dataset = rct.run(num_samples=500, show_progress=True)

# Run the suite of estimates
estimated_effects = wn.causal_suite(
    dataset.covariates, dataset.treatments, dataset.outcomes)

# Evaluate the relative error of the estimates
true_sate = dataset.sate
for estimator, estimate in estimated_effects.items():
    relative_error = np.abs((estimate.ate - true_sate) / true_sate)
    print("{}: {:.2f}".format(estimator, relative_error))
# ols: 0.50
# propensity_weighted_ols: 0.51
# propensity_score_matching: 0.28
# matching: 0.75
# causal_forest: 0.06
# tmle: 0.06
```

In addition to experiments studying average treatment effect, WhyNot also supports
experiments studying
1. Heterogeneous treatment effects,
2. Causal structure discovery,
3. Time-varying treatments, sequential decision making, and reinforcement learning.

For more examples and demonstrations of how to design and conduct
experiments in each of these settings, check out 
[usage](https://whynot-docs.readthedocs-hosted.com/en/latest/usage.html) and
our collection of
[examples](https://whynot-docs.readthedocs-hosted.com/en/latest/examples.html).


## Simulators in WhyNot
WhyNot provides a large number of simulated environments from fields ranging
from economics to epidemiology. Each simulator comes equipped with a
representative set of causal inference experiments and exports a uniform
Python interface that makes it easy to construct new causal inference
experiments in these environments.

The simulators in WhyNot currently include:
- Adams HIV (ODE-based HIV simulator)
- Dynamic Integrated Climate Economy Model (DICE)
- World3
- World2
- Opioid Epidemic Simulator
- Lotka-Volterra Model
- Incarceration Simulator 
- Civil Violence Simulator
- Schelling Model
- LaLonde Synthetic Outcome Model

For an overview of these simulators, please see the [simulator
documentation](https://whynot-docs.readthedocs-hosted.com/en/latest/simulators.html).

## Using estimators in R
WhyNot ships with a small set of causal estimators written in pure Python.
To access other estimators, please install the companion library
[whynot_estimators](https://github.com/zykls/whynot_estimators), which includes a host of
state-of-the-art causal inference methods implemented in R.

To get the basic framework, run
```
pip install whynot_estimators
```
If you have R installed, you can install the `causal_forest` estimator by using
```
python -m  whynot_estimators install causal_forest
```
To see all of the available estimators, run
```
python -m  whynot_estimators show_all
```
See [whynot_estimators](https://github.com/zykls/whynot_estimators) for
instructions on installing specific estimators, especially if you do not have an
existing R build.


## Frequently asked questions
**1. Why is it called WhyNot?**

Why not?

**2. What are the intended use cases?**

WhyNot supports multiple use cases, some technical, some pedagogical, each
suited for a different group of users. We envision at least five primary use
cases:

- **Developing**: Researchers can use WhyNot in the process of developing new causal inference methods. WhyNot can serve as a substitute for ad-hoc synthetic data where needed, providing a greater set of challenging test cases. 

- **Testing**: Researchers can use WhyNot to design robustness checks for methods and gain insight into the failure cases of these methods.

- **Benchmarking**: Practitioners can use WhyNot to compare multiple methods on the same set of tasks. WhyNot does not dictate any particular benchmark, but rather supports the community in creating useful benchmarks.

- **Learning**: Students of causality might find WhyNot to be a helpful training resource. WhyNot is easy-to-use and does not require much prior experience to get started with.

- **Teaching**: Instructors can use WhyNot as a tool students engage with to learn and solve problems.

**3. What uses are *not* intended?**

- **Basis of real-world policy and interventions**: The simulators included in WhyNot were selected because they offer realistic technical challenges for cause inference methods, not because they offer faithful models of the real world. In many cases, they have been contested or criticized as representations of the real world. For this reason, the simulators should not directly be used to design real-world interventions or policy.

- **Substitute for healthy debate**: Success in simulated environments does not guarantee success in real scenarios, but a failure in simulated environments can nonetheless lead to insight into weaknesses of a particular approach. WhyNot does not obviate the need for debate around common assumptions in causal inference.

- **Substitute for real world experiments and data**: WhyNot does not substitute for high-quality empirical work on real data sets. WhyNot is a tool for understanding and evaluating causal inference methods, not certifying their validity in real-world scenarios. 

- **Substitute for theory**: WhyNot can help create understanding in contexts where theoretical analysis is challenging, but does not reduce the need for theoretical guarantees and formal analysis in causal inference.


**4. Why start from dynamical systems?**

Dynamical systems provide a natural setting to study causal inference. The
physical world is a dynamical system, and causal inference inevitably has to
grapple with data generated from some dynamical process. Moreover, the temporal
structure of the dynamics gives rise to nontrivial problem instances with both
confounding and mediation. Dynamics also naturally lead to time-varying causal
effects and allow for time-varying treatments and sequential decision making.

**5. What what simulators are included and why?**

WhyNot contains a range of different simulators, and an overview is provided in
the documentation
[here](https://whynot-docs.readthedocs-hosted.com/en/latest/).


**6. What’s the difference between WhyNot and CauseMe?**

[CauseMe](https://causeme.uv.es) is an online platform for benchmarking causal
inference methods. Users can register and evaluate causal inference methods on
an existing repository of data sets, or contribute their own data sets with
known ground truth. CauseMe is an excellent platform that we recommend in
addition to WhyNot. We encourage users to export data sets derived from WhyNot
and make them accessible through CauseMe. In this case, we ask that you
reference WhyNot.


**7. What’s the difference between WhyNot and CausalML?**

CausalML is a Python package that provides a range of causal inference methods.
The estimators provided by CausalML are available in WhyNot via the
[whynot_estimators](https://github.com/zykls/whynot_estimators)
package.  While WhyNot provides simulators and derived
experimental designs on synthetic data, CausalML focuses on providing
estimators. We made these estimators available for use on top of WhyNot.

**8. What’s the difference between WhyNot and EconML?**

EconML is a Python package that provides tools from machine learning and
econometrics for causal inference. Like CausalML, EconML focuses on providing
estimators, and we made these estimators available for use on top of WhyNot.

**9. How can I best contribute to WhyNot?**

Thanks so much for considering to contribute to WhyNot. The package is open
source and MIT licensed. We invite contributions broadly in a number of areas,
including the addition of simulators, causal estimators, documentation,
performance improvements, code quality and tests.
