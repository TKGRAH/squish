# squish
GISC 4011K class project

## Database Setup/Schema
After the loader has finished, the database will contain 30 different tables.  The breakdown of those tables is as follows:
  + blckgroup_1990 - polygon information for the 1990 block groups in the census data
  + blckgroup_2000 - polygon information for the 2000 block groups in the census data
  + blckgroup_2010 - polygon information for the 2010 block groups in the census data
  + block_1990 - polygon information for the 1990 blocks in the census data
  + block_2000 - polygon information for the 2000 blocks in the census data
  + block_2010 - polygon information for the 2010 blocks in the census data
  + catchments - polygon information for the catchments in the NHD Plus data
  + codebook - table that lists some of the column titles in the census data and explains their meanings
  + county_1990 - polygon information for the 1990 counties in the census data
  + county_1990_pop - tabular information for the 1990 county populations in the census data
  + county_2000 - polygon information for the 2000 counties in the census data
  + county_2010 - polygon information for the 2010 counties in the census data
  + gages - point information for the gages in the NHD Plus data
  + huc - polygon information for the HUC-8, HUC-10, and HUC-12 in one table for the NHD data
  + nhd_flowlines - line information for streams in the NHD data
  + nhd_waterbodies - polygon information for the waterbodies in the NHD data
  + nhdp_flowlines - line information for streams in the NHD Plus data
  + nhdp_waterbodies - polygon information for the waterbodies in the NHD Plus data
  + nlcd2001 - raster information for the 2001 NLCD
  + nlcd2006 - raster information for the 2006 NLCD
  + nlcd2011 - raster information for the 2011 NLCD
  + spatial_ref_sys - table that lists all spatial referenced available in the creation of a spatial database
  + tabulation_huc - tabulation data calculated from the huc table compared to the NLCD tables land cover codes
  + tabulation_huc_generalized - tabulation data calculated from the huc table compared to the NLCD tables land cover types
  + tabulation_tract - tabulation data calculated from the tract tables compared to the NLCD tables land cover codes
  + tabulation_tract_generalized - tabulation data calculated from the tract tables compared to the NLCD tables land cover types
  + tract_1990 - polygon information for the 1990 tracts in the census data
  + tract_1990_pop - tabular information for the 1990 tracts in the census data
  + tract_2000 - polygon information for the 2000 tracts in the census data
  + tract_2010 - polygon information for the 2010 tracts in the census data
