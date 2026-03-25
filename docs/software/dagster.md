# Dagster

**Relevant to:** Users managing or debugging the MAXIMA automated data pipeline  
**Last Updated:** March 2025  
**Author:** Harrison Park

---

## Overview

**Dagster** is an open-source data orchestration platform used to define, schedule, and monitor
data processing workflows as directed acyclic graphs (DAGs) of tasks called **ops** and **assets**.

In MAXIMA, Dagster orchestrates the automated data processing pipeline that runs continuously
during and after experiments. As described in the MAXIMA publication (Park et al., 2025,
arXiv:2511.14905):

> "Data processing utilizes automated workflow orchestration using Dagster. When new XRD data
> are streamed, the workflow performs a correction for detector tilt and calibrates the
> sample-to-detector distance. [...] XRF data are similarly streamed and analyzed using a
> PyMCA fundamental parameters pipeline. Processed data are written back to the data platform
> with concomitant metadata registration. All steps run inside containerized environments,
> enabling reproducible execution and isolation. Tasks are executed in parallel where possible
> to reduce overall latency and support high-throughput operation."

Dagster documentation: [https://docs.dagster.io/](https://docs.dagster.io/)

---

## Role in the MAXIMA Pipeline

The Dagster pipeline connects OpenMSIStream (data ingestion) with PyFAI (XRD reduction)
and PyMCA (XRF analysis):

```
Raw .h5 / .mca data (streamed via OpenMSIStream)
        │
        ▼
  Dagster workflow triggered
        │
        ├──► XRD branch (PyFAI)
        │       ├── Detector tilt correction
        │       ├── Beam center / detector distance calibration
        │       └── Azimuthal integration → 1D I(q) output
        │
        └──► XRF branch (PyMCA)
                ├── Fundamental parameters peak fitting
                └── Elemental concentration output
        │
        ▼
  Processed data + metadata written back to data platform
```

All steps run in **containerized environments** (Docker), ensuring reproducibility. XRD and
XRF branches run in **parallel** to minimize latency.

---

## Installation (Local Development)

To run or develop the Dagster pipeline locally:

```bash
# Create environment
conda create -n dagster python=3.10
conda activate dagster

# Install Dagster and the web UI
pip install dagster dagster-webserver

# Install pipeline-specific dependencies
pip install pyFAI PyMca5 fabio silx h5py
```

> **TODO:** Add the specific Dagster version pinned in the AIMD-L pipeline and link to the
> `requirements.txt` or `pyproject.toml` from the repository.

---

## Starting the Dagster Web UI

The Dagster UI (Dagit) provides a visual interface for monitoring pipeline runs, viewing asset
lineage, and triggering manual runs.

```bash
conda activate dagster
cd /path/to/pipeline/repo
dagster dev
```

Then open `http://localhost:3000` in a browser.

> **TODO:** Document the correct repository path and any environment variables that need to
> be set before starting the server (e.g., data platform credentials, storage paths).

---

## Key Concepts

| Concept | Meaning in MAXIMA Context |
|---------|--------------------------|
| **Asset** | A persistent data artifact produced by the pipeline (e.g., a calibrated `.poni` file, an integrated 1D pattern, a concentration table) |
| **Op** | A single computation step (e.g., "run PyFAI integration on this `.h5` file") |
| **Job** | A named collection of ops/assets that can be run together |
| **Sensor** | A trigger that watches for new data and launches a job automatically (used here to watch for new streamed files) |
| **Run** | A single execution of a job |

---

## Monitoring Pipeline Runs

In the Dagster UI:

1. Navigate to **Runs** to see all pipeline executions with status (success / failure / in progress).
2. Click a run to see per-op execution logs and timing.
3. Navigate to **Assets** to see the lineage and materialization history of processed outputs.

If a run fails, click into the failed op to see the stack trace. Common failure modes are in
[Troubleshooting](../troubleshooting.md).

---

## Re-running Failed Jobs

If a pipeline run fails partway through:

```python
# From the Dagster UI, use "Re-execute from failure" to retry only the failed ops.
# Alternatively, trigger a full re-run from the Launchpad for a specific file.
```

> **TODO:** Document the correct way to trigger a manual re-run for a specific file or scan
> point from the AIMD-L pipeline.

---

## Containerized Execution

All Dagster ops run inside Docker containers. This means:

- Dependencies are isolated from the host system
- The same container image used in production can be run locally for debugging
- Code changes require rebuilding the relevant container image before they take effect

> **TODO:** Document the container image names, rebuild process, and deployment workflow for
> the AIMD-L Dagster pipeline.

---

## Changelog

| Version | Date | Author | Notes |
|---------|------|--------|-------|
| 1.0 | March 2025 | Harrison Park | Initial draft |
