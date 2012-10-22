Report File Format
------------------

.. seealso::

  :doc:`tutorial-writing-a-report/index`

Reports in Indicia are defined by placing XML files in the **reports** folder on
the warehouse which define the SQL query to run against the database, the
input parameters and the output columns. Report files must be valid XML 
documents with the following element structure.

Element <report>
================

This is the top level element of the XML document.

Attributes
^^^^^^^^^^

* **title** - the display title of the report.
* **description** - the description of the report.

Child elements
^^^^^^^^^^^^^^

* :ref:`query <query-label>`
* :ref:`order-bys <order-bys-label>`
* :ref:`field-sql <field-sql-label>`
* :ref:`parameters <parameters-label>`
* :ref:`columns <columns-label>`
* :ref:`vagueDate <vaguedate-label>`

Example
^^^^^^^

.. code-block:: xml

  <report title="My report" description="A list of records">
    ...
  </report>

.. _query-label:

Element <query>
===============

There is a single element called query in the XML document, which should be a 
direct child of the report element. This contains the SQL statement to run, 
which may contain modifications and replacement tokens to allow it to integrate
with the reporting system, as described elsewhere.

Attributes
^^^^^^^^^^

There are several attributes available for the ``<query>`` element, which serve
to override the default field used when the warehouse adds certain filters to 
the query. For example, the **sharing** options for a report allow the report to
be filtered to a website or list of websites, or to the current user. The 
warehouse therefore will need to know the field name to use when inserting each
filter into the query.

* *website_filter_field* - field name, including the table alias, used for 
  identifying and filtering websites within the query. Defaults to "w.id" which
  is based on the assumption that the websites table is joined into the query
  with a table alias "w". 
* *created_by_field* identifies the field in the SQL query which is used to 
  filter for the current user when using the option **sharing=me**. Defaults
  to o.created_by_id which is based on the assumption that the occurrences table
  is joined into the query with a table alias "o".
* *samples_id_field* - identifies the field in the SQL query which is used to 
  join to the sample_attribute_values table in order to include sample custom
  attributes in the report output. Use in conjunction with the **smpattrs**
  datatype for a report parameter. Defaults to "s.id" which is based on the 
  assumption that the samples table is joined into the query with a table alias
  "s".
* *occurrences_id_field* - identifies the field in the SQL query which is used to 
  join to the occurrence_attribute_values table in order to include occurrence 
  custom attributes in the report output. Use in conjunction with the 
  **occattrs** datatype for a report parameter. Defaults to "o.id" which is 
  based on the assumption that the samples table is joined into the query with a
  table alias "o".
* *locations_id_field* - identifies the field in the SQL query which is used to 
  join to the location_attribute_values table in order to include location 
  custom attributes in the report output. Use in conjunction with the 
  **locattrs** datatype for a report parameter. Defaults to "l.id" which is 
  based on the assumption that the locations table is joined into the query with 
  a table alias "l".

Replacements Tokens
^^^^^^^^^^^^^^^^^^^

Within the SQL you include in the ``<query>`` element, you can insert the 
following tokens which will be replaced when the warehouse builds the query to
run:

* #columns# - replaced by a list of fields generated from the **sql** attributes
  of each ``<column>`` element in the ``<columns>`` section. For example, the 
  query could read ``select #columns# from taxa`` and there could be 2
  ``<column>`` definitions with the **sql** attribute set to "id" and "taxon"
  respectively, resulting in a query ``select id, taxon from taxa".
* #field_sql# - replaced by the contents of the ``<field_sql>`` element and used
  to separate the list of fields from the rest of the SQL statement, which 
  allows the warehouse to replace the field list with ``count(*)`` in order to 
  count the query results. If using #columns# then it is not necessary. See 
  :ref:`field-sql-label` for more information on using this replacement token. 
* #agreements_join# - if you are using the **sharing** parameter for the 
  reporting web service, then this replacement token specifies where in the 
  query that the warehouse will insert a join to the 
  **index_websites_website_agreements** table when needing to find the list of
  websites whose records can be included in the report output.
* #sharing_filter# - if you are using tbe **sharing** parameter for the 
  reporting web service, then this replacement token specifies where in the
  query's ``WHERE`` clause to insert any filter required for the sharing, e.g.
  this could be a filter on the occurrence **created_by_id** field when the 
  sharing mode is "me", or it could be a filter on the websites joined by the 
  **index_websites_website_agreements** table for other sharing modes which 
  allow records from other specific websites to be included in query output.
