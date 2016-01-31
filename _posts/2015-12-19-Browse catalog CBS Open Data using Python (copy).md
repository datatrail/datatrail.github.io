---
layout: post
title: Using Python to access CBS Open Data - py2cbs 
comments: false
---

## Using Python to access CBS Open Data: py2cbs 

[Statline](http://opendata.cbs.nl/dataportaal/portal.html) is the electronic databank of [CBS (Statistics Netherlands)](http://www.cbs.nl/en-GB/menu/home/default.htm?Languageswitch=on). It enables users to compile their own tables and graphs. The information can be accessed, printed and downloaded free of charge.

Statistics Netherlands makes StatLine available as Open Data via [ODATA](http://www.odata.org). CBS published a <a href="http://www.cbs.nl/NR/rdonlyres/1C647C35-54E2-48A9-8FC0-BC851FB84E88/0/2014handleidingpowerpivotmetcbsopendatav2.pdf" target="_blank">tutorial</a> how to consume data using PowerPivot (Excel). This post describes py2cbs, a small Python library that can be used CBS Open Data. The library enables users to create their own Python application using CBS Open Data. The code is available on <a href="" target="_blank">GitHub</a>. 


### OData

OData (Open Data Protocol) is an OASIS standard that defines the best practice for building and consuming RESTful APIs. OData services are web services that expose some resources (Collections, Entries, Links, etc). The resources can be accessed via URL. OData supports two formats for representing the resources: the XML-based Atom format and the JSON format. Atom is an XML-based document format that describes Collections of related information known as "feeds". A Service Document lists the available Collections. Feeds are composed of a number of items, known as Entries. 

### CBS Open Data Services

CBS provides three web services to access the Open Data: Catalog, Api and Feed.

*__Catalog web service__*

The catalog web service can be accessed via [http://opendata.cbs.nl/ODataCatalog/](http://opendata.cbs.nl/ODataCatalog/) and returns Atom-xml. The Service Document list the available Collections. 

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

Data is stored as tables in the CBS Open Data portal, tables are categorized by themes. The collections contain information about the themes, tables and the relationship between the two. 
Each Collection or feed is composed of a number of items, known as Entries. A feed can be accessed by concatenating the catalog URL and the title of the collection, for example: feed Tables_Themes ([http://opendata.cbs.nl/ODataCatalog/Tables_Themes](http://opendata.cbs.nl/ODataCatalog/Tables_Themes)\) returns:

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
    <entry>
    ....
~~~

<br>
*__Api and Feed web service__*


<p>
  Each table has a unique identifier, e.g. 81162eng. Based on the identifier, the <a href="http://opendata.cbs.nl/ODataApi/OData/81162eng" target="_blank">API</a> and <a href="http://opendata.cbs.nl/ODataFeed/OData/81162eng" target="_blank">Feed</a> data service give access to collections containing information about the metadata and data of the table: 
  <ul>
    <li>TableInfos (<a href="http://opendata.cbs.nl/ODataApi/OData/81162eng/TableInfos" target="_blank">API</a>, <a href="http://opendata.cbs.nl/ODataFeed/OData/81162eng/TableInfos" target="_blank">Feed</a>)</li>
    <li>DataProperties (<a href="http://opendata.cbs.nl/ODataApi/OData/81162eng/DataProperties" target="_blank">API</a>, <a href="http://opendata.cbs.nl/ODataFeed/OData/81162eng/DataProperties" target="_blank">Feed</a>)</li>
    <li>Dimensions (e.g. periods <a href="http://opendata.cbs.nl/ODataApi/OData/81162eng/Periods" target="_blank">API</a>, <a href="http://opendata.cbs.nl/ODataFeed/OData/81162eng/Periods" target="_blank">Feed</a>)</li>
    <li>UntypedDataSet (<a href="http://opendata.cbs.nl/ODataApi/OData/81162eng/UntypedDataSet" target="_blank">API</a>, <a href="http://opendata.cbs.nl/ODataFeed/OData/81162eng/UntypedDataSet" target="_blank">Feed</a>)</li>
    <li>TypedDataSet (<a href="http://opendata.cbs.nl/ODataApi/OData/81162eng/TypedDataSet" target="_blank">API</a>, <a href="http://opendata.cbs.nl/ODataFeed/OData/81162eng/TypedDataSet" target="_blank">Feed</a>)</li>
  </ul>
  <br>
  The Feed data service can be used for large datasets (>10.000 records) and returns XML-based Atom, the API data service can be used for smaller datasets (<10.000 records) and returns JSON.
</p>

<p>
  More information about the data services can be in read in a <a href="http://www.cbs.nl/NR/rdonlyres/2561A2B7-CE51-47B9-A838-9968EF67FFB0/0/2014handleidingcbsopendataservices.pdf" target="_blank">tutorial</a> (dutch) published by CBS.</p>

<br>
<h4 class="alert alert-info">How to access the data services using Python</h4>
<p>
  OData supports two formats for representing the resources (Collections, Entries, Links, etc) it exposes: the <a href="http://www.odata.org/documentation/odata-version-2-0/Atom-format/">XML-based AtomPub</a> format and the <a href= "http://www.odata.org/documentation/odata-version-2-0/json-format/">JSON format</a>.
  <br>

  The Catalog and Feed data service return XML-based Atom, the Api data service returns JSON.
</p>

<br>
<h4>XML-based Atom</h4>
<p>As described in Atom <a href="http://tools.ietf.org/html/rfc4287">[RFC4287]</a>, Atom is an XML-based document format that describes Collections of related information known as "feeds". 
</p>

<p>
  <i>Example collections / feeds exposed by catalog data service: <a href="http://opendata.cbs.nl/ODataCatalog/" target="_blank">http://opendata.cbs.nl/ODataCatalog/</a></i>
</p>
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
<p>
  Feeds are composed of a number of items, known as Entries. In OData, Entries are represented as Atom elements with all the Properties of the Entry represented as elements within the element which is a direct child of the element. 
</p>

<p>
  <i>Example Entries & Properties exposed by catalog data service: <a href="http://opendata.cbs.nl/ODataCatalog/Themes" target="_blank">http://opendata.cbs.nl/ODataCatalog/Themes</a></i>
</p>

    <?xml version="1.0" encoding="UTF-8"?>
    <feed xmlns="http://www.w3.org/2005/Atom" xmlns:d="http://schemas.microsoft.com/ado/2007/08/dataservices" xmlns:m="http://schemas.microsoft.com/ado/2007/08/dataservices/metadata" xml:base="http://opendata.cbs.nl/ODataCatalog/">
       <id>http://opendata.cbs.nl/ODataCatalog/Themes</id>
       <title type="text">Themes</title>
       <updated>2015-12-19T06:52:04Z</updated>
       <link rel="self" title="Themes" href="Themes" />
       <entry>
          <id>http://opendata.cbs.nl/ODataCatalog/Themes(0)</id>
          <category term="Cbs.OData.Catalog.Theme" scheme="http://schemas.microsoft.com/ado/2007/08/dataservices/scheme" />
          <link rel="edit" title="Theme" href="Themes(0)" />
          <title type="text">50750</title>
          <summary type="text">Arbeid en sociale zekerheid</summary>
          <updated>2015-12-19T06:52:04Z</updated>
          <author>
             <name />
          </author>
          <content type="application/xml">
             <m:properties>
                <d:ID m:type="Edm.Int32">0</d:ID>
                <d:ParentID m:type="Edm.Int32" m:null="true" />
                <d:Number>50750</d:Number>
                <d:Title>Arbeid en sociale zekerheid</d:Title>
                <d:Language>nl</d:Language>
                <d:Catalog>CBS</d:Catalog>
             </m:properties>
          </content>
       </entry>
       <entry>
          <id>http://opendata.cbs.nl/ODataCatalog/Themes(1)</id>
          ......
<p>  

  Python can parse the Atom XML output that is returned by the CBS Open Data webservices, in this case <a href="https://docs.python.org/2/library/xml.etree.elementtree.html" target="_blank">The ElementTree XML API</a> is used.
  The following function describes how an Atom feed can be parsed. The function has two parameters:
  <ul>
    <li>url: the url of Atom feed</li>
    <li>keys: the properties you want to retrieve from the Atom feed</li>
  </ul>  
  The function returns a dictionary with key-value pairs of properties you want to retrieve from the Atom feed.
</p>

    def get_Atom_xml(self, url, keys):
        data = str(urllib.urlopen(url).read())
        namespaces = self.get_namespaces_feed(data)
        tree = ET.ElementTree(ET.fromstring(data))
        root = tree.getroot()
        result = {}
        nodes = root.findall("./Atom:entry/Atom:content/m:properties",namespaces)
        for node in nodes:
            prop = {}
            for key in keys.keys():
                if node.find('d:{0}'.format(keys[key]),namespaces) is not None:
                    prop[key] = node.find('d:{0}'.format(keys[key]),namespaces).text
            if 'id' in keys:
                result[prop['id']] = prop
            else:
                result[prop['key']] = prop
        return result

<p>
  For example, the following code:
</p>
    URL_THEMES = 'http://opendata.cbs.nl/ODataCatalog/Themes'
    properties = {'id':'ID', 'parent_id':'ParentID', 'title':'Title', 'language':'Language'}
    themes = self.get_Atom_xml(URL_THEMES, properties)
    print themes
<p>
  returns
</p>

     {
       .....
       '996': {'id': '996',
               'language': 'en',
               'parent_id': '995',
               'title': 'Car and motorcycle trade'},
       '997': {'id': '997',
               'language': 'en',
               'parent_id': '995',
               'title': 'Retail trade'},
       '998': {'id': '998',
               'language': 'en',
               'parent_id': '995',
               'title': 'Wholesale trade'},
       '999': {'id': '999',
               'language': 'en',
               'parent_id': None,
               'title': 'Traffic and transport '}
      }

Once the data is retrieved in Python, you can do everything you want it. For example: replicate (a simplified version of) the Open Data portal or create a tailor made application (e.g. an applciation that contains data for a specific theme for municipalities). A demo application can be found on Heroku: <a href="">.....</a>. The application is created with Flask, the code is published on GitHub: <a href="">.....</a>


<br>
<h4>Resources:</h4>
<p>
<hr>
<ul>
  <li><a href="http://opendata.cbs.nl/dataportaal/portal.html?_la=en&_catalog=CBS">CBS Open Data Statline</a></li>
  <li><a href="http://www.cbs.nl/NR/rdonlyres/2561A2B7-CE51-47B9-A838-9968EF67FFB0/0/2014handleidingcbsopendataservices.pdf">CBS Open Data Services: Handleiding</a></li>
  <li><a href="http://www.cbs.nl/NR/rdonlyres/1C647C35-54E2-48A9-8FC0-BC851FB84E88/0/2014handleidingpowerpivotmetcbsopendatav2.pdf">PowerPivot gebruik met CBS Open Data: Handleiding</a></li>
  <li><a href="http://www.odata.org/documentation/odata-version-2-0/Atom-format/">Atom Format (OData Version 2.0)</a></li>
  <li><a href="http://www.odata.org/">OData - the best way to REST</a></li>
  <li><a href="https://docs.python.org/2/library/xml.etree.elementtree.html">The ElementTree XML API</a></li>
  <li><a href=""></a></li>
  <li><a href=""></a></li>
  <li><a href=""></a></li>
  <li><a href=""></a></li>

</ul>

</p>
<br><br>