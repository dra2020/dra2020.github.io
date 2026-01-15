---
layout: page
title: Data Tools
permalink: _data-tools/
---

The tools here handle several different scenarios in our data pipeline workflow, including:

*   Ingesting new election datasets from partners like RDH and VEST -- This process takes an input file,
    census shapes & data in a local 'Redist' directory and our `dra-data` repository, and produces new and
    updated election data files in `dra-data`. From there, they are deployed into the app and our CDN (Backblaze).
    The process of ingesting new/updated election data is described in detail in 
    [New & Updated Election Data](/docs/new-election-data.md).

*   Publishing our data -- This process takes the dataset files in `dra-data`, packages them up, and
    adds them to our publc-facing data repositories (`vtd_data` and `block_data`).
    The process of publishing data is described in detail in 
    [Publishing CSV & GeoJSON Data](/docs/publishing-data.md).

*   Ingesting new ACS estimates from the Census Bureau -- TODO: Describe this process.

*   Ingesting a new decennial census -- TODO: Describe this process.

### Questions / Issues to Resolve

*   TODO: I'm fuzzing on when `importdataset.js` should be run with or without keys.

    ```
    From Terry:
    If you specify -B (builtin) then you can optionally also specify a set of regular expressions to match against to filter the import to just those datasets (using the -b flag). If you do not specify the -b flag, the default is to (re-)import all (as well as importing any new datasets) that are encountered in the file(s).
    So for a given state XX, "node testdist/importdataset.js -s XX -O -B" will reimport or newly import all builtin datasets in the files specified in filelist2.json for that state. -s can take multiple states as well.

    Just to clarify on the arguments, when processing builtin datasets (-B flag specified), the code processes all the builtin files and will:

    - only import NEW builtin datasets if the -I flag is specified. So these are datasets that were never imported before.
    - will only import builtin datasets that match one of the patterns specified in the -b argument(s). These can be only NEW (if -I was specified) or also existing (if -I was not specified).
    - if the -b flag is omitted, all builtin datasets match (but -I will still restrict to only NEW)
    If a dataset that previously existed is re-imported, it overwrites the previous contents. This is different from the normal behavior of importdataset, which will always create new datasets.

    ...

    both -I and -b flags are just optimizations to avoid re-importing the exact same datasets. If you leave them off, we just re-import all builtin datasets for that state (and presumably most of them are unchanged).

    The "scenario" for -I is that you have just added a new dataset and want it to show up in the app. So this is the first time the dataset is showing up in the app. [TODO: Does 'dataset' here mean 'election'?]

    The "scenario" for -b is that you have updated the underlying data for some existing dataset and want to re-import the new information (e.g. replacing NYT data with RDH data but the builtin dataset key is the same).
    ```

    What is a "builtin" dataset? (Do you mean vs. custom datasets?)

*   TODO: There are some operational questions to resolve wrto how one gets the files that need to be in this directory structure.
    This is the sharing / common Redist issue.

*   TODO: I need to better understand caching in `aggs_working` and caching in `Temp*` directories.
    I don't know when I can or when I should / shouldn't delete those directories.
    I got bit at least once by something cached causing work I thought I was doing to not have an effect.
*   TODO: Is there a caching issue when creating a new GeoJSON for publishing?

*   TODO: If we can tolerate reformatting the two summary JSON files, there is a helper script that can add 
    entries to both summary files automatically.

*   TODO: What are the potential/valid sources (RDH, VEST, ...) for the summary files?

*   TODO: Add notes wrto restricted datasets.

*   TODO: What's the fourth state that uses BG's instead of VTD's: CA, OR, WV, and ???
*   TODO: What, if anything, is different for new/updated election data for these states?

*   TODO: Do the common data steps apply to new ACS data?
