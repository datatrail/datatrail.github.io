---
layout: post
title: py2cbs - a Python wrapper for CBS Open Data 
comments: false
---

## py2cbs - a Python wrapper for CBS Open Data  

[Statline](http://opendata.cbs.nl/dataportaal/portal.html) is the electronic databank of [CBS (Statistics Netherlands)](http://www.cbs.nl/en-GB/menu/home/default.htm?Languageswitch=on). It enables users to compile their own tables and graphs. The information can be accessed, printed and downloaded free of charge.

StatLine is made available as Open Data via the [ODATA](http://www.odata.org) protocol. CBS published a <a href="http://www.cbs.nl/NR/rdonlyres/2561A2B7-CE51-47B9-A838-9968EF67FFB0/0/2014handleidingcbsopendataservices.pdf" target="_blank">tutorial</a> describing the Data Services provided and a <a href="http://www.cbs.nl/NR/rdonlyres/1C647C35-54E2-48A9-8FC0-BC851FB84E88/0/2014handleidingpowerpivotmetcbsopendatav2.pdf" target="_blank">tutorial</a> how to retrieve data using PowerPivot for Excel. 

The Data Services can also be consumed by other programs or languages. This post gives an introduction to py2cbs, a client library for working with CBS (Netherlands Statistics) Open Data from within Python applications and from the command line. The <a href="https://github.com/datatrail/py2cbs" target="_blank">code</a> and <a href="https://github.com/datatrail/py2cbs" target="_blank">documentation</a> is available on GitHub. A <a href="http://cbsod-catalog.herokuapp.com/" target="_blank">demo</a> of a web application build with Flask and py2cbs is hosted on Heroku.

<br>
<h4 class="alert alert-info">OData</h4>

OData (Open Data Protocol) is an OASIS standard that defines the best practice for building and consuming RESTful APIs. OData services are web services that expose some resources. The service document is a static resource that lists all of the top-level entity collections exposed by the service. Collections, also known as feeds, are composed of a number of items, known as entries. Each entry  contains properties, that contain the actual data. Following table shows a simplified mapping between OData resources and a relational database:
<table class="display nowrap table-striped " cellspacing="0">
  <thead>
    <tr>
      <th>OData</th>
      <th>Relational database</th>
   </tr>
  </thead>
  <tbody>
    <tr>
      <td>Service</td>
      <td>Database</td>
    </tr>
    <tr>
      <td>Collection/Feed</td>
      <td>Table</td>
    </tr>
    <tr>
      <td>Entry</td>
      <td>Row</td>
    </tr>
    <tr>
      <td>Property</td>
      <td>Column value</td>
    </tr>
  </tbody>
</table>

The resources (service documents, collections, entries, etc.) can be accessed via Uniform Resource Identifiers (URLs), for example <a href="http://services.odata.org/V4/Northwind/Northwind.svc/Customers?$top=10" target="_blank">http://services.odata.org/V4/Northwind/Northwind.svc/Customers?$top=10</a>. A URL used by an OData service has at most three significant parts: 
<ul>
  <li>service root, the URL that identifies the service document (http://services.odata.org/V4/Northwind/Northwind.svc/)</li>
  <li>resource path, the name of the resource, for example a collection (Customers)</li>
  <li>query options: options defining the output returned ($top=10)</li> 
</ul>
<p>
  <img class="img-responsive displayed" src="/static/img/ODataUri.png" width="70%">
</p>
OData supports two formats for representing the resources: XML-based Atom format and JSON format. The output format can be specified by adding a query option $format=json or $format=atom to the URL.
<br>

<br>
<h4 class="alert alert-info">CBS Open Data Services</h4>

CBS provides three data services to access the Open Data: Catalog, Api and Feed.

__Catalog:__ 

The service document of the catalog data service can be accessed via the service root ([http://opendata.cbs.nl/ODataCatalog/](http://opendata.cbs.nl/ODataCatalog/)) and returns a list of collections/feeds in Atom-XML:


~~~XML
<?xml version="1.0" encoding="UTF-8"?>
<service xmlns="http://www.w3.org/2007/app" xmlns:Atom="http://www.w3.org/2005/Atom" xml:base="http://opendata.cbs.nl/ODataCatalog/">
   <workspace>
      <Atom:title>Default</Atom:title>
      <collection href="Featured">
         <Atom:title>Featured</Atom:title>
      </collection>
      <collection href="Table_Featured">
         <Atom:title>Table_Featured</Atom:title>
      </collection>
      <collection href="Tables">
         <Atom:title>Tables</Atom:title>
      </collection>
      <collection href="Themes">
         <Atom:title>Themes</Atom:title>
      </collection>
      <collection href="Tables_Themes">
         <Atom:title>Tables_Themes</Atom:title>
      </collection>
   </workspace>
</service>  
~~~

<br>
Data is stored as tables in the CBS Open Data portal, tables are categorized by themes. The collections contain information about the themes, tables and the relationship between the two. 
Collections, also known as feeds, can be accessed by adding resource path (= name of the collection) to the service root (= the URL of the catalog service document).  
<br>
For example: the feed Tables_Themes ([http://opendata.cbs.nl/ODataCatalog/Tables_Themes](http://opendata.cbs.nl/ODataCatalog/Tables_Themes)\) returns information about the relation between tables and themes in Atom-XML:

~~~XML
<?xml version="1.0" encoding="utf-8"?>
<feed xml:base="http://opendata.cbs.nl/ODataCatalog/"
    xmlns="http://www.w3.org/2005/Atom"
    xmlns:d="http://schemas.microsoft.com/ado/2007/08/dataservices"
    xmlns:m="http://schemas.microsoft.com/ado/2007/08/dataservices/metadata">
    <id>http://opendata.cbs.nl/ODataCatalog/Tables_Themes</id>
    <title type="text">Tables_Themes</title>
    <updated>2015-12-22T10:58:12Z</updated>
    <link rel="self" title="Tables_Themes" href="Tables_Themes" />
    <entry>
        <id>http://opendata.cbs.nl/ODataCatalog/Tables_Themes(0)</id>
        <category term="Cbs.OData.Catalog.Table_Theme" scheme="http://schemas.microsoft.com/ado/2007/08/dataservices/scheme" />
        <link rel="edit" title="Table_Theme" href="Tables_Themes(0)" />
        <title />
        <updated>2015-12-22T10:58:12Z</updated>
        <author>
            <name />
        </author>
        <content type="application/xml">
            <m:properties>
                <d:ID m:type="Edm.Int32">0</d:ID>
                <d:TableID m:type="Edm.Int32">0</d:TableID>
                <d:ThemeID m:type="Edm.Int32">4</d:ThemeID>
            </m:properties>
        </content>
    </entry>
    ....
~~~

<br>
__Api and Feed:__


<p>
  The API and Feed data service give access to collections containing information about the metadata and data of a table in the Open Data portal.   The Feed data service can be used for large datasets (>10.000 records) and returns XML-based Atom, the API data service can be used for smaller datasets (&lt; 10.000 records) and returns JSON.
</p>
  Each table has a unique identifier. The service root of a table is a combination of the URL of the data service and the unique identifier of the table. For example: the Api data service for table with identifier 81162eng can be accessed via URL <a href="http://opendata.cbs.nl/ODataApi/OData/81162eng" target="_blank">http://opendata.cbs.nl/ODataApi/OData/81162eng</a> and returns a list of colections in JSON-format:
</p>

~~~JSON
{
  "odata.metadata":"http://opendata.cbs.nl/ODataApi/OData/81162eng/$metadata","value":[
    {
      "name":"TableInfos","url":"http://opendata.cbs.nl/ODataApi/OData/81162eng/TableInfos"
    },{
      "name":"UntypedDataSet","url":"http://opendata.cbs.nl/ODataApi/OData/81162eng/UntypedDataSet"
    },{
      "name":"TypedDataSet","url":"http://opendata.cbs.nl/ODataApi/OData/81162eng/TypedDataSet"
    },{
      "name":"DataProperties","url":"http://opendata.cbs.nl/ODataApi/OData/81162eng/DataProperties"
    },{
      "name":"SectorBranchSIC2008","url":"http://opendata.cbs.nl/ODataApi/OData/81162eng/SectorBranchSIC2008"
    },{
      "name":"Periods","url":"http://opendata.cbs.nl/ODataApi/OData/81162eng/Periods"
    }
  ]
}
~~~

<p>
  The Feed data service for the same table can be accessed via the URL <a href="http://opendata.cbs.nl/ODataFeed/OData/81162eng" target="_blank">http://opendata.cbs.nl/ODataFeed/OData/81162eng</a> and returns the output as Atom-XML:
</p>

~~~XML
<?xml version="1.0" encoding="utf-8"?>
<service xml:base="http://opendata.cbs.nl/ODataFeed/OData/81162eng" xmlns="http://www.w3.org/2007/app" xmlns:atom="http://www.w3.org/2005/Atom">
  <workspace>
    <atom:title type="text">Default</atom:title>
    <collection href="http://opendata.cbs.nl/ODataFeed/OData/81162eng/TableInfos">
      <atom:title type="text">TableInfos</atom:title>
    </collection>
    <collection href="http://opendata.cbs.nl/ODataFeed/OData/81162eng/UntypedDataSet">
      <atom:title type="text">UntypedDataSet</atom:title>
    </collection>
    <collection href="http://opendata.cbs.nl/ODataFeed/OData/81162eng/TypedDataSet">
      <atom:title type="text">TypedDataSet</atom:title>
    </collection>
    <collection href="http://opendata.cbs.nl/ODataFeed/OData/81162eng/DataProperties">
      <atom:title type="text">DataProperties</atom:title>
    </collection>
    <collection href="http://opendata.cbs.nl/ODataFeed/OData/81162eng/SectorBranchSIC2008">
      <atom:title type="text">SectorBranchSIC2008</atom:title>
    </collection>
    <collection href="http://opendata.cbs.nl/ODataFeed/OData/81162eng/Periods">
      <atom:title type="text">Periods</atom:title>
    </collection>
  </workspace>
</service>
~~~

<p>
  More information about the data services can be in read in a <a href="http://www.cbs.nl/NR/rdonlyres/2561A2B7-CE51-47B9-A838-9968EF67FFB0/0/2014handleidingcbsopendataservices.pdf" target="_blank">tutorial</a> (dutch) published by CBS.</p>

<br>
<h4 class="alert alert-info">py2cbs - a Python wrapper for CBS Open Data</h4>
<p>
  Py2cbs is a client library for working with CBS (Netherlands Statistics) Open Data from within Python applications and from the command line.
</p>
<p>
  This introduction shows how to get connected to the CBS Open Data portal using py2cbs and how to carry out some basic operations. The two most important classes are <b>CatalogTree</b> and <b>Table</b>. The simplest way to try to connect to the CBS Open Data portal is via the Python console. First we import the CatalogTree and Table class from py2cbs.
</p>

~~~Python
>>> from py2cbs import CatalogTree, Table
~~~

<p>
  <br>
  <b>CatalogTree</b>
  <br>
  Catalog is a wrapper around the Catalog data service and provides access to the catalog of the Open Data portal. The Catalog data service gives access to five collections that can be divided into two groups:
  <ul>
    <li>Featured, Table_Featured: information about featured themes and tables</li>
    <li>Tables, Themes, Tables_Themes: information about the themes, tables and the relationship between the two</li>
  </ul>

  CatalogTree is a subclass of Catalog and only gives access to the latter three collections. CatalogTree offers some extra functionality to deal with the hierarchical structure of the themes and tables.
</p>


<p>
  Next we create an instance bound to the catalog of the CBS Open Data portal. The catalog is available in two languages: dutch ('nl') and english ('en'). Once the instance is created the collections / feeds can be loaded into the instance of the CatalogTree:
</p>

~~~Python
>>> ct = CatalogTree(language = 'en')
>>> ct.set_feeds()
~~~

<p>
  Now the CatalogTree is loaded, we can perform some basic operations to retrieve information from the CatalogTree and its collections:
  <ul>
    <li>show the URL of the service document and the list of collections available in the CatalogTree</li>
    <li>show some basic information for a specific collection or feed (for example the feed Themes): URL, number of entries, property names, primary key (unique identifier of an entry) and the property values of the first entry</li>
    <li>perform a search query on one of the feeds, for example search all tables that belong to the theme identified by ThemeID 1028</li>
  </ul>
</p>

~~~Python
>>> ct.url
'http://opendata.cbs.nl/ODataCatalog'
>>> print ct.collections
[u'Tables', u'Themes', u'Tables_Themes']

>>> themes = ct.feeds['Themes']
>>> themes.url
u"http://opendata.cbs.nl/ODataCatalog/Themes?$filter=Language eq 'en'"
>>> print 'Number of entries in Themes:', len(ct.get_entries('Themes'))
Number of entries in Themes: 167
>>> print 'Property names:', themes.property_names
Property names: [u'ID', u'ParentID', u'Number', u'Title', u'Language', u'Catalog']
>>> print 'Primary key:', themes.primary_key
Primary key: ID
>>> print 'Property values first entry:', ct.get_entries('Themes')[0].values()
Property values first entry: [934, None, u'5350', u'Agriculture', u'en', u'CBS']

>>> record_set = catalog.query('Tables_Themes',{'ThemeID':1028}, ['TableID'])
>>> for record in record_set:
...     identifier = catalog.get_property('Tables', record['TableID'], 'Identifier')
...     title = catalog.get_property('Tables', record['TableID'], 'Title')
...     print 'identifier, title: {0}, {1}'.format(identifier, title)
... 
identifier, title: 7413eng, Residential construction; stock, changes and progress, region, 1988 - 2011
identifier, title: 81885ENG, House Price Index by region; existing own homes; 2010 = 100
identifier, title: 37263eng, Changes in the dwelling stock; 1995-2011
 

~~~

<p>
  Examples of functionality available in CatalogTree but not in Catalog are functions to retrieve the parents and the children of a theme, to help navigating the hierarchical tree structure of tables and themes:
</p>

~~~Python
>>> parents = ct.get_parents(1097) 
>>> for parent_id in parents:
...     print ct.get_property('Themes', parent_id, 'Title')
Archive
Regional statistics

>>> children = ct.get_children(1097)
>>> for child in children['Tables_Themes']:
...     print ct.get_property('Tables', child['TableID'], 'Title')
Consumer prices; rent increase for dwellings by region
House Price Index by type of dwelling, region; existing own homes;1995-2012
Bankruptcies; monthly figures by legal form and region, Jan.1993-April 2014
Economic totals per region, 1995-2001
..
>>> for child in children['Themes']:
...     print ct.get_property('Themes', child['ID'], 'Title')
Macro-economics

~~~  

<p>
  <br>
  <b>Table</b>
  <br>
  Table is a wrapper around the Feed data service and provides access to a table in the Open Data portal. The Feed data service gives access to four or more collections that can be divided into two groups:
  <ul>
    <li>TableInfos, DataProperties, Dimensions (optional, for example Periods): metadata about the table, data and dimensions</li>
    <li>UntypedDataSet, TypedDataSet: the data itself</li>
  </ul>
</p>
<p>
  To create a Table instance we need to specify the identifier of the table, for example 81162eng. Once the instance is created, the collections or feeds can can be loaded into the instance of the Table:
</p>

~~~Python
>>> table = Table('81162eng')
>>> table.set_feeds()
~~~

We can now perform the same basic operations on the Table-object, just as we for the CatalogTree:
  <ul>
    <li>show the URL of the service document and the list of collections available in the Table</li>
    <li>show some basic information for a specific collection or feed (for example the feed DataProperties): URL, number of entries, property names, primary key (unique identifier of an entry) and the property values of the first entry</li>
  </ul>

~~~Python
>>> table.url
'http://opendata.cbs.nl/ODataFeed/odata/81162eng'
>>> print table.collections
[u'TableInfos', u'UntypedDataSet', u'TypedDataSet', u'DataProperties', u'SectorBranchSIC2008', u'Periods']

>>> data_properties = table.feeds['DataProperties']
>>> data_properties.url
u'http://opendata.cbs.nl/ODataFeed/odata/81162eng/DataProperties'
>>> print 'Number of entries in DataProperties:', len(table.get_entries('DataProperties'))
Number of entries in DataProeprties: 40
>>> print 'Property names:', data_properties.property_names
Property names: [u'odata.type', u'ID', u'Position', u'ParentID', u'Type', u'Key', u'Title', u'Description']
>>> print 'Primary key:', data_properties.primary_key
Primary key: ID
>>> print 'Property values first entry:', table.get_entries('DataProperties')[0].values()
Property values first entry: [u'Cbs.OData.Dimension', 0, 0, None, u'Dimension', u'SectorBranchSIC2008', u'Sector/branch (SIC 2008)', None]

~~~

<p>
  More specific functionality of Table is to retrieve information about the variables and the dimensions in a table, and the data itself:
</p>

~~~Python
>>> print 'variables dataset = ', table.get_variables_dataset()
variables =  {'labels': ['ID', u'Sector/branch (SIC 2008)', u'Periods', u'Employee', .. ], 
               'names': ['ID', u'SectorBranchSIC2008', u'Periods', u'Employee_1', .. ]}

>>> print 'dimensions dataset = ', table.get_dimensions_dataset()
dimensions =  [{'labels': [u'016 Support activities for agriculture'], 'values': [u'303600'], 'name': u'SectorBranchSIC2008'}, .. ]

>>> print 'typed dataset = ', table.get_typed_dataset()
typed data set =  [[0, u'303600', u'2009JJ00', 19.1, 36.1, 15.2, 27.4, 3355.0, 3260.0, 95.0, 2856.0, 1138.0, 674.0, 455.0, 113.0, 72.0, 41.0, 10.0, 96.0, 721.0, 116.0, 74.0, 210.0, 113.0, 22.0, 18.0, 56.0, 112.0, 324.0, 499.0, -66.0, 1.0, 23.0, 457.0], .. ]

~~~

More information on how to use py2cbs can be found in the <a href="https://github.com/datatrail/py2cbs">documentation</a> and in the source code on <a href="https://github.com/datatrail/py2cbs">GitHub</a>. 

<br>
<h4>Resources:</h4>
<p>
  <hr>
  <ul>
    <li><a href="http://opendata.cbs.nl/dataportaal/portal.html?_la=en&_catalog=CBS">CBS Open Data Statline</a></li>
    <li><a href="http://www.cbs.nl/NR/rdonlyres/2561A2B7-CE51-47B9-A838-9968EF67FFB0/0/2014handleidingcbsopendataservices.pdf">CBS Open Data Services: Handleiding</a></li>
    <li><a href="http://www.odata.org/">OData - the best way to REST</a></li>
    <li><a href="http://www.odata.org/documentation/odata-version-3-0/">OData Version 3.0</a></li>
    <li><a href="http://www.cbs.nl/NR/rdonlyres/1C647C35-54E2-48A9-8FC0-BC851FB84E88/0/2014handleidingpowerpivotmetcbsopendatav2.pdf">PowerPivot gebruik met CBS Open Data: Handleiding</a></li>
  </ul>

</p>
<br><br>