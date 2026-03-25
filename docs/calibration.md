# Calibration

**Relevant to:** All operators running experiments on MAXIMA  
**Last Updated:** March 2025  
**Author:** Harrison Park  
**Prerequisites:** [Startup Procedure](startup.md) must be completed before calibration.

---

## Overview

MAXIMA requires two distinct types of calibration before data collection:

1. **Machine calibration** — aligns and stabilizes the X-ray source (MetalJet)
2. **Sample calibration** — establishes the sample-to-detector geometry using known calibrant
   materials placed on the sample holder

Both are required when starting fresh for the day. Sample calibration is also required whenever
the detector distance is changed or the sample holder is repositioned.

---

## Machine Calibration

### When to Perform

Machine calibration of the **Excillum MetalJet E1+** source is required:

- **At the start of every day**, before any X-ray operation
- **After any extended idle period** where X-rays have been off for several hours or more

### Procedure

Machine calibration is performed through the **Excillum software**, not through the AIMDRC
controller. The source self-calibrates once triggered.

1. Open the **Excillum** software on the control PC.
2. Initiate the calibration sequence (see Excillum software documentation for the exact menu path).
3. Monitor progress in the Excillum interface. Calibration typically completes in **1–5 minutes**.
4. Do not operate the X-ray shutter or begin experiments until calibration is confirmed complete.

> **TODO:** Add screenshot of Excillum calibration screen and exact menu navigation steps.

---

## Sample (Detector Geometry) Calibration

### Purpose

Sample calibration uses alumina powder calibrants to determine the beam center position on the
detector and the accurate sample-to-detector distance. This information is used by PyFAI during
azimuthal integration to convert 2D diffraction patterns into 1D intensity profiles. Without
accurate calibration, all downstream `q` or `2θ` values will be incorrect.

The calibration workflow in the automated data pipeline performs:
- Correction for detector tilt
- Beam center location
- Sample-to-detector distance calibration

### Calibrant Materials and Positions

Alumina (Al₂O₃) calibrant pucks are placed at four cardinal positions on the sample holder.
Their coordinates in the instrument coordinate system are:

| Position | Coordinates `[x, y]` |
|----------|----------------------|
| North    | `[6, 31]`            |
| West     | `[-23, 0]`           |
| South    | `[6, -32]`           |
| East     | `[35, 0]`            |

> **Coordinate convention:** Negative Y values are toward the lab bench (opposite side from
> Joseph's office). See [Sample Handling](sample_handling.md) for the full coordinate system
> description.

### Procedure

1. Confirm calibrant pucks are seated correctly at all four cardinal positions on the sample holder.
2. Run calibration scans on all four calibrant positions using the controller API.
3. The automated Dagster pipeline will pick up the resulting `.h5` files and run the PyFAI
   calibration workflow automatically. See [Dagster](software/dagster.md) for details.
4. Verify the output calibration parameters (beam center, detector distance, tilt) are
   reasonable before proceeding to sample data collection.

> **TODO:** Add example calibration script and expected output parameter ranges.

### Manual Calibration with PyFAI

If the automated pipeline is unavailable, calibration can be performed manually using PyFAI's
`pyFAI-calib2` GUI or the recalibration notebook:

```
https://www.silx.org/doc/pyFAI/dev/usage/tutorial/Recalib/Recalib_notebook.html
```

See [PyFAI](software/pyfai.md) for installation and usage details.

---

## Re-Calibration Triggers

Re-run sample calibration when any of the following occur:

- The detector is moved to a new distance along the translation stage
- The sample holder is removed and replaced
- Data quality or peak positions appear systematically off
- After a power cycle (note: power cycling can cause a slight timestamp shift — see
  [Troubleshooting](../troubleshooting.md))

---

## Changelog

| Version | Date | Author | Notes |
|---------|------|--------|-------|
| 1.0 | March 2025 | Harrison Park | Initial draft |
