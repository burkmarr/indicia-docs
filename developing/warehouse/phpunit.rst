Running phpUnit tests
=====================

The Indicia warehouse included phpUnit test classes for some key areas of functionality.
These provided automated tests ensuring that the code remains robust whilst changes are
implemented.

#. To run phpUnit tests, you must first `install phpUnit 
   <http://phpunit.de/manual/current/en/index.html>`_. Currentlt phpUnit versions up to 5.x 
   are supported.
#. You MUST install your warehouse against a local database called 'indicia' and the 
   warehouse MUST only be used for unit testing, since each test run may clear existing
   data from the database.
#. Enable the following modules by adding them to the list in 
   `application/config/config.php`:
   
     * rest_api
     * sref_mtb
     * data_cleaner_period_within_year
     
#. Enable the phpUnit warehouse module. You will need to disable it after running your
   tests. Whilst the phpUnit module is enabled, your warehouse will not be fully 
   functional so do not run tests on a live server. :doc:`Click here for more information 
   on enabling modules <../../administrating/warehouse/modules/index>`.
#. Start a command prompt or terminal window and navigate to the root folder of your 
   warehouse installation.
#. Enter the following command. The bootstrap option tells the phpUnit framework to load
   the Kohana framework::
  
     phpUnit --bootstrap=index.php application/tests
     
The output will tell you how many tests ran, how many assertions ran and how many failed.
It also gives time and memory usage information. 

The last parameter in the command given above is the folder to run the tests from. You can 
also run tests for warehouse modules by pointing to the tests folder within that module,
e.g.::

  phpUnit --bootstrap=index.php modules/indicia_svc_data/tests
  
Or you can run tests against a single class::

  phpUnit --bootstrap=index.php application/tests/helpers/vague_dateTest.php
  
More information on writing tests is available on `the phpUnit website 
<http://phpunit.de/manual/current/en/writing-tests-for-phpunit.html>`_. 

Continuous Integration
----------------------

The Indicia warehouse is automatically rebuilt and tested each time a code change is 
committed into GitHub. The output of these builds including test results can be viewed
`on Travis CI <https://travis-ci.org/Indicia-Team/warehouse>`_.
