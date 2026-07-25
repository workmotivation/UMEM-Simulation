# Changelog

All notable changes to this project are documented here. The format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/) and this project
adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.1.0] - 2026-07-23

First public release.

### Added

- **Simulation engine.** Finite-lifetime, discrete-time model with nested time
  steps, seasons, and lifetimes; hierarchical heritable propensities over
  motivational domains and tasks; homeostatic internal state; a declarative
  task-effect rule system with no user code execution; eight environmental
  process types with noisy observation; seven attributed mortality sources;
  sexual and asexual reproduction with four mate-selection modes; recombination
  and multiplicative lognormal mutation.
- **Cognition.** Three independently switchable mechanisms: intergenerational
  success memory (A), noisy environmental forecasting (B), and individual
  reinforcement learning (C, off by default).
- **Experiments.** Single runs, replicated experiments, and parameter sweeps of
  up to three parameters, with process-based parallelism, cooperative
  cancellation, and optional common random numbers across sweep cells.
- **Analysis.** Descriptive metrics including realised lifetime reproductive
  success and Crow's opportunity for selection; all six specified cognitive
  misallocation metrics; replicate aggregation with t-based confidence
  intervals and Wilson intervals for extinction rates; a colourblind-safe
  figure set.
- **Exports.** Formatted Excel workbooks, CSV tables, PNG/SVG/PDF figures at up
  to 300 DPI, and a complete ZIP reproducibility bundle.
- **Interfaces.** A Streamlit web interface with Simple and Advanced modes, a
  PySide6 desktop application with a background worker thread, and a
  command-line interface for headless and batch use.
- **Legacy support.** A faithful reimplementation of the AOM 2.1 generational
  model for continuity, with its differences from the main engine documented.
- **Tests.** 145 automated tests spanning configuration, reproducibility,
  scientific mechanisms, regression, exports, and the web interface.

### Notes on calibration

Preset constants were selected by seed sweeps so that each scenario produces a
usable demographic trajectory. They are numerical demonstration settings and are
not empirically derived life-history parameters.
