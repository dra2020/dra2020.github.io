# Publishing CSV & GeoJSON Data

These are the steps to publish data our data to the public-facing repositories `vtd_data` and `block_data`.
As with new election data, it's a good idea to create branches in both repositories for the changes you're making.

The example below for the Tennessee 2024 data from RDH uses these bindings for generality:

```bash
XX=TN
CYCLE=2020
VERSION=v07
```

where `CYCLE` is the census cycle, not the year of an election.


## Step 1: Make a New GeoJSON

Before packaging and publishing the data files, you must produce a new `.geojson` file
that contains the data you're publishing.

From `dra-cli`, run these commands:

```bash
./newdata.js -p vtddatasets -s "${XX}"
./stat.js -b dra-us-west-datafiles -g "_${XX}_${CYCLE}_VD_tabblock.vtd.datasets.geojson"
```

The first command makes the new GeoJSON file with the data included and writes it to S3.
The second command downloads the new GeoJSON file to your local machine into a temp Redist directory.
Then copy and rename the GeoJSON file to the appropriate location for packaging and publishing, e.g.,
the `"${DRA_REDIST}/TempVtdArchives/${XX}/Geojson_${XX}.${VERSION}"` directory without the leading underscore.

There's a helper script to do all three commands at once:

```bash
scripts/publish/MAKE-GEOJSONS.sh --version "${VERSION}" --states "${XX}"
```


## Step 2: Prepare the Data Files

Then make the CSV files and package them and package the GeoJSON for publishing.

NOTE: An important decision here is what *new* data you're publishing. 
There are two subsets:

*   Demographic data (vtd_demo, block_demo)
*   Election data (vtd_elec, block_elec)

The GeoJSON files include both.
If only demographic data is being published, then only the `vtd_demo` and `block_demo` formats should be specified.
If only election data is being published, then only the `vtd_elec` and `block_elec` formats should be specified.
If both demographic and election data are being published, then all four formats should be specified.
In all cases, the `vtd_geojson` format should be specified to package the new data in a GeoJSON file.

```bash
scripts/publish/publish_data.py \
--states "${XX}" \
--formats vtd_demo,vtd_elec,block_demo,block_elec \
--version "${VERSION}"
```

```bash
scripts/publish/publish_data.py \
--states "${XX}" \
--formats vtd_geojson \
--version "${VERSION}"
```

As of November 7, 2025, use "v07" as the version.

NOTE: This can be done in one step by including all formats in a single command, 
but conceptually two different things are happening: in the first command, CSV files are created and packaged,
while in the second command, the already-created (in Step 1) GeoJSON file is just downloaded and packaged.

NOTE: The block data files are large--too big for GitHub without large file support--so they are written to the Redist directory.
The files are generated into the `TempBlockArchives/DeployArchives` subdirectory with the names like
`Demographic_Data_Block_TN.v07.zip` and `Election_Data_Block_TN.v07.zip`.


## Step 3: Deploy the Block Data to the CDN

Deploy the block data to the production CDN (Backblaze):

```bash
./stat.js -b b2.dra-block-data -p "${DRA_REDIST}/TempBlockArchives/DeployArchives/Demographic_Data_Block_${XX}.${VERSION}.zip" 
./stat.js -b b2.dra-block-data -p "${DRA_REDIST}/TempBlockArchives/DeployArchives/Election_Data_Block_${XX}.${VERSION}.zip" 
```

There's a helper script to do these:

```bash
scripts/publish/DEPLOY-BLOCK-DATA.sh --elections --demographic --version "${VERSION}" --states "${XX}"
```

It takes multiple states, if you're working on more than one at a time.


## Step 4: Update GitHub

The final step is committing and pushing all the changes to the `vtd_data` and `block_data` repositories to GitHub.
You can do at the command line or using VS Code or another Git client.

As noted above, it's a good practice to create branches for your changes.
If you're working in a branch, you can simply commit and push your changes to GitHUb creating a pull request.
Then those changes can merged into `master` even if someone else has made changes in the meantime.


## Step 5: You're Done!

Typically we announce new data on social media though we may batch announcements for multiple states.