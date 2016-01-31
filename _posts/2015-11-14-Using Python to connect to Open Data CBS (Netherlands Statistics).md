---
layout: post
title: Using Python to connect to Open Data CBS (Netherlands Statistics}
comments: false
---

### Using Python to connect to Open Data CBS (Netherlands Statistics).

<p>
  <a href="http://opendata.cbs.nl/dataportaal/portal.html#_la=en" target="_blank">StatLine</a> is the electronic databank of <a href="http://www.cbs.nl/en-GB/menu/home/default.htm?Languageswitch=on" target="_blank">CBS (Statistics Netherlands)</a>. It enables users to compile their own tables and graphs. The information can be accessed, printed and downloaded free of charge.
</p>

<p>
  Statistics Netherlands makes StatLine available as Open Data via the <a href="http://www.odata.org/" target="_blank">ODATA</a> protocol. The default way to retrieve data is PowerPivot for Excel. This post describes how to connect to Open Data Netherlands Statistics using Python.
</p>

<p> 
  The data are accessible through three webservices: catalog, api and feed.
<p>
  The data is stored as tables in the Open Data portal, tables are categorized by themes. Information about themes and tables can be retreived by using the catalog webservice: <a href="http://opendata.cbs.nl/ODataCatalog/" target="_blank">http://opendata.cbs.nl/ODataCatalog/</a>. The catalog webservice returns ATOM, an XML language used for web feeds.
  <img src="/static/tables_themes.png" class="img-responsive center-block" width="50%">

</p>
<p>
  The API and the feed give access to the data for a single table: TableInfos, DataProperties, Dimensions, UntypedDataSet, TypedDataSet.
  The feed can be used for large datasets (>10.000 records) and returns ATOM (XML) as output, the API can be used for smaller datasets (&lt10.000 records) and returns JSON.
</p>

<p>
  Python can parse the ATOM XML output that is returned by the CBS Open Data webservices, in this case <a href="https://docs.python.org/2/library/xml.etree.elementtree.html" target="_blank">The ElementTree XML API</a> is used.
  The following function describes how an ATOM feed can be parsed. The function has two parameters:
  <ul>
    <li>url: the url of ATOM feed</li>
    <li>keys: the properties you want to retrieve from the ATOM feed</li>
  </ul>  
  The function returns a dictionary with key-value pairs of properties you want to retrieve from the ATOM feed.
</p>

    def get_atom_xml(self, url, keys):
        data = str(urllib.urlopen(url).read())
        namespaces = self.get_namespaces_feed(data)
        tree = ET.ElementTree(ET.fromstring(data))
        root = tree.getroot()
        result = {}
        nodes = root.findall("./atom:entry/atom:content/m:properties",namespaces)
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
    themes = self.get_atom_xml(URL_THEMES, properties)
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