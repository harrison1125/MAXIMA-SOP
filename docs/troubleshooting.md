# Troubleshooting

**Relevant to:** All operators and users of MAXIMA  
**Last Updated:** March 2025  
**Author:** Harrison Park

---

## Overview

This page documents known issues, error states, and their resolutions. For safety-critical
issues (interlock failures, E-stop behavior, radiation concerns), always escalate to Proto
or AIMD-L personnel rather than attempting to resolve them independently.

For contact information, see [Safety — Points of Contact](safety.md#points-of-contact).

---

## Hardware Issues

### Interlock will not reset after E-stop

**Symptom:** After pressing the emergency stop, the interlock does not clear when the E-stop
button is physically released.

**Likely cause:** The E-stop circuit may have a bi-stable latch behavior — either in hardware
or software. The exact mechanism is currently unclear.

**Resolution:** This requires **remote access from Proto** to resolve. Do not attempt to bypass
the interlock manually. Contact Proto support directly:
- Shaun Ostoic (software): sostoic@protoxrd.com
- Rob Drake (lead): rdrake@protoxrd.com
- General support: support@protoxrd.com

---

### Timestamp shifts after power cycle

**Symptom:** After shutting MAXIMA off and back on, data timestamps are slightly offset
from previous sessions.

**Likely cause:** Known instrument behavior — the system clock or timestamping reference
resets on restart.

**Resolution:** This is a known issue without a root-cause fix. Account for the timestamp
offset during data post-processing. Note the time of the restart to use as a reference.

---

## XRF Issues

### XRF files not saving / all intensities are zero

**Symptom:** XRF `.mca` files are created but contain all zero counts, or files are not
created at all.

**Likely cause:** Communication fault between the SDD detector and the acquisition system,
often caused by a loose cable connection.

**Resolution:**
1. Disconnect all cables connected to the XRF detector.
2. Reconnect all cables firmly.
3. Power cycle MAXIMA.
4. Retry the scan.

> This is a known workaround, not a permanent fix. If the issue recurs frequently, contact
> Proto Hardware (Eric Schober: eshober@protoxrd.com).

---

## Software Issues

### 500 server error when uploading JSON

**Symptom:** A 500 Internal Server Error is returned when uploading a scan configuration
JSON to the AIMDRC API.

**Likely cause:** The JSON body is malformed — a syntax error in the scan configuration.

**Resolution:**
1. Copy the JSON from your script.
2. Validate it using an online JSON linter (e.g., [jsonlint.com](https://jsonlint.com)) or
   run `python -m json.tool < your_file.json` in a terminal.
3. Fix any syntax errors (common issues: trailing commas, missing brackets, unquoted keys).
4. Re-submit.

---

### `docker exec` fails with "No such container"

**Symptom:** Running `winpty docker exec -it --user aimdrc aimdrc bash` returns
`Error: No such container: aimdrc`.

**Likely cause:** Docker Desktop is not running, or the `aimdrc` container has not been started.

**Resolution:**
1. Open Docker Desktop and wait for it to fully initialize.
2. Run `docker ps -a` to see if the `aimdrc` container exists but is stopped.
3. If stopped, run `docker start aimdrc` and retry.
4. If the container is absent entirely, contact Eric Walker or Harrison Park — the container
   may need to be recreated from the Docker image.

---

### `venv/bin/activate` not found inside container

**Symptom:** `. venv/bin/activate` fails after entering the container.

**Likely cause:** You may be in the wrong directory, or the virtual environment path differs
from what is documented.

**Resolution:**
1. Run `ls` inside the container to confirm the directory structure.
2. Look for a directory named `venv` or `.venv`.
3. If in doubt, contact Eric Walker.

---

### PyFAI integration output looks wrong (peaks at wrong q values)

**Symptom:** Integrated 1D patterns show peaks shifted from expected positions for a known
calibrant or sample.

**Likely cause:** Stale or incorrect calibration (`.poni` file), or the detector was moved
since the last calibration run.

**Resolution:**
1. Confirm the detector has not been moved since calibration.
2. Re-run sample calibration (see [Calibration](calibration.md)).
3. Verify the correct `.poni` file is being passed to the PyFAI integration step.

---

### Dagster pipeline run fails

**Symptom:** A Dagster run shows as failed in the UI.

**Resolution:**
1. Click into the failed run in the Dagster UI.
2. Click on the failed op to view the stack trace.
3. Common causes:
   - Missing input file (data streaming issue upstream)
   - Malformed `.h5` file (incomplete write — retry the scan)
   - PyFAI calibration failure (re-run calibration)
   - Container not running (check Docker Desktop)
4. Use **Re-execute from failure** in the UI to retry without re-running successful steps.

See [Dagster](software/dagster.md) for more detail on monitoring and re-runs.

---

## Escalation Path

| Severity | Description | Contact |
|----------|-------------|---------|
| Safety / radiation | Any suspected radiation exposure or interlock failure | Todd Hufnagel + JHU Radiation Safety immediately |
| Hardware fault | Instrument not responding, robot fault, detector issue | Proto support (support@protoxrd.com) |
| Software / control | AIMDRC API errors, Docker issues, pipeline failures | Eric Walker or Harrison Park |
| Data quality | Bad calibration, unexpected peak positions, zero intensities | Harrison Park or Joseph Nkansah-Mahaney |

---

## Changelog

| Version | Date | Author | Notes |
|---------|------|--------|-------|
| 1.0 | March 2025 | Harrison Park | Initial draft |
