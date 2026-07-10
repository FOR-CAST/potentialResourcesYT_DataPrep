potentialResourcesYT_DataPrep Manual
================
Last updated: 2026-07-10

- [potentialResourcesYT_DataPrep
  Module](#potentialresourcesyt_dataprep-module)
  - [Authors:](#authors)
  - [Module Overview](#module-overview)
    - [Module summary](#module-summary)
    - [Module inputs and parameters](#module-inputs-and-parameters)
    - [Events](#events)
    - [Plotting](#plotting)
    - [Saving](#saving)
    - [Module outputs](#module-outputs)
    - [Links to other modules](#links-to-other-modules)
    - [Getting help](#getting-help)

# potentialResourcesYT_DataPrep Module

[![made-with-Markdown](figures/markdownBadge.png)](https://commonmark.org)

#### Authors:

Tati Micheletti <tati.micheletti@gmail.com> \[aut, cre\]

## Module Overview

### Module summary

This is a data-preparation module that harmonizes anthropogenic
disturbance datasets – specifically mining, oil/gas, seismic lines, and
cutblocks – into standardized “potential” layers, where higher values
flag the places most likely to see future development first.

The approach is idiosyncratic and NOT generalizable, but can serve as a
basis for other development types. It was originally developed for the
Northwest Territories by Tati Micheletti; this is the FOR-CAST fork used
in the Yukon Northern Mountain Caribou project. The built-in defaults
still reference the original NWT sample data (union of BCR6 and NT1);
the pipeline supplies Yukon inputs.

### Module inputs and parameters

The module expects three inputs: `disturbanceList` (from
`anthroDisturbance_DataPrep`), the study area (`studyArea`), and the
matching raster (`rasterToMatch`). In `disturbanceList`, the outer list
names are `dataName` and the inner list names are `dataClass` from
`disturbanceDT`.

The full list of module inputs:

| objectName | objectClass | desc | sourceURL |
|:---|:---|:---|:---|
| disturbanceList | list | List (general category) of lists (specific class) needed for generating disturbances. This last list contains: Outter list names: dataName from disturbanceDTInner list names: dataClass from disturbanceDT, which is a unique class after harmozining, except for any potential resources that need idiosyncratic processing. This means that each combination of dataName and dataClass (except for ‘potential’) will only have only one element. | <https://drive.google.com/file/d/1v7MpENdhspkWxHPZMlmx9UPCGFYGbbYm/view?usp=sharing> |
| studyArea | SpatVector | Study area to which the module should be constrained to. Defaults to NT1+BCR6. Object can be of class ‘vect’ from terra package | <https://zenodo.org/records/20434361/files/NT1_BCR6.zip> |
| rasterToMatch | SpatRaster | All spatial outputs will be reprojected and resampled to it. Defaults to NT1+BCR6. Object can be of class ‘rast’ from terra package | <https://zenodo.org/records/20434240/files/RTM_BCR6_NT1.tif> |

The main parameter is `whatToCombine`, a `data.table` naming the
`dataName` / `dataClasses` from `disturbanceList` to combine, plus
columns identifying how to filter active processes: for mining,
`CLAIM_STAT` (potential exploration) and `PERMIT_STA` (permits that may
become claims), with claims the most likely values; for oil/gas, the
potential layer (`C2H4_BCR6_NT1`) provides values from which structures
are placed highest-first, using exploration permits as a starting point.
(These column names belong to the NWT sample data schema.)

| paramName | paramClass | default | min | max | paramDesc |
|:---|:---|:---|:---|:---|:---|
| whatToCombine | data.table | c(“oilGa…. | NA | NA | Here the user should specify a data.table with the dataName and dataClasses from the object `disturbanceList` anthroDisturbance_DataPrep, first and (input from second levels) to be combined. The table also contains a column identifying which to be used to filter active processes for mining (CLAIM_STAT and PERMIT_STA) For Oil/Gas (it needs to identify which layer is the potential one (C2H4_BCR6_NT1) and which is used to constrain where oil and gas will be added. For oil and gas, the other potential layer (exploration permits) is used as a starting point to add structures, followed by randomly placing them in the highest values of C2H4_BCR6_NT1and going down until the total amount is reached. For mining, CLAIM_STAT is the potential exploration, while PERMIT_STA are the ones that might become CLAIMS. The most likely values are CLAIMS and followed by PERMITS. |
| .plots | character | screen | NA | NA | Used by Plots function, which can be optionally used here |
| .plotInitialTime | numeric | 0 | NA | NA | Describes the simulation time at which the first plot event should occur. |
| .plotInterval | numeric | NA | NA | NA | Describes the simulation time interval between plot events. |
| .saveInitialTime | numeric | NA | NA | NA | Describes the simulation time at which the first save event should occur. |
| .saveInterval | numeric | NA | NA | NA | This describes the simulation time interval between save events. |
| .seed | list |  | NA | NA | Named list of seeds to use for each event (names). |
| .useCache | logical | FALSE | NA | NA | Should caching of events or module be used? |
| allowPre2011 | logical | FALSE | NA | NA | If TRUE, allows simulations whose start time is before 2011. Intended for specialised validation runs where pre-2011 baselines are available. |

### Events

After `init`, the module runs five events. Each works on the current and
potential disturbance layers for a resource type, unifying them and
setting the highest values as the locations to be developed first:

- `createPotentialMining`;
- `createPotentialOilGas`;
- `createPotentialSeismicLines`;
- `createPotentialCutblocks` – reproduces the steps used by ENR (J.
  Hodson) to build the potential forest layer (details in
  `makePotentialCutblocks()`);
- `replaceInDisturbanceList` – writes the harmonized potential layers
  back into `disturbanceList`.

### Plotting

This module does not schedule any plot events.

### Saving

This module does not schedule any save events.

### Module outputs

| objectName | objectClass | desc |
|:---|:---|:---|
| disturbanceList | list | List (general category) of lists (specific class) needed for generating disturbances. This is a modified input, where we replace multiple potential layers (i.e., mining and oilGas) by only one layer with the highestvalues being the ones that need to be filled with new developments first, or prepare potential layers (i.e., potentialCutblocks). |
| potentialOilGas | list | List (general category) of lists (specific class) holding the harmonized oil/gas potential layer, where higher values mark the locations most likely to be developed first (the starting point for adding oil and gas structures downstream). |

`disturbanceList` is the modified input, with multiple potential layers
(e.g., mining and oilGas) replaced by a single prioritized layer;
`potentialOilGas` is the harmonized oil/gas potential layer used
downstream as the starting point for placing new oil and gas structures.

### Links to other modules

Part of the anthropogenic-disturbance module collection: run after
`anthroDisturbance_DataPrep` and before `anthroDisturbance_Generator`.
The collection combines with landscape-simulation modules (e.g.,
`Biomass_core`) and caribou modules to improve realism in forecasts.

### Getting help

- <https://github.com/FOR-CAST/potentialResourcesYT_DataPrep/issues>
