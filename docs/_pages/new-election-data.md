# New & Updated Election Data

These are the steps to take a new or updated election dataset and map it to existing geometry. 

The example illustrated in the workflow below is for TN 2024 from RDH.


## Step 1: Prerequisites / Setup

There are a few things you need to have in place before you can process new election data.


### Step 1.1: Get the Census Block Shapes and Data

These two files need to be in the `{DRA_REDIST}/Census/${CYCLE}/${XX}` directory for the state and
redistricting cycle. For example, for XX=CA and CYCLE=2020:

* 2020 census block shapes -- `tl_2020_06_tabblock20.zip`
* 2020 census data -- `2020Census_block_06.json`

Dave Bradlee has the census data files.
The block shapes are available from the Census Bureau. Note, however, that the constituent files of the 
block shapefile need to be at the top level of the zip file, not in a subdirectory.
Frequently, the zip file from the Census Bureau has a subdirectory with the same name as the .shp file, 
so you may need to unzip it and re-zip it properly.

NOTE: This step would not be necessary, if `Redist` were a repository that could be cloned.


### Step 1.2: Create Branches in `data_tools` and `dra-data`

In the process of adding new election data, you'll be making changes to both the `data_tools` and `dra-data` repositories.

Creating branches in them is not required, but that makes it easier for more than one person to work
on the data pipeline at the same time as it simplifies the merge process.

So, for this example you could create `tn-2024-rdh` branches in both repositories.


## Step 2: Copy the Election File to `Redist`

An election data partner (like RDH or VEST) packages elections in a zipped shapefile or geojson.
The first step is putting that file in the right directory.

For example, the `tn_2024_gen_prec.zip` file from RDH for updated TN 2024 data goes in:

`{DRA_REDIST}/Census/TN/2024/tn_2024_gen_prec.zip`

where `{DRA_REDIST}` is the environment variable that points to your Redist root directory.


## Step 3: Determine the Name Inside the Zip File

The next step is determining the name of the shapefile or geojson inside the zip file.
You can unzip it manually or you can list the contents like this:

```bash
unzip -l tn_2024_gen_prec.zip
```

If there are multiple shapefiles or geojson files in the zip file, choose the "all" one.


## Step 4: Update `sourcepaths.py`

Edit `data_tools/base/sourcepaths.py` to add entries to `get_source_shapes_zip_path()` and `get_source_shapes_path()`.
These point to the zip file and the shapefile/geojson inside the zip, respectively.

For example, the entry for TN 2024 data for the former is:

```Python
    elif (year == 2024 and state == "TN"): # Modified for TN 2024 RDH data
        source_shapes_zip_path = zip_root + "tn_2024_gen_prec.zip"
```

... and for the latter it is:

```Python
    elif state == "TN" and year == 2024: # Modified for TN 2024 RDH data
        source_geo_path = source_shapes_path + "tn_2024_gen_all_prec/tn_2024_gen_all_prec.shp"
```

NOTE: If the path *inside* the zip contains directories, include those directories, as shown.


## Step 5: Update `filter_prop_key()`

Edit `disaggregate.py/filter_prop_key()` to add the dataset. 
This function does two things: it filters (selects) which fields to keep from the source data, and
maps their names from source field names to our canonical ones.

For most 2024 data from RDH and VEST, the only change that you need to make is to add the state to the list of states
in this block:

```Python
    elif ( ... ):
        # Mostly RDH States; also VEST
```

To see what elections are in the dataset, run this script in log mode (`--step 1`):

```bash
scripts/elections/new_election_data.py \
--years 2024 \
--states TN \
--step 1 \
--pattern vest
```

where the `--pattern` flag identifies the pattern of fields in the data, not the actual source of the data.
So here the `vest` pattern is used for RDH data as well.
The list of valid patterns is list in the script itself.
They correspond to different "handlers" for the `convert()` function in `convert_data_file.py`.

The resulting log file -- `output1_3_2024to2020_elec_listprops.log` -- will list the elections found in the data.
Some elections won't be selected by the normal code, and you don't need to worry about those.
Sometimes, however, there is an election that we'd typically include that we don't want for some reason.
The congressional elections in CA 2024 RDH were an example of that:
several contests were Dem vs Dem, so we didn't want to include that.


## Step 6: Run `new_election_data.py` for Real

With `filter_prop_key()` updated, now run the new election data script for real (steps 2 and 3).
For the RDH TN 2024 data, this looks like:

```bash
scripts/elections/new_election_data.py \
--years 2024 \
--states TN \
--step 2 \
--pattern vest
```

Note: If the dataset is restricted, like some VEST data is, use the `--restricted` flag.

After Step 2, check the new log in the `agg_working` directory -- `output1_3_2024to2020_elec_fulldisagg.log`
-- for errors. Do a basic sanity check to make sure votes weren't dropped.

At the bottom of this log you'll see a report of totals like this:

```
Props totals
{   'G24CONDVAR': 977870,
    'G24CONIOTH': 64394,
    'G24CONRVAR': 1884691,
    'G24PREDVAR': 1056265,
    'G24PREIOTH': 40812,
    'G24PRERVAR': 1966865,
    'G24USSDVAR': 1027461,
    'G24USSIOTH': 61404,
    'G24USSRVAR': 1918743}
```

From this you can glean the elections that were processed (kept) and the total votes for each.
Here the elections are Congress (CON), President (PRE), and US Senate (USS), all for the 2024 general election.
You may need that list down the line.

Step 3 reads files in the `agg_working` directory and files in the `dra-data/data/TN/2020_VD` directory.
After running it, check the `makecomposite{yyyy}.log` file for errors.
Again, you're doing a basic sanity check.

This produces new or updated `_data2.json` files in the `dra-data/data/TN/2020_VD` directory, like
`2020vt_from_2024Elec_prec_block_47_data2.json.zip` for the TN 2024 elections, and 
`2020vt_2024Composite_block_47_data2.json.zip` for the 2024 composite election.

To see which dataset keys are in these files, you can unzip them and use `jq`, e.g.,

```bash
cat 2020vt_from_2024Elec_prec_block_47_data2.json | jq '.datasets | keys'
```

NOTE: When we get to the 2030 redistricting cycle, the directory will be `TN/2030_VD` instead of `TN/2020_VD`.


## Step 7: Do the Common Steps for New Data

Once you've successfully completed the previous steps, the new/updated election data is in the `dra-data` repository
but not yet in the app.
To get it in the app, you have to complete the [Common Steps for Any New Data](common-steps.md).