# H1 Transportation Documentation
	Source Summary
  o	Our data came exclusively from the Government’s Census Bureau Topologically Integrated Geographic Encoding and Referencing Road         database.
  o	[TIGER Site](https://www.census.gov/geo/maps-data/data/tiger.html)
	Data Structure
  o	13 Counties that are included in our area of study, the Upper Chattahoochee watershed
    	Cherokee
    	Cobb
    	Dawson
    	Dekalb
    	Forsyth
    	Fulton
    	Gwinnett
    	Habersham
    	Hall
    	Lumpkin
    	Towns
    	Union
    	White
  o	2 Sets of the data by year
    	2011
    	2016
  o	What does it include?
    	The Tiger road data is a line shape file that includes the street name, road type and the state/county’s ID.
    	Contains Primary and Secondary road data
	Spatial Information
  o	North American Datum of 1983
  o	Uses this Geographical Coordinate system for all data
	Python Loader
  o	We edited the loader to work for transportation data
    	### Import Census Tracts to Database ###
    roads_2011 = 'data\\transportation_2011.shp'
    roads_2016 = 'data\\transportation_2016.shp'
    	cursor.execute("""CREATE EXTENSION IF NOT EXISTS pgrouting;""")
    	Set the correct spatial reference system for our data (Code too long to post)
    	shp2pgsql_command = " ".join([shp2pgsql_path, "-c", "-s 94269", '-W latin1', os.path.join(BASE_DIR, roads_2011), "roads2011"])
  o	Upon running, it crashed and did not work
    	This is due to the size of the data, using a smaller data set proved to work fine
	QGIS /pgRouting Procedure
  1.	Open PostGIS Shapefile and DBF loader 2.2
  2.	Set the SRID as 4269 and import the road data to the database
  3.	Create postgis and pgrouting extension to the database. 
    	CREATE EXTENSION postgis;
    	CREATE EXTENSION pgrouting;
  4.	Add ‘source’ and ‘target’ column and create topology for the road data
    	ALTER TABLE roaddata ADD COLUMN 'source' integer;
    	ALTER TABLE roaddata ADD COLUMN 'target' integer;
    	SELECT pgr_createTopology('roaddata', 0.00001, 'geom', 'gid');
  5.	Fails to create topology
  6.	Attempt pgRouting via QGIS pgRouting extension tested on two random points and it fails to return a value
	PSQL Procedure
  1.	Zip the road data and upload to dropbox 
  2.	Dl link: https://www.dropbox.com/s/ji9y05ae92y5f2y/2016%2B2011.zip?dl=0
  3.	Go to the console window on the Vultr machine and log in
  4.	In the command line, install the “unzip” apt
  5.	cmd: sudo apt-get install unzip
  6.	Install the “postgis” apt
  7.	cmd: sudo apt-get install postgis
  8.	Download the road data from dropbox
  9.	cmd: wget “https://www.dropbox.com/s/ji9y05ae92y5f2y/2016%2B2011.zip?dl=0”
  10.	Unzip the road data
  11.	cmd: unzip 2016+2011.zip?dl=0 –d /home
  12.	Create an output folder and name it “output_20XX”
  13.	cmd: cd /home
      cd /home/2011
      mkdir output_2011
  14.	Change directory to the folder where the .shp file locate (hint. Use “ls”)
  15.	Use “shp2pgsql” tool that is initially installed with pgrouting to convert .shp file to a table (.sql file)
  16.	cmd: shp2pgsql tl_2011_13057_roads.shp cherokee > /home/2011/output_2011/Cherokee_2011_13057_roads.sql
  17.	Change directory back to root
  18.	Now, we are going to import the .sql file into the database
  19.	cmd: su postgres
  20.	In pgadmin III, create a new database (In our case, we named it “project”)
  21.	In the command line, type:
      psql
      /connect project
      create extension postgis;
      create extension pgrouting;
  22.	Import the .sql file to the database
  23.	cmd: psql –h 45.32.210.174 –d project –U postgres –f /home/2011/output_2011/Cherokee_2011_13057_roads.sql
	ArcMap Procedure
  1.	Open the Road Data up into ArcMap(One county at a time)
  2.	Run the “Rebuild Connectivity” tool
  3.	Fill out the pop up window accordingly for the dataset(Spatial Reference System, buffer size, etc)
  4.	Run the tool, it finishes successfully
  5.	Go back to QGIS and run pgRouting on the corrected shapefile, it still fails
  6.	Test multiple other parameters in the “Rebuild Connectivity” tool, continues to fail after each attempt
	Georgia Clearing House Procedure(Successful Attempt)
  1.	Download Data from Georgia Clearing House
  2.	Open QGIS
  3.	Import the road data and export that by eliminating the Z-dimension (Elevation) and M-dimension(Route) data.
  4.	Open PostGIS Shapefile and DBF loader 2.2 in pgAdminIII
  5.	Set the SRID as 4269 and import the road data to the database
  6.	Create postgis and pgrouting extension to the database. 
    	CREATE EXTENSION postgis;
    	CREATE EXTENSION pgrouting;
  7.	Add ‘source’ and ‘target’ column and create topology for the road data
    	ALTER TABLE roaddata ADD COLUMN 'source' integer;
    	ALTER TABLE roaddata ADD COLUMN 'target' integer;
    	SELECT pgr_createTopology('roaddata', 0.00001, 'geom', 'gid');
  8.	Run the query that returns a set of pgr_costResult (seq, id1, id2, cost) rows, that make up a path
  9.	In order to visualize the result, we can display the path in QGIS. Open QGIS and connect to the Postgres database
  10.	Open the DB Manager and open the SQL window
  11.	Run the query and export the visual representation in QGIS
      SELECT seq, id1 AS node,id2 AS edge, cost, geom FROM pgr_dijkstra('
         SELECT gid AS id, 
                source, 
                target, 
                minute::float8 AS cost
         FROM roaddata',
      117232,
      90731,
      false,
      false) as route JOIN roaddata on roaddata.gid = route.id2
	Conclusion
  o	For the future, don’t use Tiger data for any pgRouting or Network Analysis, the data set is fragmented and does not work properly,       even after spatial correction.
    	Fragmentation refers to the issue that the line features that represent the roads within the shapefile do not connect with one         another at a vertex where a connection should be present.
    	Georgia Clearing House data did not have this issue and worked correctly using the exact same procedure. 
      •	[Georgia Clearing House](https://data.georgiaspatial.org/login.asp)
