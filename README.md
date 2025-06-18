# Probability Estimation Using Monte Carlo Simulation of Boolean Logic on Hardware-Accelerated Platforms

## Quick Links
- [Presented Slides](./exported/slides_earthperson_psa_2025.pdf)
- [Detailed Slides from Prelim](./exported/slides_earthperson_prelim.pdf)
- [Theoretical Framework from Prelim Report](./exported/report_earthperson_prelim.pdf)
- [Theoretical Framework from DOE NEUP Report](./exported/report_earthperson_excerpts_from_neup.pdf)


## An Informal Overview
Probabilistic risk assessment (PRA) for nuclear systems requires evaluating complex Boolean
functions that represent failure scenarios across thousands of components in two main steps:
identifying minimal cut sets (the smallest sets of conditional events leading to end states
of interest) and estimating the likelihood of these end states given component reliabilities.
Although additional steps exist, PRA quantification primarily revolves around these tasks,
which are computationally burdensome because exact solutions involve exhaustive inclusion-
exclusion calculations that grow exponentially with the number of components, making
large-scale analyses intractable. In response, many approximations have emerged: restricting
model logic to a limited set of gate-types, describing only the failure space, applying probability
truncation schemes like the rare-event approximation, or employing bounding methods such
as the min-cut upper bound. Yet, even with these strategies, PRA quantification remains
unwieldy in practice, demanding careful model management and tempered expectations
regarding both accuracy and runtime.

## Core Rationale - Building a Probability Estimator
We propose a data-parallel Monte Carlo (MC) scheme that samples directly from input
component-reliability distributions. Rather than generating every possible failure combination,
we focus on random draws of the system’s global state, then tally how often particular
sequences lead to an end state of interest. This sampling-based approach trades the intrinsic
explosion of inclusion-exclusion terms with an exponential number of sample draws. This
approach can accommodate a wider range of logic gates and event dependencies while putting
the analyst in control of convergence. To unify these ideas, we use the concept of probabilistic
circuits. Here, probability flow is represented as a structured computation graph over Boolean
logic, but crucially, each gate’s input distributions come from random samplers rather than a
symbolic enumeration of all possible states. This allows the circuit to model both failures and
successes while keeping the model size manageable. However, MC approaches for PRA model
quantification are not new. The key design principle rests on a massively parallel, hardware
accelerated sampling scheme. By encoding component states in compact bit vectors, we
exploit hardware-level parallelism. Boolean logic gates, including primitives and complex logic
such as K/N translate into efficient bitwise operations. These operations are performed as a
computation graph defined by the logic structure of the PRA model itself. We implemented
the framework in an open-source tool named Canopy, which is built with SYCL to promote
code portability across consumer GPUs, multi-core CPUs, or specialized accelerators like
FPGAs.

## The Role of Partial Derivatives
One notable benefit of modeling probability distributions over the entire Boolean space is the
ability to compute partial derivatives using the Shannon decomposition. Conceptually, the
Shannon decomposition defines the derivative of Boolean function f(x), with respect to a subset
of inputs. It is efficiently evaluated as a combination of bitwise XOR operations. Our approach
samples from computation graphs representing arbitrary Boolean functions, including partial
derivatives or more complex operations. This has immediate use for computing not only the
conventional Birnbaum importance measure (which indicates how sensitive a system’s failure
probability is to a component’s reliability) but also expanding to multi-component subsets.
Additionally, since derivatives allow for gradient-based optimizations, there is potential for
future work bridging advanced PRA and emerging machine-learning techniques. By embedding
an auto-differentiation layer in the MC pipeline, one can imagine learning gate probabilities
or even partial system structures from operational data, with the same data-parallel routines
driving updates to reliability estimates.

## Limitations
### Sampling Rare Events
Ensuring accurate estimates for extremely low-probability events demands careful convergence
criteria. In typical nuclear safety analyses, component failures can be vanishingly rare, so
polling enough samples to capture these tail probabilities can prove challenging. Techniques
such as importance sampling or other variance-reduction strategies may be needed to mitigate
the slow convergence that arises when event likelihoods are measured in the range of 10-7 or
lower.

### Sampling Correlated or Dependent Events
A related concern is the treatment of common-cause failures (CCFs) and component depen-
dencies. Characterizing CCF related failures or performing dependency analysis are important
PRA activities. Future extensions of this framework thus require sampling from joint distribu-
tions, where correlated events are generated as a block. Integrating such dependent draws into
bitpacked logic operations is feasible in principle but will require additional data-processing
layers to ensure that correlated assignments remain both realistic and efficient to evaluate in
large batches.