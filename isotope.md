# Running HadCM3 with Isotopes

This example demonstrates how to enable stable water isotopes
(δ18O and δD) in HadCM3.

The example is based on:
- Step 1: Initialise isotope fields
- Step 2: Continue simulation with isotope tracers enabled

The required isotope modifications and STASH settings are already included in the example jobs.

Users should not modify these settings unless they understand the isotope implementation.

## Overview

Adding isotopes to an existing HadCM3 experiment requires:

1. Initialising isotope fields in atmosphere and ocean.
2. Restarting from the generated dump files.

The provided example jobs already contain all required isotope code modifications and STASH settings.

## Step 1 – Initialise isotope fields

Copy the example job:

- xqklc

Compile and run for one year.

## Step 2 – Restart from the isotope dump

Copy the example job:

- xqkld

### Required changes

#### Update start year

Increase the model start year by one.

#### Update dump files

Use the atmosphere and ocean dumps produced in Step 1.

## User modifications

After copying the example, users may modify:

- orbital parameters
- greenhouse gas concentrations
- land-sea mask
- vegetation
- ice sheets
- initial atmosphere dump
- initial ocean dump
- simulation length

Users should not modify:

- isotope STASH entries
- isotope Fortran mods
- isotope prognostic field settings

## Global salinity

The GLOBAL_SALINITY variable should be adjusted according to ice volume.

| Configuration | GLOBAL_SALINITY |
|--------------|----------------|
| No ice sheet | 34.23 |
| One polar ice sheet | 34.63 |
| Two polar ice sheets | 34.84 |

## Isotope diagnostics

The example includes isotope diagnostics for:

- precipitation δ18O
- precipitation δD
- cloud water isotopes
- runoff isotopes

For most applications no STASH modifications are required.
