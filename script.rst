Script
=====================

Presenting myself
--------------------

Who I am, where I come from?

- My name is Jorge Sanz, @xurxosanz
- I work at Prodevelop
- We do several things, but our main sector are ports

Real world(tm) is hard
------------------------

- On harbours we have draughtsmen, surveyors, civil engineers

  - High skills on technical drawings, surveying, etc
  - Almost 0 knowledge on GIS or databases
  - The live inside AutoCAD

    - http://www.flickr.com/photos/kingcounty/8139974275/
    - http://www.flickr.com/photos/kozmicdogz/5618334387/
    - http://www.flickr.com/photos/the-lobster/4014103606/

- But harbours want to use GIS

  - Management of their space (€€€)
  - Emergency management
  - Real time harbour knowledge (of facilities, staff, etc)
  - ...

- So they usually have needs for GIS

  - Need to load their CAD data into spatially enabled databases
  - Our daily fight (as a company) over the last 20 years

Real use case
-----------------------

- We have a space management application
- Cartography is maintained in DWG format
- We have to load that data into an Oracle database
- The draughtsman has to be able to load it as he does
  changes on the cartography

Requirements
-----------------------

- On demand execution
- From AutoCAD (those guys LIVE inside AutoCAD)
- They need to have a minimal feedback on the results
  of the process
- It has to be tolerant (at least do it's best) to
  the typical geometry problems.

Our approach
----------------------

- GeoKettle as an ETL tool to migrate this data
- Development of a simple AutoCAD tool that:

  - saves a copy of the current draw as DXF
  - starts a batch process that calls GeoKettle to load
    that DXF into an Oracle table
  - show some results



First part: AutoCAD
----------------------------

This is not really too interesting. It's just a
Visual Basic button that calls a DOS batch file
with all the parameters of the process (folders,
table name, etc) and that in the end, will call
the ``kitchen`` tool.

::

  CALL Kitchen.bat /level %LOG% /file "sincronizar-dxf-oracle.kjb" "/param:GPC_logfile=%LOGFILE%" "/param:GPC_logfile_detail=%LOGFILEDETAIL%" "/param:GPC_dxf_file=%DXF%" "/param:GPC_dxf_capa=%CAPA%" "/param:GPC_db_host=%DB_HOST%" "/param:GPC_db_port=%DB_PORT%" "/param:GPC_db_name=%DB_NAME%" "/param:GPC_db_user=%DB_USER%" "/param:GPC_db_pass=%DB_PASS%" "/param:GPC_tabla_tmp=%BP_TABLA%" "/param:GPC_tabla_tmp_geom=%BP_TABLA_GEOM%" "/param:GPC_tabla_tmp_idx=%BP_TABLA_INDEX%"

Second part: GeoKettle
------------------------------

What is GeoKettle?
+++++++++++++++++++++++++++++

- Kettle (now Pentaho Data Integration) is an ETL tool:

  - Extraction
  - Transformation
  - Load

- It has three user interfaces:

  - Spoon: Graphical User Interface to design and test
  - Kitchen: to call the processes from the command line (or in cron)
  - Carte: to deploy a web service to run the processes

- Building blocks at Kettle :

  - Step: minimum element of a process to perform several tasks like

    - Read data from an origin
    - Transform data or create new derivative data
    - Write the results into an output resource

  - Transformation: ETL process made by steps
  - Job: a work-flow of transformations or other jobs

> The Data Integration perspective of Spoon allows you to create two
> basic document types: transformations and jobs. Transformations are
> used to describe the data flows for ETL such as reading from a source,
> transforming data and loading it into a target location. Jobs are used
> to coordinate ETL activities such as defining the flow and
> dependencies for what order transformations should be run, or prepare
> for execution by checking conditions such as, "Is my source file
> available?," or "Does a table exist in my database?"


- GeoKettle:

  - Technical fork to add geographical capabilities
  - Multiplatform
  - LGPL
  - Integrating other geo tools like OGR/GDAL/SEXTANTE
  - It uses JTS and geotools internally
  - It has geo-steps to work with CSW catalogues, WPS,
    data analysis, SRS transformations, ...


Returning to our job:
++++++++++++++++++++++++++++++++++

Steps:

#. Delete the geometry field index on the table (Oracle Hack, SQL)
#. Load the DXF using OGR

  - filter by layer to avoid loading into GeoKettle unnecessary data

#. Filter geometries that aren't lines (JavaScript, JTS)
#. Polygonize the lines (JavaScirpt, JTS)
#. Filter fields
#. Load the result data into an Oracle table, by first removing previous data
#. Update the CRS of the table geometries (Oracle Hack, SQL)
#. Fix any geometry loaded with any of these errors (SQL):

   - The rings are not correctly oriented
   - There are duplicated nodes
   - ???

#. Create a new geometry index (Oracle Hack, SQL)

.. note:: A log is created to show to the user the results
          of the entire process

.. attention:: Paste an example of the log


Conclusions
---------------------------

- GeoKettle is a powerful tool to manipulate data
- It is a specialized desktop GIS to automate processes that will be executed many times
- It is mainly focused on users with knowledge on SQL, JavaScript, OGR/GDAL,...
- ...