* #idlist# - when used in conjunction with the **idlist** datatype for a report
  parameter, this is replaced by a list of selected IDs to filter the report by
  as provided for the parameter. A typical use of the idlist is to allow a 
  report to integrate with a map featuring polygon based querying. Once the 
  polygon is drawn on the map and the contained points are found, the IDs of the
  points can be passed to the idlist parameter so that the grid filters to show
  just the points within the polygon. Therefore the idlist token should mark a
  position in the report ``WHERE`` clause which is suitable for the warehouse
  to insert SQL along the lines of ``AND o.id IN (1,2,3,4,5)``.
* #order_by# - When a report output is required in a particular sort order, e.g.
  after clicking on a column title in a grid to sort it, Indicia will append an 
  SQL ``ORDER BY`` clause to the end of the query. This token is only required 
  in the unusual circumstance that the clause needs to be inserted into the 
  query somewhere other than the very end of the report SQL, e.g. if it needs
  to precede a ``LIMIT`` statement. 

In addition any declared :ref:`parameters <parameters-label>` are available as 
replacement tokens, so if there is a parameter called "survey_id" then the
replacement token ``#survey_id#`` can be used in the report and it will be 
replaced by the selected survey ID when the report is run.

Example
^^^^^^^

.. code-block:: xml

  <query website_filter_field="o.website_id">
  SELECT #columns#
  FROM cache_occurrences o
  JOIN websites w on w.id=o.website_id 
  #agreements_join#
  #joins#
  WHERE #sharing_filter# 
  AND o.record_status not in ('I','T') AND (#ownData#=1 OR o.record_status not in ('D','R'))
  AND ('#searchArea#'='' OR st_intersects(o.public_geom, st_geomfromtext('#searchArea#',900913)))
  AND (#ownData#=0 OR CAST(o.created_by_id AS character varying)='#currentUser#')
  #idlist#
  </query>

.. _order-bys-label:

Element <order_bys>
===================

Contains elements defining the default sort order of the report. This can be
overriding by an ascending or descending sort on any column, e.g. when clicking
on a report grid title.

Child elements
--------------

* :ref:`order_by <order-by-label>`

.. _order-by-label:

Element <order_by>
===================

Contains the SQL for a single sort order field or comma separated group of 
fields, e.g. ``s.date_start ASC``.


.. _field-sql-label:

Element <field_sql>
===================

When the #field_sql# replacement token is used in the query, provide the SQL for
the list of fields in this element which will be replaced into the token when 
the query is run. The #field_sql# token should go immediately after the 
``SELECT`` keyword and before the ``FROM`` keyword to form a valid SQL statement
when it is replaced. This approach provides a quick way of allowing Indicia to 
perform a count of the records in a report without running the entire report
query. For a fully featured paginator to be shown for any report grids, Indicia
needs to know the total count of rows in the report result. Although this is 
achievable by simply loading the entire results of a query and counting rows, 
Indicia does not take this approach as it could lead to severe performance
impacts on the server for inefficient queries or large result sets. Using a 
``count(*)``  query is much faster.

Example
-------

.. code-block:: xml

  ...
  <query>SELECT #field_sql# FROM cache_occurrences</query>
  <field_sql>id, preferred_taxon_name, public_entered_sref</field_sql>
  ...

.. _parameters-label:

Element <parameters>
====================

Attributes
----------

* **name**
  The name of the attribute. Must consist of alphabetic characters,
  numbers and underscores only. The attribute is wrapped in hashes to create the
  replacement token which will be replaced in the query. For example, if the 
  parameter named "startdate" is set to 01/10/2012, then the report might 
  include a clause ``WHERE date>'#startdate#'`` which would be replaced when the
  report is run to form the SQL ``WHERE date>'01/10/2012'``.
* **display**
  The text used to label the parameter in the parameter request screen.
* **description**
  Gives a further description, and may be used in describing reports/parameters.
* **datatype**
  Used in determining the type of control to show when requesting the parameter. 
  Currently, the core module report interface supports datatypes 'text', 
  'lookup', 'date', 'geometry', 'polygon', 'line', 'point', 'idlist', 
  'smpattrs', 'occattrs', 'locattrs'. All other values default to text. Date 
  will show a datepicker control. Lookup will show a select box. Geometry, 
  Polygon, Line and Point all require a map for the user to draw the input 
  parameter shape onto. Finally, idlist, smpattrs, occattrs and locattrs are 
  special datatypes that are described below. When viewing the parameters form 
  in the Warehouse interface, the contents of the lookup are populated using the 
  query in the query attribute. When using the report_grid control in the 
  data_entry_helper class, the contents of the lookup are populated using the 
  population_call attribute. Alternatively a fixed set of values can be 
  specified by using the lookup_values attribute.
* **query** is used to provide an SQL query used to populate the select box for 
  lookup parameters. The query should return 2 fields, the key and display 
  value. This only works on the warehouse and does not work for reports run from
  client websites, since they cannot directly issue SQL queries, so it is 
  recommended that you use the **population_call** attribute instead.
* **population_call**
  Allows report parameter forms on client websites to populate the select boxes 
  when the datatype is lookup, for example when using the report_grid control. 
  The format is either direct:table name:id field:caption field, e.g. 
  "direct:survey:id:title" or report:report name:id field:caption field, e.g. 
  "report:myreport:id:title" where myreport must return fields named id and 
  title. At the moment additional parameters cannot be provided.
* **lookup_values** allows specification of a fixed list of values for a lookup 
  rather than one populated from the database. Specify each entry as key:value 
  with commas between them, for example "all:All,C:Complete,S:Sent for 
  verification,V:Verified".
* **linked_to**
  Available only for select parameters and allows another select to be specified
  as the parent. In this case, the values in this select are filtered using the 
  value in the parent select. For example, a select for survey might be linked 
  to a select for website, meaning that selecting a website repopulates the list 
  of available surveys.
* **linked_filter_field**
  Applies when using **linked_to**, and allows the filtered field in the entity 
  accessed by the population_call to be specified. In the above example of a 
  survey lookup linked to a website lookup, the survey lookup would specify this 
  as website_id.
* **emptyvalue** 
  Allows a special value to be used when the parameter is left 
  blank by the user. As an example, take an integer parameter, with SQL syntax 
  WHERE id=#id#. If the user leaves this parameter blank, then invalid SQL is 
  generated (WHERE id=). But, if emptyvalue='0' is specified in the parameter 
  definition, then the SQL generated will be WHERE id=0, which is valid and in 
  most cases will return no records. Consider replacing the SQL with ``WHERE 
  (id=#id# OR #id#=0)`` to create a filter that will return all records when 
  left blank.
* **fieldname**
  Use in conjunction with the **idlist** datatype. For more information see
  :ref:`idlist-label`
* **alias**
  Use in conjunction with the **idlist** datatype. For more information see
  :ref:`idlist-label`

.. _idlist-label:

More information on the idlist datatype
---------------------------------------

The **idlist** is a special datatype that will not add a control to the input 
form. Instead it provides a hidden input in the form which other code on the 
page can use to filter the report. An example of the use of this field is when 
using the report_map control linked with a report_grid so that clicking on the 
map passes a comma separated list of occurrence IDs into the hidden input, then 
reloads the report grid. In order for this to work it is necessary to provide 2 
additional attributes of the parameter alongside the datatype="idlist". These 
are **fieldname** which defines the name of the field in the SQL (including 
table alias if necessary) and **alias** which is the aliased fieldname that is 
output by the query. The former is used when constructing the SQL report query, 
the latter is used when retrieving the ids to filter against from the report 
output. So, in a simplified report example which includes this SQL:

.. code-block:: sql

  SELECT o.id as occurrence_id FROM occurrences
  WHERE o.deleted=false
  #idlist#

you would expect a parameter defined like:

.. code-block:: xml

  <param name='idlist' display='List of IDs' description='Comma separated list of occurrence IDs to filter to.' datatype='idlist' fieldname='o.id' alias='occurrence_id' />

Parameters which require additional joins
-----------------------------------------

.. todo:: 

  complete this section

Optional custom attributes
--------------------------

.. todo::

  complete this section

.. _columns-label:

Element <columns>
=================

The ``<columns>`` element provides an area within the report definition to list
output columns and provide configuration for each column. A report which lists
the columns directly in the ``<query>`` element's SQL statement does not need
to specify the columns here to work, although the flexibility of the report is
greatly increased if columns are specified.

Child elements
--------------

* :ref:`column <column-label>`

.. _column-label:

Element <column>
================

Provides the definition of a single output column for the report query.

Attributes
----------

* **name**
  Should match the name used in the query:

  * ``SELECT foo FROM websites`` should have name *foo*
  * ``SELECT bar AS baz`` FROM websites should have name *baz* (not *bar*)
  * ``SELECT w.foo FROM websites`` should have name foo, not w.foo, though where 
    there is ambiguity renaming your columns with 'AS' is the recommended 
    solution. Failing to match this correctly may leave phantom columns in the 
    report.

* **display**
  Will be displayed as the column header.
* **style** 
  Provides CSS which will be applied to the column of the output HTML table 
  (though not the header).
* **class**
  Defines a css class that will be applied to the body cells in the column.
  For example, in a species column you can specify "sci binomial" to define that 
  this is the name part of the row. This can then be detected as a `Species 
  Microformat <http://microformats.org/wiki/species>`_.
* **visible** can be set to false to hide a column.
* **img** can be set to true for a field that contains the filename of an image 
  uploaded to the Warehouse. This will then be replaced by a thumbnail of the 
  image, with support for FancyBox image popups to show the full image size. 
  Multiple images can be comma separated in the field output to output mutiple
  thumbnails.
* **mappable** can be set to true to declare a column which can then be output 
  using the ``report_helper::report_map`` method. The column must output a 
  `WKT <http://en.wikipedia.org/wiki/Well-known_text>`_ definition of the 
  geometry to be mapped, e.g. the column definition in the SQL might be 
  ``st_astext(geom)``.
* **orderby** can be set to the name of another column in the report (including 
  hidden columns) when a column that is logically selected for sorting 
  physically uses another column to provide the sort order. For example terms in 
  Indicia termlists support a sort_order field which gives an optional non-
  alphabetical sort order for the list of terms (good, better, best is an 
  example of a non-alphabetical but logical sort order). By specifying 
  ``orderby="sort_order"`` for the term column, this causes the logical rather
  than alphabetical sort to be used when clicking on this column's header.
* **datatype** can be used to declare the datatype of a column to enable column 
  filtering in the grid. Set to one of text, date, integer or float. When set,
  a text box is shown at the top of the column into which the user can type
  filters.
* **aggregate**
  Described in the section :ref:`declaring-column-sql-label` below.
* **distincton**
  Described in the section :ref:`declaring-column-sql-label` below.
* **in_count** 
  Described in the section :ref:`declaring-column-sql-label` below.
* **feature_style** can be used when there is a mappable column on the report, 
  to define a column which provides the value for one of the map styling 
  parameters supported in OpenLayers. Supported options include **strokeColor** 
  (a CSS colour specification, e.g. '#00FF00'), **strokeOpacity** (a number from 
  0 to 1), **strokeWidth** (number of pixels wide to draw the perimeter line), 
  **strokeDashStyle** (dot, dash, dashdot, longdash, longdashdot or solid), 
  **fillColor** (as strokeColor), fillOpacity (as strokeOpacity). For example, a
  report could vary the opacity of output grid references on the map according 
  to size by including this column in the SQL:

  .. code-block:: sql

    length(s.entered_sref) / 24.0 as fillopacity,

  This column then has a definition:

  .. code-block:: xml

    <column name='fillopacity' visible='false' feature_style="fillOpacity"  />

.. _declaring-column-sql-label:

Declaring SQL for each column
-----------------------------

.. todo::
 
  complete this section

Element <vagueDate>
===================

By default, vague dates provided as a **date_start**, **date_end** and 
**date_type** field in the report output columns are processed to result in a 
single **date** column containing the vague date as a readable string. It is 
possible to override this behaviour and leave the original columns in place, by 
adding the following element to the ``<report>`` element in the xml:

.. code-block:: xml

  <vagueDate enableProcessing="false" />

When vague date processing is enabled, as an example your query might output the 
following table:

=================  ===============  ================
sample_date_start  sample_date_end  sample_date_type
=================  ===============  ================
2011-12-14	       2011-12-14	      D
2010-01-01	       2011-12-31	      Y
=================  ===============  ================

This would be output as:

===================  =================  ==================  ===========
*sample_date_start*  *sample_date_end*  *sample_date_type*  sample_date
===================  =================  ==================  ===========
2011-12-14           2011-12-14         D                   14/12/2011
2010-01-01           2010-12-31         Y                   2010
===================  =================  ==================  ===========

Note that the columns with titles in italics are not visible in the output grid,
though the data is returned in the dataset so is accessible. 