# ADR 0001: Reproducibility strategy for NorthStar models

## Context

NorthStar Logistics currently stores trained models directly in S3 using inconsistent naming conventions such as eta_v2_FINAL.onnx. The team has no dataset versioning, no centralized registry, and no guaranteed method for reproducing training runs.

## Decision

### Environment

All training and inference environments will be containerized using Docker. Each model training run will store the Docker image digest and Python dependency lockfile. CI pipelines in GitHub Actions will build immutable images for every release.

### Data

Training datasets will be versioned using DVC (Data Version Control) with S3 as remote storage. Every training run must reference a specific dataset hash and frozen evaluation dataset version. Feature schemas will be validated using Great Expectations before training begins.

### Code

All experiments must be tied to a Git commit SHA stored in MLflow. Model artifacts, metrics, parameters, and lineage metadata will be tracked inside MLflow Tracking Server. Pull requests and code review will be mandatory before promotion to production.

### Randomness

Training runs will use fixed random seeds for Python, NumPy, and framework-level randomness. Seeds must be logged in MLflow metadata. GPU nondeterministic operations will be disabled where supported.

## Alternatives rejected

* Using only S3 timestamps for dataset tracking because timestamps do not guarantee dataset integrity.
* Allowing engineers to manually name model files because naming conventions drift over time.
* Reproducing experiments from notebooks alone because notebook state is not deterministic.

## Consequences

* Engineers must maintain dataset versions and metadata for every experiment.
* CI/CD pipelines become mandatory for all production promotions.
* Storage usage will increase because historical datasets and model artifacts are retained.

## Revisit if

* The company becomes subject to regulatory audit requirements or expands to multi-region training infrastructure.
