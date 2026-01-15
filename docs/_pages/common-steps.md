# Common Steps for Any New Data

These are the steps to deploy new/updated election data or a new census.

NOTE: For simplicity and generality, these examples assume that these environment variables are set:

```bash
export DRA_REDIST="/path/to/Redist"
export DRA_DATA="/path/to/dra-data"
export DRA_VTD_DATA="/path/to/vtd_data"
export DRA_BLOCK_DATA="/path/to/block_data"
```

The example illustrated in the workflow below is for TN 2024 election data from RDH.
These additional bindings generalize the examples:

```bash
XX=CA
CYCLE=2020
```

## Step 1: Review New Election Data Results

There are several steps to getting ready to update development and production with new election data.


### Step 1.1: Figure Out Which `_data2.json` Files Are New or Updated

Using VS Code or the command line (`git status`), look at the `dra-data` repository to see which 
files like `*_data2.json` have been added or modified. 

For the TN 2024, there were two new files:

*   2020vt_2024Composite_block_47_data2.json.zip
*   2020vt_from_2024Elec_prec_block_47_data2.json.zip

one for the precinct-level data election data and one for the composite election.


### Step 1.2: Update the file list

New files need to be added to the `filelist2.json` file in the same directory, 
but don't include the `.zip` extensions in the entries there.


### Step 1.3: Delete the versions with 'old' in the filename.	

If the `new_election_data.py` script modifies any existing files, it first creates backup versions
with 'old' in the filename. You'll delete these before committing and pushing changes to GitHub.


### Step 1.4: Identify the Dataset Keys Affected

If you're updating existing election data, as opposed to adding new elections, you need to identify the internal dataset keys.
To do that, look at the elections in `*_elec_fulldisagg.log` in the Redist `aggs_working` directory.

If you don't know how to encode elections into dataset keys, you can get help translating elections into dataset keys by using 
this interactive script: `scripts/elections/dataset_keys.py`.

For the TN 2024 example below, these are the new/updated files and dataset keys:

```bash
FILES="2020vt_2024Composite_block_47_data2.json.zip 2020vt_from_2024Elec_prec_block_47_data2.json.zip"
KEYS="E24GPR E24GSE E24GCO C24GCO"
```

The keys were not needed for the TN 2024 workflow, because the elections were new. They are shown here for completeness.


## Step 2: Update Development Database

There are 3 steps to updating the development database with new/updated data.


### Step 2.1: Deploy New/Updated Datasets 

First, from from `dra-cli`, deploy the files that have changed.

For TN 2024, these were (note the `-d` option for development!):

```bash
./deploy.js -d -f "${DRA_DATA}/data/${XX}/${CYCLE}_VD/2020vt_2024Composite_block_47_data2.json.zip"
./deploy.js -d -f "${DRA_DATA}/data/${XX}/${CYCLE}_VD/2020vt_from_2024Elec_prec_block_47_data2.json.zip"
./deploy.js -d -f "${DRA_DATA}/data/${XX}/${CYCLE}_VD/filelist2.json"
```

There's a helper script to deploy all the new/updated files:

```bash
scripts/common/deploy-datasets.sh --state "${XX}" --dev --files ${FILES}
```

It also deploys the updated `filelist2.json` automatically.


## Step 2.2: Import Datasets 

Next, import the datasets into the development database.
Here you use `fn-all` instead of `dra-cli`. 

There are two scenarios: new data and updated data.

For TN 2024, the elections were all new, so the first command applies.
Use the second command for updated elections.

```bash
node testdist/importdataset.js -d -O -B -S "${XX}" -I
node testdist/importdataset.js -d -O -B -S "${XX}" -b ${KEYS}
```

It's a good practice to copy the Terminal output as a log for future reference.

There's a helper script to import all the new/updated datasets:

```bash
scripts/common/import-datasets.sh --state "${XX}" --dev
scripts/common/import-datasets.sh --state "${XX}" --dev --keys "${KEYS}"
```


### Step 2.3: Verify the Dataset Shows Up in Production DRA Locally

To make sure everything work as expected, run DRA locally with the development database.
You should see the new/updated data that you've added.

It can take 30-45 minutes or more for the updated files to show up.
If you need to push propagation through the system, you can close DRA, run this command:

```bash
node testdist/pubquery.js -f” 
```

in `fn-all` and re-open DRA in a new tab.


## Step 3: Update Production

Then repeat the steps for production.


### Step 3.1: Deploy New/Updated Files

From from `dra-cli`, deploy the files that have changed (note, no `-d` flag here).

```bash
./deploy.js    -f "${DRA_DATA}/data/${XX}/${CYCLE}_VD/2020vt_2024Composite_block_47_data2.json.zip"
./deploy.js    -f "${DRA_DATA}/data/${XX}/${CYCLE}_VD/2020vt_from_2024Elec_prec_block_47_data2.json.zip"
./deploy.js    -f "${DRA_DATA}/data/${XX}/${CYCLE}_VD/filelist2.json"
```

The helper script works here too without the `--dev` option:

```bash
scripts/common/deploy-datasets.sh --state "${XX}"       --files ${FILES}
```


## Step 3.2: Import Datasets 

Then, from `fn-all`, import the datasets into the production database.

Again, for the TN 2024 example, these were new elections, so the first command applies.
Use the second command for updated elections.

```bash
node testdist/importdataset.js    -O -B -S "${XX}" -I
node testdist/importdataset.js    -O -B -S "${XX}" -b ${KEYS}
```

Again, capturing the Terminal output as a log is a good practice.

The helper script works here too without the `--dev` option:

```bash
scripts/common/import-datasets.sh --state "${XX}"
scripts/common/import-datasets.sh --state "${XX}"      --keys "${KEYS}"
```


### Step 3.3: Verify the Dataset Shows Up in Production DRA Locally

As with development, run DRA locally with the production database to verify that the new/updated data shows up.
It can take 30-45 minutes or more for the updated files to show up.

If you need to push propagation through the system, you can close DRA, run this command:

```bash
node testdist/pubquery.js -f” 
```

in `fn-all` and re-open DRA in a new tab.


## Step 4: Update & Deploy Summary Files

After the election data itself is deployed and imported, you need to update the summary files that
describe the data available in DRA.


### Step 4.1: Update Summary Files

There are two summary files to update: `statedatasets2.json` and `whatsNew.json`.

For TN 2024, this is the `statedatasets2.json` entry:

```json
          "2024": {"Contests": ["Pres", "Sen", "Cong"], "attribute": "RDH",
                    "note": " ",
                    "urls": [{"url": "https://redistrictingdatahub.org/wp-content/uploads/2025/12/readme_tn_2024_gen_prec.txt", "label": "RDH Notes"}]},
```

that goes in the `2020_VD` section for TN.
This is a sample list of contests:

Here are the unique contest values:

* Pres
* Sen
* Gov
* AG

* Cong

* LtGov
* SoS
* Aud
* Treas, Treas (CFO), and Treas (Comptr)
* Comptr
* PubLands

* SuprCt Chief, SuprCt 1, SuprCt 2, SuprCt 3, SuprCt 4, SuprCt 5, SuprCt 6

* Del -- DC delegate
* Sen (shdw) -- DC shadow senator
* May -- DC mayor

* PVI

Add contests as needed.


For TN 2024, this is the addition to the top of `whatsNew.json`:

```json
{ "variant": "beginExpansion", "text": "December 18, 2025" },
  {
    "variant": "link", 
    "text": "We added 2024 election data for TN. Thanks to",
    "label": "Redistricting Data Hub.",
    "link": "https://redistrictingdatahub.org/data/about-our-data/"
  },
{ "variant": "endExpansion"},
```

TODO: If we can tolerate reformatting the two summary files, there is a helper script that 
can add entries to both summary files automatically:

```bash
scripts/common/update_summary_files.py \
--state TN \
--year 2024 \
--source "Redistricting Data Hub (RDH)" \
--contests Pres Sen Cong \
--readme https://redistrictingdatahub.org/wp-content/uploads/2025/12/readme_tn_2024_gen_prec.txt \
--about https://redistrictingdatahub.org/data/about-our-data/
```

TODO: What are the potential/valid sources (RDH, VEST, ...) for the summary files?


### Step 4.2: Deploy Summary Files

Then deploy the updated summary files to both development and production.

From dra-cli:

```bash
./deploy.js -d -f "${DRA_DATA}/data/statedatasets2.json"
./deploy.js -d -f "${DRA_DATA}/data/whatsNew.json"

./deploy.js    -f "${DRA_DATA}/data/statedatasets2.json"
./deploy.js    -f "${DRA_DATA}/data/whatsNew.json"

./copybucket.js -b data b2.dra-datafiles -k _statedatasets2.json
./copybucket.js -b data b2.dra-datafiles -k _whatsNew.json
```

There's a helper script to do all these commands at once:

```bash
scripts/common/deploy-summary-files.sh --state TN
```

It deploys `statedatasets2.json` and `whatsNew.json` to both development and production and copies them to Backblaze.


## Step 5: Commit and Push Changes to GitHub

Delete the 'old' backup files if you haven't already.

The final step is committing and pushing all the changes to the `data_tools` and `dra-data` repositories to GitHub.
You can do at the command line or using VS Code or another Git client.

As noted in [New & Updated Election Data](new-election-data.md), it's a good practice to create branches for your changes.
If you're working in a branch, you can simply commit and push your changes to GitHUb creating a pull request.
Then those changes can merged into `master` even if someone else has made changes in the meantime.


## Step 6: You're Done!

Typically we announce new election data on social media though we may batch announcements for multiple states.