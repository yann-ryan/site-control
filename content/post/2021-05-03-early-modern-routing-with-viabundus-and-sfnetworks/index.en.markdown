---
title: "Early modern routing with Viabundus and sfnetworks"
author: "R package build"
date: '2021-05-03'
slug: early-modern-routing-with-viabundus-and-sfnetworks
categories: []
tags: []
subtitle: ''
summary: ''
authors: []
lastmod: '2021-05-03T15:12:07+02:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---

<script src="{{< blogdown/postref >}}index.en_files/kePrint/kePrint.js"></script>
<link href="{{< blogdown/postref >}}index.en_files/lightable/lightable.css" rel="stylesheet" />
<script src="{{< blogdown/postref >}}index.en_files/htmlwidgets/htmlwidgets.js"></script>
<script src="{{< blogdown/postref >}}index.en_files/jquery/jquery.min.js"></script>
<link href="{{< blogdown/postref >}}index.en_files/leaflet/leaflet.css" rel="stylesheet" />
<script src="{{< blogdown/postref >}}index.en_files/leaflet/leaflet.js"></script>
<link href="{{< blogdown/postref >}}index.en_files/leafletfix/leafletfix.css" rel="stylesheet" />
<script src="{{< blogdown/postref >}}index.en_files/Proj4Leaflet/proj4-compressed.js"></script>
<script src="{{< blogdown/postref >}}index.en_files/Proj4Leaflet/proj4leaflet.js"></script>
<link href="{{< blogdown/postref >}}index.en_files/rstudio_leaflet/rstudio_leaflet.css" rel="stylesheet" />
<script src="{{< blogdown/postref >}}index.en_files/leaflet-binding/leaflet.js"></script>
<script src="{{< blogdown/postref >}}index.en_files/kePrint/kePrint.js"></script>
<link href="{{< blogdown/postref >}}index.en_files/lightable/lightable.css" rel="stylesheet" />
<script src="{{< blogdown/postref >}}index.en_files/kePrint/kePrint.js"></script>
<link href="{{< blogdown/postref >}}index.en_files/lightable/lightable.css" rel="stylesheet" />
<script src="{{< blogdown/postref >}}index.en_files/kePrint/kePrint.js"></script>
<link href="{{< blogdown/postref >}}index.en_files/lightable/lightable.css" rel="stylesheet" />
<script src="{{< blogdown/postref >}}index.en_files/kePrint/kePrint.js"></script>
<link href="{{< blogdown/postref >}}index.en_files/lightable/lightable.css" rel="stylesheet" />
<script src="{{< blogdown/postref >}}index.en_files/htmlwidgets/htmlwidgets.js"></script>
<script src="{{< blogdown/postref >}}index.en_files/jquery/jquery.min.js"></script>
<link href="{{< blogdown/postref >}}index.en_files/leaflet/leaflet.css" rel="stylesheet" />
<script src="{{< blogdown/postref >}}index.en_files/leaflet/leaflet.js"></script>
<link href="{{< blogdown/postref >}}index.en_files/leafletfix/leafletfix.css" rel="stylesheet" />
<script src="{{< blogdown/postref >}}index.en_files/Proj4Leaflet/proj4-compressed.js"></script>
<script src="{{< blogdown/postref >}}index.en_files/Proj4Leaflet/proj4leaflet.js"></script>
<link href="{{< blogdown/postref >}}index.en_files/rstudio_leaflet/rstudio_leaflet.css" rel="stylesheet" />
<script src="{{< blogdown/postref >}}index.en_files/leaflet-binding/leaflet.js"></script>

The last couple of months have been particularly good ones for anyone working with spatial and historical data: An major new ‘online streetmap’ of early modern Northern Europe, [Viabundus](https://www.landesgeschichte.uni-goettingen.de/handelsstrassen/index.php), was released at the end of April, about a month after the first CRAN release of an R package called [sfnetworks](https://github.com/luukvdmeer/sfnetworks). This package and dataset seem made for each other: with some pretty basic code I’ve been able to do some really fun things.

# sfnetworks package

The `stnetworks` package has been in development for some time, but the first I heard of it was in March when they announced the first CRAN release (which means it’s in R’s ‘official’ list of packages that can be installed through the programming language itself). Essentially, two of my favourite packages, `sf` for spatial analysis and `tidygraph` for networks, have got together and had a baby: `sfnetworks` combines both packages, providing a really easy interface for performing spatial network analysis tasks such as calculating shortest paths.

As I wrote in an [earlier post](https://yann-ryan.github.io/post/my-network-analysis-workflow/), `tidygraph` allows you to perform network analysis within a tidy data workflow, by linking together a nodes table and an edges table, allows you to switch between the two and easily do typical data analysis tasks such as grouping and filtering, directly to the network itself.

`sfnetworks` extends this idea by making the nodes and edges tables into [simple features](https://en.wikipedia.org/wiki/Simple_Features) spatial objects: the nodes table becomes a table of spatial points, and the edges table one of spatial lines. You can also just supply one or the other and it’ll automatically make the second - so for example if you supply a table of spatial points, it’ll draw implicit edges between them, and, conversely, if you supply a dataset of spatial lines, it’ll give you a table of nodes, consisting of all the start and end points of each of the lines. Simple features objects are particularly easy to work with thanks to the R package sf: the sfnetworks format lets you leverage all the great GIS analysis from sf along with network-specific tasks.

The examples below are all based on the [package vignettes](https://luukvdmeer.github.io/sfnetworks/articles/structure.html) and the result of a couple of days playing around. I highly recommend taking a look at these to understand the full data structure - which I have simplified and, I’m sure in parts, misunderstood.

# Viabundus

As luck would have it, the data type needed for sfnetworks is pretty similar to the second resource I mentioned, Viabundus. Viabundus is the result of a four-year project by a group of researchers in Universities across Europe, which the creators describe as a ‘online street map of late medieval and early modern northern Europe (1350-1650).’ It’s an atlas of early modern roads, which have been digitised and converted into a nifty [online map](https://www.landesgeschichte.uni-goettingen.de/handelsstrassen/map.php) which allows you to calculate routes across Northern Europe, including travel times, tolls passed, fairs visited, to a pretty mind-boggling level of detail. Basically Google Maps for the seventeenth-century.

Alongside the map, the team have released the dataset as Open Access (CC-BY-SA) data. There’s extensive [documentation](https://www.landesgeschichte.uni-goettingen.de/handelsstrassen/index.php#documentation), but in essence, to reconstruct the basic map and perform spatial analysis with it, the data contains two key tables: one edges table, the roads, and one nodes table, the towns, markets, bridges and other points through which the routes pass. As I said - conveniently this is more or less the data structure required as an input for `sfnetworks`. The rest of this post is a short demonstration on how the two can be used together.

# Create a `sfnetworks` object from Viabundus data

The data is freely available [here](https://www.landesgeschichte.uni-goettingen.de/handelsstrassen/index.php#download), with a CC-BY-SA licence. I recommend having a good read of the documentation to understand all the additional tables, but the ones we’ll use today are the two main ones: the nodes and edges. First, import them into R:

``` r
library(tidyverse)
nodes = read_csv('Viabundus-1.0-CSV/Nodes.csv')
edges = read_csv('Viabundus-1.0-CSV/Edges.csv')
```

Next, you need to convert them into `sf` objects. The geographic data in the Viabundus edges table is supplied in a format called ‘Well Known Text’ (WKT). The sf package supports this as a data input - just supply the column containing the spatial data as the argument `wkt =`. In this case, the WKT column in the edges table is also called WKT. Use `st_set_crs` to set the Coordinate Reference System, which needs to be the same for both tables.

``` r
library(sf)
edges_sf = edges %>% st_as_sf(wkt = "WKT")
edges_sf = edges_sf %>% st_set_crs(4326)
```

The nodes table doesn’t use WKT, but contains a regular latitude and longitude column. Use `st_as_sf` again, and supply the column names to the `coords =` argument.

``` r
nodes_sf = nodes %>% st_as_sf(coords = c('Longitude', 'Latitude'))
nodes_sf = nodes_sf %>% st_set_crs(4326)
```

To create the sfnetworks object, you supply data in one of a number of forms. The most complete is to supply a both a table of edges and a table of nodes. The edges table should contain ‘from’ and ‘to’ columns, corresponding to the node IDs in the node table, which will link them together.

As far as I can tell, the Viabundus data doesn’t explicitly connect the nodes and edges data like this. I’ve used a sort of work-around to get around this.

First, use `as_sfnetwork` to create an sfnetwork from the edges table alone.

``` r
library(sfnetworks)
sf_net = as_sfnetwork(edges_sf, directed = F)

sf_net
```

    ## # A sfnetwork with 12567 nodes and 14819 edges
    ## #
    ## # CRS:  EPSG:4326 
    ## #
    ## # An undirected multigraph with 10 components with spatially explicit edges
    ## #
    ## # Node Data:     12,567 x 1 (active)
    ## # Geometry type: POINT
    ## # Dimension:     XY
    ## # Bounding box:  xmin: 3.224534 ymin: 48.72051 xmax: 37.6206 ymax: 60.71302
    ##                   WKT
    ##           <POINT [°]>
    ## 1 (22.47944 58.24702)
    ## 2 (22.80928 58.30961)
    ## 3 (22.86145 58.36151)
    ## 4 (23.00004 58.40795)
    ## 5  (23.04775 58.5087)
    ## 6 (23.03553 58.57549)
    ## # … with 12,561 more rows
    ## #
    ## # Edge Data:     14,819 x 12
    ## # Geometry type: LINESTRING
    ## # Dimension:     XY
    ## # Bounding box:  xmin: 3.222333 ymin: 48.72051 xmax: 37.6206 ymax: 60.71302
    ##    from    to     ID Section Type  Certainty Zoomlevel From  To    Comment_ID
    ##   <int> <int>  <dbl> <chr>   <chr>     <dbl>     <dbl> <chr> <chr> <chr>     
    ## 1     1     2 227856 EST     land          2         1 NULL  NULL  NULL      
    ## 2     2     3 227858 EST     land          2         1 NULL  NULL  NULL      
    ## 3     3     4 227860 EST     land          3         1 NULL  NULL  NULL      
    ## # … with 14,816 more rows, and 2 more variables: Length <dbl>, WKT <LINESTRING
    ## #   [°]>

Looking at the data, you can see it has created a simple nodes table of all the start and end points of the spatial lines in the edges table. We need to know what towns and other points correspond to these nodes.

To do this, I used the `sf` function `st_join`. Use the join function ‘nearest feature,’ which will link the sfnetwork nodes table to the closest point in the original Viabundus nodes table. This is the best method I could think of for now, though it’s not perfect—it would be better if there was an explicit link between the nodes and edges table, which I haven’t been able to figure out.

``` r
sf_net = sf_net %>% 
  st_join(nodes_sf, join = st_nearest_feature)
```

One of the key things which makes a spatial network object different to a regular network object is that we treat the length of each edge as a weight. This can be done automatically with sfnetworks using `edge_length`, though the Viabundus edges table has a column with this information already.

``` r
sf_net = sf_net %>%
  activate("edges") %>%
  mutate(weight = edge_length())
```

# Network Calculations

This new object can be used to calculate regular network analysis metrics, such as betweenness centrality, using the same set of functions as `tidygraph`:

``` r
library(tidygraph)
library(kableExtra)
betweenness_scores = sf_net %>% 
  activate(nodes) %>% 
  mutate(betweenness = centrality_betweenness(weights = NULL)) %>% 
  as_tibble() %>% 
  arrange(desc(betweenness)) %>% 
  filter(Is_Town == 'y') %>% head(10)

betweenness_scores %>% st_drop_geometry() %>% select(Name, betweenness) %>% kbl(caption = "Nodes with highest betweenness scores:")
```

<table>
<caption>
Table 1: Nodes with highest betweenness scores:
</caption>
<thead>
<tr>
<th style="text-align:left;">
Name
</th>
<th style="text-align:right;">
betweenness
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
Wittenberge
</td>
<td style="text-align:right;">
21883786
</td>
</tr>
<tr>
<td style="text-align:left;">
Cölln (Berlin)
</td>
<td style="text-align:right;">
21044547
</td>
</tr>
<tr>
<td style="text-align:left;">
Havelberg
</td>
<td style="text-align:right;">
20997097
</td>
</tr>
<tr>
<td style="text-align:left;">
Hitzacker
</td>
<td style="text-align:right;">
20414717
</td>
</tr>
<tr>
<td style="text-align:left;">
Berlin
</td>
<td style="text-align:right;">
19693706
</td>
</tr>
<tr>
<td style="text-align:left;">
Berlin
</td>
<td style="text-align:right;">
19316790
</td>
</tr>
<tr>
<td style="text-align:left;">
Bernau bei Berlin
</td>
<td style="text-align:right;">
19291164
</td>
</tr>
<tr>
<td style="text-align:left;">
Potsdam
</td>
<td style="text-align:right;">
19272557
</td>
</tr>
<tr>
<td style="text-align:left;">
Bernau bei Berlin
</td>
<td style="text-align:right;">
19257977
</td>
</tr>
<tr>
<td style="text-align:left;">
Malchow
</td>
<td style="text-align:right;">
19240900
</td>
</tr>
</tbody>
</table>

Or color all nodes on their betweenness score, and display on a map:

``` r
betweenness_sf = sf_net %>% 
  activate(nodes) %>% 
  mutate(betweenness = centrality_betweenness(weights = NULL)) %>%
  as_tibble()

ggplot() + geom_sf(data = betweenness_sf, aes(color = betweenness), alpha =  .8) + scale_color_viridis_c() +theme_void() + labs(title  = "Viabundus Network, Betweenness Centrality Scores", caption = 'Made with data from Viabundus.eu') + theme(title = element_text(size = 14, face = 'bold'))
```

<img src="{{< blogdown/postref >}}index.en_files/figure-html/unnamed-chunk-8-1.png" width="672" />

Interestingly, the highest betweenness nodes all seem to fall along a single path. Is this a particularly important trunk road? We can also plot the whole network. To use ggplot, create separate nodes and edges tables using `as_tibble`, and add them as separate geoms to the same plot:

``` r
ggplot() + 
  geom_sf(data = sf_net %>% activate(nodes) %>% as_tibble(), alpha =  .1, size  = .01)  + 
  geom_sf(data = sf_net %>% activate(edges) %>% as_tibble(), alpha = .5) +theme_void() + 
  labs(title  = "Viabundus Network, All Routes", caption = 'Made with data from Viabundus.eu') + 
  theme(title = element_text(size = 14, face = 'bold'))
```

<img src="{{< blogdown/postref >}}index.en_files/figure-html/unnamed-chunk-9-1.png" width="672" />

# Shortest-path calculation

The nicest application of these two resources I’ve found so far is to use `sfnetworks` to calculate the shortest path between any set of points in the network. This uses [Dijkstra’s algorithm](https://en.wikipedia.org/wiki/Dijkstra%27s_algorithm), along with the lengths of the spatial lines as weights (or impedance), to calculate the shortest route between any two points (or from one start point to a range of end points). This is done with the function `st_shortest_paths`.

`st_network_paths` takes `from` and `to` arguments, using the node ID from the nodes table. You’ll need to figure out some way to find the relevant node IDs to enter here—the most convenient way I’ve found is first to make a copy of just the nodes table:

``` r
nodes_lookup_table = sf_net %>% activate(nodes) %>% as_tibble()
```

The ID you need is the row index - create a column called node\_id containing this info:

``` r
nodes_lookup_table  = nodes_lookup_table %>% mutate(node_id = 1:nrow(.))
```

Looking at the table, you’ll see that sometimes the same place has been assigned to multiple closest coordinates. For now, I’m just going to pick the first one.

``` r
utrecht_node_id = nodes_lookup_table %>% filter(Name == 'Utrecht') %>% head(1) %>% pull(node_id)
```

``` r
hamburg_node_id = nodes_lookup_table %>% filter(Name == 'Hamburg') %>% head(1) %>% pull(node_id)
```

Run `st_network_paths` supplying the points you want to calculate the route between as from and to arguments:

``` r
paths = st_network_paths(sf_net, from =utrecht_node_id, to = hamburg_node_id)

paths
```

    ## # A tibble: 1 x 2
    ##   node_paths edge_paths
    ##   <list>     <list>    
    ## 1 <int [99]> <int [98]>

The result of st\_network\_paths is a dataframe with two columns, each containing a vector of edge or node IDs. First convert each into a dataframe with a single columns of IDs:

``` r
 node_list = paths %>%
  slice(1) %>%
  pull(node_paths) %>%
  unlist() %>% as_tibble()

edge_list = paths %>%
  slice(1) %>%
  pull(edge_paths) %>%
  unlist() %>% as_tibble()
```

Use `inner join()` to attach the edge and node IDs in the route to the edge and nodes tables from the sfnetwork object. You’ll need to recreate them as sf objects:

``` r
line_to_draw = edge_list %>% inner_join(sf_net %>% 
  activate(edges) %>% 
  as_tibble() %>% 
  mutate(edge_id = 1:nrow(.)) , by = c('value' = 'edge_id'))

line_to_draw = line_to_draw %>% st_as_sf()
line_to_draw = line_to_draw %>% st_set_crs(4326)

nodes_to_draw = node_list %>% inner_join(sf_net %>% 
  activate(nodes) %>% 
  as_tibble() %>% 
  mutate(node_id = 1:nrow(.)) , by = c('value' = 'node_id'))

nodes_to_draw = nodes_to_draw %>% st_as_sf()
nodes_to_draw = nodes_to_draw %>% st_set_crs(4326)
```

Plot this using Leaflet:

``` r
library(leaflet)

leaflet() %>% 
  addTiles() %>% 
  addCircles(data = nodes_to_draw, label = ~Name) %>% 
  addPolylines(data  = line_to_draw)
```

<div id="htmlwidget-1" style="width:672px;height:480px;" class="leaflet html-widget"></div>
<script type="application/json" data-for="htmlwidget-1">{"x":{"options":{"crs":{"crsClass":"L.CRS.EPSG3857","code":null,"proj4def":null,"projectedBounds":null,"options":{}}},"calls":[{"method":"addTiles","args":["//{s}.tile.openstreetmap.org/{z}/{x}/{y}.png",null,null,{"minZoom":0,"maxZoom":18,"tileSize":256,"subdomains":"abc","errorTileUrl":"","tms":false,"noWrap":false,"zoomOffset":0,"zoomReverse":false,"opacity":1,"zIndex":1,"detectRetina":false,"attribution":"&copy; <a href=\"http://openstreetmap.org\">OpenStreetMap<\/a> contributors, <a href=\"http://creativecommons.org/licenses/by-sa/2.0/\">CC-BY-SA<\/a>"}]},{"method":"addCircles","args":[[52.091844,52.104481,52.101371,52.153184,52.15427,52.156362,52.161969,52.172371,52.175487,52.164211,52.185609,52.187098,52.20401,52.218511,52.214558,52.241551,52.248354,52.252153,52.256011,52.251815,52.303657,52.306799,52.320311,52.308754,52.308009,52.357344,52.351456,52.35587,52.406696,52.408124,52.410181,52.428739,52.433517,52.43356,52.434269,52.435794,52.437788,52.489641,52.507742,52.509446,52.509959,52.510661,52.522354,52.523037,52.600366,52.671096,52.671517,52.673601,52.733187,52.795285,52.825826,52.84306,52.84583,52.84825,52.849548,52.863906,52.898445,52.900309,52.898445,52.894791,52.89731,52.899991,52.942256,53.036234,53.056729,53.059044,53.06917,53.072859,53.073386,53.075887,53.074656,53.068596,53.051802,53.038871,53.038019,53.028939,53.01297,53.029817,53.06366,53.110276,53.140233,53.218914,53.273983,53.296491,53.295483,53.314912,53.336027,53.346258,53.365284,53.378304,53.414627,53.468829,53.470173,53.476376,53.477427,53.555098,53.541838,53.54469,53.54798],[5.118906,5.184108,5.205463,5.383635,5.387491,5.389577,5.422375,5.436732,5.455731,5.520019,5.603746,5.60728,5.674221,5.873604,5.962314,6.104787,6.124177,6.143585,6.152218,6.160268,6.270168,6.292892,6.47423,6.512146,6.521151,6.592483,6.665096,6.665282,6.894704,6.897136,6.902075,7.050764,7.067999,7.06945,7.070189,7.070657,7.070518,7.232169,7.289806,7.295653,7.299841,7.302806,7.316653,7.323251,7.411333,7.48676,7.486634,7.486447,7.758713,7.864115,7.997846,8.03707,8.04294,8.04252,8.04852,8.063696,8.173414,8.210821,8.231068,8.432223,8.43632,8.444025,8.573448,8.710165,8.720241,8.736295,8.760915,8.799608,8.804305,8.807392,8.809272,8.860087,8.886329,8.918209,8.941825,8.970547,9.032515,9.054039,9.085822,9.143259,9.146686,9.203015,9.257674,9.274875,9.278606,9.334932,9.373245,9.398899,9.426945,9.465251,9.549767,9.68523,9.689763,9.700347,9.702158,9.795553,9.936447,9.98846,9.9931],10,null,null,{"interactive":true,"className":"","stroke":true,"color":"#03F","weight":5,"opacity":0.5,"fill":true,"fillColor":"#03F","fillOpacity":0.2},null,null,["Utrecht","De Bilt","De Bilt",null,"Amersfoort","Amersfoort","Amersfoort","Hoevelaken","Hoevelaken","Terschuur","Voorthuizen","Voorthuizen","Garderen","Hoog-Soeren","Apeldoorn","Twello","Twello",null,"Deventer","Deventer","Heeten","Heeten","Rijssen","Rijssen","Rijssen","Wierden","Almelo","Almelo","Ootmarsum","Ootmarsum","Ootmarsum",null,null,null,null,"Nordhorn","Nordhorn","Südlohne (Bentheim)","Schepsdorf","Schepsdorf",null,null,"Lingen","Lingen","Bawinkel",null,null,"Haselünne","Löningen","Lastrup","Krapendorf","Krapendorf","Cloppenburg","Cloppenburg","Cloppenburg","Bethen","Lethe","Ahlhorn","Ahlhorn","Wildeshausen","Wildeshausen","Wildeshausen","Horstedt","Varrel","Mittelshuchting","Kirchhuchting","Warturm","Neustadt",null,"Bremen","Bremen","Hastedt","Hemelingen","Arbergen","Mahndorf (Bremen)","Uphusen (Bremen)","Achim (Weser)","Borstel (Achim)","Bassen","Ottersberg","Otterstedt","Steinfeld","Oldendorf (Zeven)","Zeven","Zeven","Heeslingen","Boitzen","Steddorf","Wangersen","Ahrenswohlde","Kammerbusch","Buxtehude","Buxtehude","Buxtehude",null,"Blankenese","Ottensen",null,"Hamburg"],{"interactive":false,"permanent":false,"direction":"auto","opacity":1,"offset":[0,0],"textsize":"10px","textOnly":false,"className":"","sticky":true},null,null]},{"method":"addPolylines","args":[[[[{"lng":[5.184108,5.18408191206572,5.18353185435215,5.1834702,5.1824421,5.1820807,5.181767,5.1810762,5.18037541212663,5.179327,5.1786603161669,5.17638271768028,5.17303381740489,5.1728491,5.171529,5.1711436,5.1702536,5.1700049,5.1699107,5.169513,5.1692675,5.1689527,5.1683429,5.1679684,5.1675327,5.16691656792807,5.1666904,5.1659161,5.1657553,5.1649986,5.1639457,5.1632836,5.1631241,5.1627271,5.1616786,5.1607973,5.1601547,5.1597974,5.15948292735576,5.1585999,5.1584998,5.1572437,5.1566872,5.15654367636462,5.15634977297964,5.15616503736449,5.15597248869148,5.15579577837754,5.1557459,5.1544221,5.154226,5.1539694,5.1526992,5.15175689943781,5.1496705,5.149667,5.14933176457253,5.14374722655162,5.14344859113303,5.14289457084517,5.1417745,5.1415925,5.1414049,5.14134724652312,5.1412876,5.1411431,5.1409713,5.140785,5.1393718,5.13925382038256,5.1392513,5.1390949,5.1389801,5.1387998,5.13877464942673,5.1385764,5.1383697,5.1383103,5.1382125,5.1379564,5.1375214,5.1374479,5.137381,5.13717351279288,5.1354503,5.1345707050126,5.1345554,5.1328425,5.1327609,5.1326189,5.13228679365802,5.1314426,5.131348,5.1311757,5.130906,5.130846,5.13059819950691,5.130135,5.13010211497808,5.1292867186888,5.1292752,5.12865467857095,5.1285616,5.12740899861577,5.1272703,5.1269133,5.1268577,5.1267472,5.12659249668308,5.1265564,5.1264105,5.1263931,5.1262456,5.1260909,5.12600927158765,5.1259729,5.1258445,5.12579739507981,5.12563725774447,5.1255851,5.1251002,5.1247226,5.12366687319359,5.1235525,5.12349,5.1233831,5.1233515,5.12333600063173,5.1224621,5.12209699232115,5.1220332,5.1217802,5.1217445,5.12157324613483,5.12102104127881,5.1209833,5.1208892,5.1205731,5.12051280976605,5.120276,5.12025934158409,5.1196409,5.11962234782893,5.1189182,5.11879988899074,5.11879988899074,5.11907523396118,5.11955346469933,5.118906],"lat":[52.104481,52.1044725350733,52.1043649688352,52.104355,52.1041631,52.1040929,52.1040311,52.1038995,52.1037606007377,52.1035528,52.1034346388667,52.1030539514178,52.1023861473695,52.1023476,52.102036,52.1019313,52.1016902,52.1016228,52.1015971,52.1014892,52.1014292,52.1013493,52.1012027,52.1011113,52.101008,52.1008640433101,52.1008112,52.1006279,52.1005891,52.1004099,52.1001589,52.1000011,52.0999635,52.0998679,52.0996153,52.0994222,52.0992786,52.0991997,52.0991302141715,52.0989351,52.0989132,52.0986536,52.0985411,52.0985120537841,52.0984728117185,52.0984354249894,52.0983964570229,52.0983606943909,52.0983506,52.0980932,52.0980526,52.0980077,52.0977649,52.0975832667618,52.0971811,52.0971791,52.0971256464722,52.0962193072214,52.0961205194145,52.0959750790313,52.095681,52.0956301,52.095588,52.0955754174868,52.0955624,52.0955522,52.0955467,52.0955504,52.0955943,52.0955982163359,52.0955983,52.0956046,52.0956093,52.0956166,52.0956171516464,52.0956215,52.0956255,52.0956245,52.0956244,52.0956218,52.095617,52.0956168,52.0956178,52.095608901711,52.095535,52.0954902781634,52.0954895,52.0954014,52.0953969,52.0953898,52.0953713073077,52.0953243,52.0953191,52.0953096,52.0952947,52.0952911,52.0952778560955,52.0952531,52.0952513482749,52.0952079135802,52.0952073,52.0951704304555,52.0951649,52.0950964383582,52.0950882,52.095071,52.0950679,52.0950561,52.0950388296619,52.0950348,52.0950238,52.0950217,52.0950049,52.0949873,52.094978791276,52.094975,52.0949621,52.0949647330818,52.094973684475,52.0949766,52.0950349,52.0950795,52.0951612441741,52.0951701,52.0951685,52.0951658,52.095165,52.0951646863182,52.095147,52.0951418924084,52.095141,52.0951374,52.0951369,52.0951294757016,52.0951055361832,52.0951039,52.0950995,52.0950146,52.0949962755175,52.0949243,52.0949125334612,52.0944757,52.0944630341748,52.0939823,52.0938314367265,52.0936088523146,52.0930746451954,52.0923089371696,52.091844]}]],[[{"lng":[5.205463,5.20321109141122,5.20185113413566,5.19977552229413,5.19817761136005,5.19724,5.1957991,5.19524,5.19513815029995,5.193862,5.1936444,5.19335129495976,5.193161,5.1927713,5.1923807,5.1899826894044,5.1898913,5.1892114,5.1885697,5.1884598,5.1883652,5.1879676,5.1869305,5.1864891,5.1864061,5.1861861,5.1859896,5.1858389,5.1857818,5.18564113845318,5.1854692,5.1845725,5.184108],"lat":[52.101371,52.1025410164095,52.1026155724686,52.1027553083223,52.1034278210086,52.10331,52.1031011,52.10302,52.1030069546886,52.1028435,52.1028286,52.1028325412139,52.1028351,52.1028985,52.1029771,52.1035459220458,52.1035676,52.1037255,52.1038587,52.103877,52.1038931,52.1039632,52.1041178,52.1041864,52.104197,52.1042194,52.1042436,52.1042617,52.1042692,52.1042859839952,52.1043065,52.1044198,52.104481]}]],[[{"lng":[5.383635,5.38343171141255,5.38208532349586,5.38174502764879,5.38131595897205,5.38081291293724,5.38017670765793,5.37974763898119,5.37917061558832,5.37849002389418,5.37769106842713,5.37522022466795,5.37424372354156,5.3735483363758,5.37072240129793,5.3681923756523,5.36567714547828,5.36461187152222,5.36382771152679,5.36295477870169,5.36215582323465,5.35984772966319,5.35267715978609,5.35061307693823,5.3502026,5.3499914,5.3494678,5.3476296,5.34468539349166,5.3407026,5.3403069,5.3401018,5.3400235,5.3399312,5.3396724,5.3395915,5.3393259,5.3387326,5.3383857,5.3351835,5.3345116,5.3343499,5.3320532,5.3316967,5.3312399,5.3295931,5.32830929596605,5.3267961,5.326534,5.3256535,5.3246769,5.3233381,5.3216177,5.3198947,5.3180175,5.317381,5.3165185,5.3159432,5.31335678387486,5.30125443790094,5.28527494280933,5.27309077175878,5.27020504703628,5.265706,5.2652884,5.2648948,5.2629503,5.26146,5.2597213,5.2594246,5.2592618,5.25813767220102,5.257707,5.257086,5.2558915,5.2543876,5.25415169772832,5.2541098,5.2539041,5.2519249,5.251687,5.2500485,5.2481494,5.24529430949217,5.243552,5.2399731,5.23875511691239,5.2384577,5.23772657064713,5.2370043,5.2369731,5.2368078,5.2365788,5.2361893,5.235338,5.223178005027,5.22239554409793,5.2218067,5.2202,5.219735,5.2193,5.21914,5.21908604822665,5.21875,5.21834406447976,5.21819,5.21748,5.21615,5.21535081114665,5.21396175534356,5.2137123,5.2130116,5.2121328,5.2095714,5.2079206,5.20761701098588,5.20597059607565,5.205463],"lat":[52.153184,52.1529795274015,52.1521806703154,52.1519264855091,52.1513454862165,52.1510277490214,52.150682774642,52.1504648947094,52.150210700107,52.1499746609625,52.1497567775646,52.1491394354801,52.1488852333104,52.148494848581,52.1467880100335,52.1451537416025,52.1436192905382,52.1429927834832,52.1426568267944,52.142302707541,52.1420303061993,52.1414491777685,52.1393504340168,52.1388554270736,52.1387486,52.1386939,52.1385549,52.138067,52.1372509884459,52.1361471,52.1360304,52.1359767,52.1359535,52.1359295,52.1358585,52.1358361,52.1357661,52.1356022,52.1355073,52.1346311,52.1344477,52.1344064,52.1337755,52.1336807,52.1335561,52.1331117,52.1327669517653,52.1323606,52.132289,52.1320479,52.1317832,52.1314191,52.1309582,52.1304747,52.1299546,52.1297782,52.1295443,52.1293899,52.128695816269,52.1254420683244,52.121123240129,52.1177107185476,52.1169494280909,52.1158178,52.1157067,52.1155969,52.1150663,52.11466,52.1141771,52.114094,52.114051,52.1137506642108,52.1136356,52.1134696,52.113141,52.1127272,52.1126623226378,52.1126508,52.1125897,52.1120343,52.1119685,52.1115004,52.11092,52.1101263370924,52.109642,52.1086575,52.1083286914791,52.1082484,52.1080990454206,52.1079515,52.1079479,52.107927,52.1078861,52.1078348,52.1077483,52.1066136448338,52.1066243190081,52.1066849,52.1069,52.1069388,52.10694,52.10693,52.1069230831093,52.10688,52.1067785161638,52.10674,52.10643,52.1059,52.1056084424235,52.1051109632132,52.1050276,52.1047833,52.1044374,52.1033371,52.1025833,52.1024615197771,52.1016858128694,52.101371]}]],[[{"lng":[5.383635,5.38433442848591,5.3845218,5.38459871767102,5.3855201,5.38628219735005,5.386526,5.3869364,5.387491],"lat":[52.153184,52.1537137006806,52.1538543,52.153871674512,52.1540798,52.1542156426499,52.1542591,52.154312,52.15427]}]],[[{"lng":[5.387491,5.3876929,5.3880775,5.3882475,5.38844155738913,5.3886569,5.3889462799543,5.3890885,5.38917665934927,5.38932312644791,5.3894906,5.3895545,5.38971127990172,5.3897317,5.38970559786196,5.3896702093685,5.38948915337362,5.3894415,5.38948630171319,5.38968679693163,5.3897878,5.3897765831954,5.38967181228424,5.38960074923291,5.389577],"lat":[52.15427,52.1543497,52.1544814,52.1545615,52.1546781052445,52.1548075,52.1549990568591,52.1550932,52.1551551155979,52.1552579813888,52.1553756,52.1554282,52.1555572868622,52.1555741,52.155589093914,52.1556094222033,52.1557134264474,52.1557408,52.155770025323,52.1559008133349,52.1559667,52.1559819284317,52.1561241698188,52.1562635272924,52.156362]}]],[[{"lng":[5.389577,5.39007885305209,5.3903856,5.39043593034294,5.39053564418951,5.3906661,5.39073096791219,5.39087907329286,5.3911474,5.3914585,5.3917992,5.3922122,5.392547,5.39255744452224,5.39272856461431,5.3927926,5.3929622,5.3929701,5.3930781,5.39317044365424,5.3936316,5.39368721387073,5.3945862,5.3950419,5.39504548718099,5.3953774876182,5.39578402515421,5.39665572943035,5.39697276877882,5.3971354,5.39728088552597,5.3973009,5.397515,5.3975685,5.39796515032194,5.39856432346906,5.39884693810242,5.39927161248473,5.39944723758691,5.39980612366526,5.40027954785373,5.40276401852504,5.40832171058319,5.41086542348673,5.41678650310252,5.4209975236235,5.422375],"lat":[52.156362,52.1563657052323,52.1563774,52.1563601387711,52.1563259410204,52.1562812,52.1563158511155,52.1563949658877,52.1565383,52.1566946,52.1568296,52.1569678,52.1570639,52.1570666727334,52.1571121003873,52.1571291,52.1571742,52.1571763,52.157205,52.1572294248049,52.1573514,52.1573670716675,52.1576204,52.1577698,52.1577712650438,52.157906857429,52.1580728909676,52.1584289006446,52.1585583808634,52.1586248,52.1586868621095,52.1586954,52.1587717,52.158795,52.1587040512978,52.1586282287145,52.1584838844833,52.1572842862927,52.1571203257638,52.1570453721781,52.157092218184,52.1574684203318,52.1583863821324,52.1588256857248,52.1604845094446,52.1616515499524,52.161969]}]],[[{"lng":[5.422375,5.42316716110004,5.4252840093891,5.4277736327867,5.43350233075433,5.43497314429979,5.43448294276651,5.43432262472637,5.43458051353706,5.4351902,5.4352843,5.4356469,5.43564874238848,5.43567858217567,5.43583494466322,5.43593381386485,5.4360550875258,5.43616921880727,5.4362844,5.43630269814782,5.4365319,5.436732],"lat":[52.161969,52.1622153900198,52.1628233500071,52.1639888171499,52.1667225756094,52.1674678186421,52.1687842559647,52.1694512497447,52.1702803993815,52.1710077,52.1710868,52.17142,52.1714216747781,52.1714487998971,52.1715909370709,52.171680811263,52.1717910513339,52.171894798596,52.1719995,52.1720159351017,52.1722218,52.172371]}]],[[{"lng":[5.436732,5.43692100959683,5.43728601557787,5.4374089,5.4375351,5.4384412,5.4385832,5.4387898,5.438835,5.4392381,5.43981377433523,5.4403921,5.4403921,5.4426514,5.4435469927404,5.44485569311162,5.44709563496218,5.4473997,5.4479091,5.4483696,5.4484763,5.4490818,5.4491448,5.4494466,5.4497258,5.4504548,5.450927,5.4510234,5.4519395,5.4527678,5.45294398641969,5.4539987,5.4548837,5.45535557532315,5.455731],"lat":[52.172371,52.1724772620467,52.1726301425995,52.172684,52.1727337,52.1729988,52.1730369,52.1730923,52.1731044,52.1731897,52.1732693356478,52.1733424,52.1733424,52.1736234,52.1737331244083,52.1738871223956,52.17425913918,52.1743053,52.1743771,52.174443,52.1744584,52.1745478,52.1745564,52.1745908,52.1746261,52.1747303,52.1747961,52.1748092,52.174938,52.1750565,52.1750849032215,52.175246,52.1753893,52.1754594709169,52.175487]}]],[[{"lng":[5.455731,5.45618022895256,5.45703557277921,5.45743124335026,5.45775263783297,5.4578928,5.4581366,5.4590254,5.4597271,5.4598033,5.4600372,5.4602735,5.4604381,5.4605905,5.4607616,5.4609251,5.4610606,5.4611918,5.46145,5.4615315,5.4620803,5.4624697,5.46326111980758,5.4638404,5.4640611,5.4643006,5.4647973,5.4654,5.4656447,5.4658777,5.466105,5.4663654,5.4667315,5.4673375,5.4675093,5.4686236,5.4687439,5.4691824,5.4696206,5.47016151414541,5.47189208919608,5.472473,5.472473,5.4733154,5.4734915,5.473666,5.4740206,5.4741678,5.4743258,5.4745036,5.4746724,5.4747799,5.475278,5.475676,5.4757939,5.4759203,5.4766126,5.477122,5.4774219,5.4777294,5.4784069,5.479132,5.4798354,5.480046,5.4802533,5.4803679,5.480811,5.4812901,5.4822,5.482704,5.4832178,5.4835359,5.4840166,5.484133,5.4842527,5.4843226,5.4843651,5.4843982,5.4844386,5.4844636,5.4845449,5.4847329,5.4849048,5.4849244,5.4849404,5.4849493,5.4849433,5.4849272,5.484901,5.4847282,5.4847056,5.484693,5.4846763,5.4846683,5.4846354,5.4846435,5.4846593,5.4846865,5.4847541,5.484808,5.4849331,5.4850131,5.4850951,5.4852026,5.4853119,5.4855167,5.4856984,5.4858208,5.4861223,5.4864181,5.4865977,5.4866977,5.4867806,5.4869325,5.4870781,5.4891311,5.489724,5.489833,5.4899216,5.4902598,5.4931998,5.4959048,5.4960206,5.4961534,5.4963328,5.4964177,5.4965165,5.496645,5.4967534,5.4968625,5.4973665,5.4978105,5.4980117,5.4982155,5.4984145,5.498597,5.500804,5.5010489,5.5012885,5.501681,5.5028599,5.5033325,5.5035438,5.5036607,5.5037673,5.5041085,5.50462,5.5046926,5.5047766,5.5048468,5.5049628,5.506249,5.5068913,5.5075374,5.5080675,5.5083288,5.5086226,5.50938,5.5099294,5.5104572,5.510584,5.510744,5.5108529,5.510953,5.51134810214892,5.51231507316224,5.5129620612604,5.51305,5.5134119,5.5137957,5.5144852,5.514838,5.51590834753315,5.51715516960228,5.5172942,5.5174489,5.5176421,5.5177451,5.5178666,5.5182346,5.5185809,5.518709,5.5188439,5.519025,5.5192168,5.51953321916289,5.520019],"lat":[52.175487,52.1755500557696,52.1756982240782,52.1757590186447,52.1758324246752,52.1758537,52.1758918,52.1760318,52.1761341,52.1761441,52.1761839,52.176215,52.1762307,52.1762336,52.1762307,52.1762219,52.1762099,52.1761908,52.17614,52.1761243,52.1760155,52.1759386,52.1757872101882,52.1756764,52.1756367,52.1756054,52.1755533,52.17549,52.175466,52.1754354,52.1754238,52.1754037,52.1753795,52.1753493,52.1753415,52.1753057,52.1752913,52.1752777,52.1752727,52.1752724475313,52.1753504835679,52.1753986,52.1753986,52.1754617,52.1754776,52.1754932,52.1755343,52.1755551,52.1755702,52.1755782,52.1755735,52.1755632,52.1754869,52.1754319,52.1754215,52.1754122,52.1753891,52.1753515,52.175337,52.1753284,52.1753102,52.1752937,52.1752557,52.1752456,52.175226,52.1752128,52.1751537,52.1750843,52.17494,52.1748679,52.1748129,52.1747888,52.174759,52.1747441,52.1747196,52.174699,52.1746782,52.1746559,52.1746128,52.1745844,52.1744624,52.1741142,52.1737664,52.1737133,52.1736572,52.1735465,52.1734235,52.1733547,52.1732825,52.1729689,52.1729151,52.1728639,52.172793,52.1727188,52.171911,52.1718696,52.1718246,52.1717805,52.1717138,52.1716866,52.1716224,52.1715921,52.1715661,52.1715423,52.171528,52.1715076,52.1714972,52.1714958,52.1714961,52.1714892,52.1714747,52.1714625,52.1714486,52.1714151,52.1713732,52.1706431,52.1704445,52.1704068,52.1703651,52.1702044,52.1687843,52.1675468,52.1675003,52.1674558,52.1674031,52.1673823,52.1673643,52.1673522,52.1673499,52.1673536,52.1673846,52.1674124,52.1674262,52.167432,52.1674309,52.1674238,52.1672651,52.1672348,52.1671917,52.1671167,52.1668894,52.1668043,52.1667719,52.1667609,52.1667537,52.1667503,52.16675,52.1667476,52.1667439,52.1667341,52.1667152,52.1664277,52.1662859,52.1661414,52.1659891,52.1659105,52.1658504,52.1657094,52.165605,52.1654898,52.1654505,52.165389,52.1653429,52.1652913,52.1650562944911,52.164461876175,52.1641330267986,52.16409,52.1639167,52.1637459,52.1634249,52.1632542,52.1627411470826,52.1633125921857,52.1634284,52.1635412,52.1636468,52.1636968,52.1637441,52.1638936,52.1640336,52.1640723,52.1641036,52.1641285,52.1641477,52.1641793542401,52.164211]}]],[[{"lng":[5.520019,5.52033228555333,5.52107682276183,5.5222539,5.5224261,5.5229879,5.5232794,5.5236024,5.5245207,5.52510041407888,5.5254823,5.5259684,5.5269015,5.5270486,5.5272438,5.5273232,5.5274508,5.5275877,5.5276612,5.52774,5.52841694636538,5.5301123,5.5306211,5.5311087,5.5320775,5.5330749,5.5332848415314,5.53385128123287,5.53408100078562,5.53455415845047,5.53541327802732,5.53693016103019,5.53745368702233,5.53799063675786,5.53953,5.5401759,5.5407889,5.5413335,5.5444655,5.549273,5.5503445,5.55263895329009,5.5530534,5.5542898,5.5546754,5.55585781007388,5.55704147008331,5.55800190821911,5.55957,5.5598793,5.56387,5.57199344268536,5.57283086067589,5.57327902468225,5.5775232,5.5793364,5.5811679,5.5820919,5.58272792356477,5.58473033350851,5.58631803548192,5.5867226,5.5873946,5.5880618,5.5884316,5.5887569,5.5926782,5.59420215225764,5.5949678,5.59568,5.5972216,5.6002515,5.6013104,5.6015752,5.60174987895673,5.60328401605256,5.603746],"lat":[52.164211,52.1642810422155,52.1644774868905,52.16476,52.1647932,52.1648874,52.1649375,52.1649904,52.1651489,52.1652481917834,52.1653136,52.1654001,52.1655718,52.1655991,52.1656303,52.1656387,52.1656489,52.1656536,52.1656536,52.1656509,52.1655611847869,52.1653365,52.1652659,52.1651854,52.1649952,52.1648235,52.1647841153467,52.1646603229862,52.1646368382027,52.1645758528279,52.164600555011,52.1649299161414,52.1652510408966,52.1656956713472,52.16633,52.1665298,52.1666815,52.1668019,52.1674349,52.1688333,52.1691483,52.1698248889495,52.1699471,52.1703075,52.1704183,52.1707692165433,52.1711111237073,52.1713888812492,52.17187,52.1719664,52.17316,52.1755201175585,52.1757695841572,52.1758967094459,52.1771441,52.1776583,52.1782238,52.1784941,52.1786814749016,52.1792584139507,52.1797579126824,52.1798794,52.1800544,52.1802438,52.1803549,52.1804527,52.181612,52.1820625441599,52.1822889,52.1825224,52.1830591,52.1841207,52.1844906,52.1845919,52.184664912052,52.1853197809799,52.185609]}]],[[{"lng":[5.603746,5.6041512,5.6042116,5.6046085,5.6046967,5.6047688,5.604888,5.6050578,5.605205,5.6053548,5.6055162,5.60559803390469,5.605745,5.6060101,5.6063275,5.6065005,5.6068084,5.60721222040464,5.60728],"lat":[52.185609,52.1859535,52.1860118,52.186372,52.186439,52.1864886,52.1865655,52.1866568,52.1867276,52.1867896,52.1868471,52.1868718504766,52.1869163,52.1869764,52.1870149,52.1870253,52.1870438,52.1870873284708,52.187098]}]],[[{"lng":[5.60728,5.60747735505511,5.60811483729499,5.6083842,5.6085606,5.60866669740937,5.6090069,5.609539,5.6099118,5.6103365,5.611319,5.6115204,5.6128729,5.613279,5.6138503,5.6143917,5.61597,5.6160462,5.6167049,5.6168475,5.6171135,5.617395,5.6175351,5.6184979,5.6190895,5.6197823,5.62008513547897,5.62148450478424,5.6223574390847,5.62313752405309,5.6234164,5.6238927,5.6242649,5.62487,5.6257249,5.62636566668577,5.62766257250407,5.628487,5.6291506,5.63348352337778,5.6347544,5.636523,5.6383503,5.64057253562166,5.6454289,5.64827,5.6489981157301,5.6495393,5.6499022,5.6500828,5.6502304,5.6504214,5.650613,5.6508879,5.6511748,5.6515298,5.65165943839825,5.6524077,5.65444649033587,5.6567981,5.65775,5.6612291858702,5.6613856,5.6632185,5.67412988946895,5.674221],"lat":[52.187098,52.1871237115125,52.1872105368185,52.187254,52.1872801,52.1872971925575,52.187352,52.187445,52.1875346,52.1876573,52.1879787,52.1880446,52.1884356,52.1885294,52.1886329,52.1887343,52.18903,52.1890429,52.1891548,52.189179,52.1892232,52.1892705,52.1892925,52.189444,52.1895263,52.1896,52.1896217892807,52.1897396915441,52.1897490801589,52.1898256957917,52.1898579,52.1899268,52.1899973,52.19015,52.1903857,52.1905623347935,52.1909198398267,52.1911471,52.191325,52.1925233281339,52.1928748,52.1933939,52.193891,52.194500643718,52.1958329,52.1966423,52.1968451379097,52.1969959,52.1971022,52.1971683,52.1972186,52.1972794,52.1973307,52.1974085,52.1974758,52.1975517,52.1975875688236,52.1977946,52.1983485564671,52.1989875,52.19925,52.2002314761799,52.2002756,52.2007973,52.2039893042728,52.20401]}]],[[{"lng":[5.674221,5.6743279465509,5.68066238019696,5.6844151,5.6899545,5.70292,5.70630462097919,5.70744683375328,5.7079514,5.70821714788273,5.709842,5.71565951008173,5.7175104,5.71863948950183,5.7227887,5.7244057,5.72489939589181,5.7263447,5.72706,5.72732344946544,5.7275443,5.72766186394665,5.728049,5.7302764,5.73405807976393,5.7388444,5.74231305764805,5.74427406894548,5.7482414,5.7538527,5.75614010485019,5.7574511,5.763434,5.7701312,5.77520833332131,5.77859162124184,5.77958402508763,5.7822704,5.7869651,5.7876569,5.789986,5.7917022,5.79360368702534,5.7945557,5.7947268,5.7951266,5.7983906,5.8083489189652,5.8149295,5.82452106582545,5.83641968998828,5.845698,5.845698,5.8464609,5.8475966,5.8529422,5.8532114,5.8546955,5.8552871,5.8557632,5.8562427,5.8566238,5.8569844,5.8572149,5.8592188,5.859282,5.8593359,5.8594158,5.8596408,5.8597839,5.8599772,5.8601743,5.8604494,5.8606129,5.8613807,5.8618335,5.8624031,5.864158,5.8648771,5.8649462,5.8702695,5.871737,5.8718182,5.87216105522726,5.87282258178125,5.873604],"lat":[52.20401,52.2040423557072,52.2058455593638,52.2069138,52.2085176,52.21228,52.2132988722876,52.2136256803993,52.2137539,52.2138220310709,52.2142386,52.215901156629,52.2164301,52.2167527685368,52.2179385,52.2184006,52.2185416820864,52.2189547,52.21908,52.219114814725,52.219144,52.2191552043356,52.2191921,52.2192783,52.2194379006667,52.2196399,52.2197689091367,52.2198418444981,52.2199894,52.2201982,52.2202778498374,52.2203235,52.2205423,52.2207873,52.2209653763554,52.2210819197056,52.221105784397,52.2211982,52.2213714,52.221397,52.2214829,52.2215462,52.2216163687972,52.2216515,52.2216578,52.2216726,52.221793,52.2221604717594,52.2224033,52.22275251782,52.2231928185874,52.2227429,52.2227429,52.2226413,52.2224259,52.2215283,52.2215128,52.2214275,52.2213562,52.2212547,52.2210565,52.2208686,52.2205532,52.2204108,52.2194827,52.2193962,52.2193073,52.2192174,52.2190511,52.2189748,52.2189053,52.2188598,52.2188336,52.218829,52.2184612,52.2183657,52.2183876,52.2184485,52.2184735,52.2184759,52.2181667,52.2182067,52.218221,52.2183381435358,52.2184314268048,52.218511]}]],[[{"lng":[5.873604,5.8738132473525,5.87422171751104,5.8752612,5.8762182,5.8766312,5.8770229,5.8772828,5.8777599,5.8781232,5.8783258,5.8785088,5.878713,5.8788205,5.8789275,5.8791459,5.8793628,5.87985657031861,5.8800467,5.8807421,5.881853,5.8830839,5.8836612,5.8842321,5.88625,5.88713742223773,5.8918988,5.8924106,5.8926568,5.892883,5.8933464,5.8951832,5.8956317,5.8960383,5.8965561,5.8970484,5.8979327,5.8989375,5.89933,5.8996614,5.9001391,5.9008239,5.9019019325541,5.9031079,5.9034652,5.9038477,5.9047712,5.9051425,5.9098973,5.910065,5.9104286,5.9107712,5.9110648,5.9138633,5.9144678,5.9149556,5.9154173,5.9169469,5.917205,5.9192278249302,5.9211657,5.9226344,5.9232036,5.9238413,5.9243499,5.9245108,5.92477428216241,5.9251593,5.9258299,5.9264293,5.9276873,5.928526,5.92869,5.9322725,5.9327057,5.9329779,5.9331603,5.9361069,5.9363814,5.9369247,5.9369555,5.9370146,5.937282,5.9374451,5.9375393,5.9376078,5.9387019,5.9400796,5.940571,5.9407725,5.9409516,5.9411507,5.9413526,5.9418177,5.94274718943098,5.94333,5.9445868,5.9454692,5.9460781,5.9461955,5.9462173,5.9464208,5.9476243,5.9480855,5.9496482,5.9503432054888,5.95197296911431,5.95247113654485,5.9536806,5.9542246624642,5.95486642477826,5.9551835,5.955466,5.9556521,5.9562905,5.9570978,5.95725171042723,5.9575368,5.95811458022415,5.95841857058313,5.9593475,5.9596265,5.960179,5.9603335,5.9607933,5.96093758152875,5.9611579,5.96134563215837,5.9614824,5.96178327274916,5.962314],"lat":[52.218511,52.2185334619811,52.2185738083125,52.2186306,52.2186323,52.2186442,52.2186817,52.2187223,52.2188109,52.218895,52.2189485,52.2189983,52.2190509,52.2190763,52.2190962,52.2191283,52.2191454,52.2191785394369,52.2191913,52.2192183,52.2192988,52.2194051,52.2194739,52.2195701,52.21986,52.2199432001804,52.2203896,52.2204424,52.2204603,52.2204648,52.2203945,52.2200045,52.2199027,52.2197899,52.2196114,52.2194109,52.2189838,52.2185667,52.21848,52.2184241,52.2184203,52.2185329,52.2187473743063,52.2189873,52.2190642,52.2191249,52.2192009,52.2192302,52.219366,52.2193667,52.2193656,52.2193641,52.2193576,52.219443,52.2194877,52.219524,52.2195584,52.2197396,52.2197702,52.2200189233024,52.2202572,52.2204537,52.2204947,52.2204895,52.2204882,52.2204873,52.2204878688127,52.2204887,52.2204788,52.2204722,52.2204624,52.2204491,52.2204496,52.2204164,52.2203966,52.2203809,52.2203704,52.2201259,52.2201045,52.2200652,52.2200619,52.2200587,52.2200247,52.2200027,52.2199919,52.2199842,52.219892,52.219785,52.2197716,52.2197539,52.2197406,52.2197258,52.2197059,52.2196662,52.2195947812224,52.21955,52.2194048,52.2192678,52.2191737,52.2191543,52.2191486,52.2191197,52.2189174,52.2188136,52.2182426,52.2179994723011,52.2174293411091,52.2171976035894,52.2169075,52.2167566090307,52.2166522502621,52.2166249,52.2165999,52.2165845,52.2165269,52.2165356,52.2165399122971,52.2165479,52.2165173441932,52.2164867586435,52.2161026,52.2160254,52.2159334,52.2159055,52.2156976,52.21562553847,52.2155155,52.2154120015266,52.2153366,52.2149978381195,52.214558]}]],[[{"lng":[5.962314,5.96237252785092,5.96247144896742,5.9632041,5.96375637478516,5.9639136,5.96428505804088,5.9645958,5.9646566,5.9646733,5.9647058,5.9647259,5.964759,5.9648002,5.9648186,5.9649149,5.96498905754663,5.9650823,5.9655347,5.9660312,5.96620166564578,5.96655604829276,5.967326,5.9675155,5.9676243,5.9677299,5.9682965,5.9684846,5.968815,5.969086,5.96936400432848,5.9695311,5.9700462,5.9701213,5.9703887,5.9704530659373,5.9704727,5.9710505,5.9716382,5.9718208,5.97242,5.9726238,5.9731497,5.97348358497969,5.9748607,5.9751608,5.9753059,5.975731,5.9758501,5.9764821,5.97691042546302,5.97723541819497,5.9777949,5.9782715,5.9783922,5.9797602,5.9807518,5.9811007,5.98241,5.98389,5.98421,5.9844053,5.98469933519465,5.98495826712761,5.98650504544944,5.98744877008195,5.9879275,5.9883002,5.9892823,5.990325,5.9904005,5.9906269,5.9907977,5.9908521,5.9913443,5.99152298246275,5.9918401,5.9953201,5.99578428883615,5.9958984,5.9960996,5.9962687,5.9978783,5.9989510126651,5.99948037152046,6.00220784893239,6.00397817406582,6.00661,6.0087118,6.0089243,6.0093449,6.0116019065871,6.0129370923115,6.0138495,6.0152339,6.0158624,6.0160877,6.0171016,6.01722523226031,6.01749641735029,6.0176172,6.0205163,6.0216535,6.0221514,6.0225036,6.0228537,6.0231068,6.0235725,6.0240684,6.0267982,6.0304618,6.0310483,6.0315415,6.0320015,6.0324342,6.03304228661937,6.0332257,6.0361811,6.0363998,6.0378369,6.0384206,6.03895001934169,6.03917208157601,6.0397969,6.0399463,6.0403312,6.0407422,6.0409986,6.04114240188394,6.04159179419829,6.0422404,6.0440214,6.0445993,6.0451427,6.0452862683921,6.0454094,6.0455603,6.0456995,6.0458028,6.0461083,6.0463621,6.04659,6.04667351306745,6.04723,6.0475406,6.047764,6.0483707,6.0484203,6.0485699,6.0487103,6.0488282,6.0489768,6.0491014,6.04935482890067,6.049419,6.049664,6.0498323,6.050029,6.0502616,6.0504816,6.0507825,6.0511115,6.0514548,6.0517303,6.0523052,6.0548486,6.0549776,6.0550848,6.0553506,6.0562222,6.0562855,6.05677455101729,6.0584926,6.05944739776883,6.06130612588343,6.0636356,6.0644314,6.06839780867329,6.0715963,6.0721695,6.0727478,6.0732835,6.0737767,6.0741393616406,6.0743316,6.07502,6.0761996,6.0769223,6.0776502,6.0798778,6.0818045,6.0821971,6.0823323,6.0825527,6.0829057,6.0833342,6.083864,6.08503516339262,6.08570560564423,6.08656322530782,6.08775002453007,6.0894928,6.08995048912132,6.0903009445426,6.09113664593178,6.09147362229839,6.09177016150101,6.09226888652359,6.09271369532751,6.09346852238871,6.09496469745645,6.09617781237624,6.09724265769472,6.09807835908391,6.09861752127048,6.09930495305836,6.10093591867275,6.10227034508452,6.104787],"lat":[52.214558,52.2145046536571,52.2144078980567,52.2138072,52.2133545610687,52.2132257,52.2129212163575,52.2126665,52.2126161,52.2126023,52.2125762,52.2125587,52.2125313,52.2124972,52.2124819,52.2124022,52.2123393389098,52.2122603,52.2118769,52.2114538,52.2113810494735,52.2113717668998,52.2114399,52.21146,52.2114708,52.2114786,52.2114944,52.2115067,52.2115121,52.2115155,52.2115132514815,52.2115119,52.2115109,52.2115099,52.2115089,52.2115106624007,52.2115112,52.2115196,52.2115346,52.2115446,52.2116229,52.2116495,52.2117075,52.2117476209116,52.2119131,52.211949,52.2119621,52.2120099,52.2120246,52.2121032,52.2121411506217,52.2121835011769,52.2122467,52.2123005,52.2123157,52.2124612,52.2125702,52.2126228,52.21298,52.21339,52.2135,52.2136317,52.2138244088296,52.2139193611463,52.2140741887731,52.2141865083203,52.2143007,52.2144302,52.2148849,52.2153662,52.2154041,52.2155093,52.2155881,52.21561,52.2158372,52.2159233699282,52.2160763,52.2177317,52.2179411991466,52.2179927,52.2180835,52.2181632,52.2189214,52.2194295925894,52.2196968809796,52.2209546821642,52.2217820167807,52.2230715,52.2240508,52.2241468,52.2243463,52.2253188614205,52.2259416470477,52.2263668,52.2270119,52.2273049,52.2274089,52.2278797,52.2279371042354,52.2280630191105,52.2281191,52.2294653,52.2299982,52.2302104,52.2303375,52.2304496,52.230518,52.2306171,52.230699,52.2310643,52.2315488,52.2316169,52.2316517,52.2316622,52.2316483,52.2315939064123,52.2315775,52.2312823,52.2312593,52.2311228,52.231075,52.2310573788893,52.2310573788893,52.2311126,52.2311263,52.2311853,52.2312771,52.23134,52.2313964293704,52.231545130397,52.2317249,52.2323113,52.2325251,52.2327364,52.2327998134494,52.2328542,52.2329222,52.2329908,52.233046,52.2332209,52.2333995,52.2336,52.2336809037754,52.23422,52.2345353,52.2347498,52.2353291,52.2353732,52.2355073,52.2356091,52.2356952,52.235813,52.2358877,52.2360244686729,52.2360591,52.2361826,52.2362535,52.2363273,52.2364047,52.2364675,52.236538,52.2366065,52.2366603,52.2366965,52.2367484,52.2369706,52.23698,52.2369867,52.2370085,52.2370814,52.2370869,52.2371338752202,52.2372989,52.2373817373534,52.237542998407,52.2377451,52.2378203,52.2381758159007,52.2384625,52.238515,52.2385419,52.2385025,52.2384368,52.2383851686056,52.2383578,52.2381895,52.2378707,52.2377295,52.2376264,52.2373886,52.2371829,52.2371824,52.2371827,52.2371901,52.2372188,52.2372677,52.2373475,52.237461291674,52.2374868266179,52.2374540017519,52.2372798141903,52.2367923,52.2366924006255,52.2370225828112,52.2379140624434,52.2382524898368,52.2384340839596,52.2386404400155,52.2388302867395,52.239028386803,52.23940807612,52.2397795081781,52.2401096673977,52.2404068085954,52.2405553784481,52.2407369631483,52.2410588614725,52.2412982202512,52.241551]}]],[[{"lng":[6.104787,6.10661958672287,6.10835389175635,6.10947714631171,6.10998036435251,6.11020501526358,6.11040270806532,6.11091491214257,6.1111934792723,6.1115708928029,6.11237065004631,6.11289184016,6.11315243521685,6.1134579604559,6.11384436002295,6.11417684337133,6.11458121501126,6.11520125152582,6.11582128804038,6.11637842229984,6.11720513765258,6.11778024398493,6.11877769403009,6.11973021389304,6.12078158015685,6.12136567252564,6.1235113383385,6.124177],"lat":[52.241551,52.2418938603438,52.2422350085743,52.2424826145184,52.2426036658104,52.242719214463,52.2428457674039,52.2433079576872,52.2435940730691,52.2438966931002,52.2444028892659,52.2447440182075,52.2449365898389,52.2452777146774,52.2457013660677,52.2459654583942,52.2461855341321,52.2464221143329,52.2465926713696,52.2466807005513,52.2467577259421,52.2468622601871,52.2470768297079,52.247362920788,52.2476655151154,52.2477865522684,52.2482083458276,52.248354]}]],[[{"lng":[6.124177,6.1286393698019,6.13220183402726,6.13627550388137,6.13911908519129,6.14074855313294,6.143585],"lat":[52.248354,52.2490934879693,52.2497683358589,52.250384492358,52.2507170177068,52.2512353610736,52.252153]}]],[[{"lng":[6.143585,6.1451404784444,6.1495211712103,6.14959530601096,6.14989184521357,6.15115887635202,6.15142171791797,6.15177217333925,6.152218],"lat":[52.252153,52.252266070085,52.2542918133948,52.2548735270642,52.2554263542613,52.2552448296728,52.2554016009521,52.2557110163246,52.256011]}]],[[{"lng":[6.160268,6.15995438074095,6.15880571255127,6.15832046658336,6.15730069185391,6.15668655242577,6.15625058925147,6.15590560969615,6.15553409325197,6.1550867571253,6.15470386772874,6.15430202341156,6.15386226925313,6.153217801952,6.1528241,6.15282074057177,6.15266158826404,6.15231040613693,6.152218],"lat":[52.251815,52.2519140825941,52.2524966010379,52.2527565271183,52.2533181479127,52.2537312363552,52.2540863036835,52.2543903133204,52.2546896795526,52.254977440538,52.2552443139769,52.255564559984,52.2559033684598,52.2556945142259,52.2555708,52.2555733962065,52.2556963908494,52.2559677870289,52.256011]}]],[[{"lng":[6.270168,6.2697367,6.2695756,6.2693933,6.2691863,6.2683686,6.267671,6.2670726,6.2669477,6.2664488,6.2661794,6.2659503,6.2657469,6.2656699,6.2654872,6.26531,6.2649777,6.2648177,6.2646902,6.264465,6.2640905,6.26369,6.263525,6.2633174,6.2631145,6.26288,6.2616687,6.2616083,6.2613369,6.26059,6.2602326,6.2599124,6.2598323,6.2597495,6.2596627,6.2595226,6.2593302,6.259052,6.2584881,6.25820044984311,6.2571581,6.2568427,6.256502,6.25586,6.2534,6.25228,6.25212,6.25137,6.2504983,6.24993257947131,6.24952,6.24928981633977,6.2490827,6.2488537,6.248662,6.2484827,6.2482856,6.248157,6.24814824193551,6.2479467,6.2475489,6.24620256892231,6.2455555,6.2445601,6.2435865,6.2432693,6.2427516,6.2422257,6.2379779,6.2358629,6.2348538,6.2347714,6.2346827,6.2337813,6.2335156,6.233277,6.23270728973104,6.23175654772177,6.22961171826767,6.2291309,6.224941,6.2247554,6.22335794817331,6.2226556,6.2198252,6.2197,6.21949997719115,6.2193329,6.2191175,6.2189057,6.2188577,6.2187575,6.2182069077128,6.21754565696622,6.21726805037639,6.2162646,6.2154205,6.215279,6.2147678,6.2145905,6.2144956,6.2143719,6.2134481,6.2110006,6.21070860564365,6.20778099478388,6.20498031295481,6.20438191515496,6.2038455,6.202902,6.19980979025107,6.199069,6.19888313438474,6.1986538,6.1985443,6.1984063,6.1982457,6.1981675,6.1979539,6.1978638,6.1976258,6.1975605,6.1974326,6.19733367287868,6.19700899696878,6.19663995975585,6.19617027603031,6.19368766205244,6.18899082479701,6.18268364333973,6.17976489447386,6.17882552702277,6.178154550272,6.1771145363083,6.17372610371688,6.16976734088731,6.16674794550882,6.1657750292202,6.1649437,6.1648446,6.1646838,6.1644017,6.1642254,6.164084,6.16377863123193,6.16364891456955,6.1635864,6.16352681934375,6.1634869,6.1634406,6.1633793,6.1632645,6.1632287928463,6.16294167623093,6.16257774175499,6.16215315153307,6.16185745477137,6.16163757769216,6.16134188093046,6.16083310569068,6.16056852016909,6.16037897096287,6.1602424955344,6.16008137870911,6.16001882747106,6.1599316348362,6.15989372499496,6.15992784385208,6.16009843813767,6.16032210620101,6.16046237261361,6.16048890950248,6.16045858162949,6.160268],"lat":[52.303657,52.3028831,52.302669,52.302479,52.3022937,52.3017237,52.3013154,52.3009905,52.3009234,52.3006706,52.3005301,52.3003988,52.3002721,52.3002209,52.3000984,52.2999854,52.2997529,52.2996255,52.2994959,52.2992695,52.2989001,52.29854,52.2983851,52.2982284,52.298089,52.29794,52.2970026,52.2969558,52.296775,52.29625,52.2959763,52.29568,52.295597,52.2955173,52.295455,52.2953703,52.2952774,52.2951629,52.2949341,52.2948187593196,52.2944008,52.294274,52.2941105,52.29378,52.29249,52.29189,52.29178,52.29102,52.2902732,52.289831866729,52.28951,52.2893125572426,52.2891349,52.2889513,52.2888049,52.2887139,52.2886525,52.2886183,52.2886166799878,52.2885794,52.2885152,52.2883395300995,52.2882551,52.2881338,52.2880091,52.2879722,52.2879467,52.2879169,52.2876931,52.2875899,52.2875308,52.2875294,52.2875334,52.2874835,52.2874685,52.2874519,52.2874031356647,52.287293280488,52.2866468108126,52.2865064,52.2853529,52.2853018,52.2849081477903,52.2847103,52.2839196,52.2838682,52.2837733922156,52.2836942,52.2835703,52.2834372,52.2834034,52.2833309,52.2828295588279,52.2822382475278,52.2819868664066,52.2810782,52.2803185,52.2801865,52.2797239,52.2795808,52.2795244,52.2794597,52.2790911,52.2780998,52.2779815166586,52.2767955598667,52.2756609915272,52.2753960433311,52.2752056,52.2748413,52.2735743292765,52.2732708,52.2731814485292,52.2730712,52.2730045,52.2729137,52.2728066,52.2727367,52.2725451,52.2724425,52.2721665,52.2720832,52.2719126,52.2716843800294,52.270756887011,52.2702025852439,52.2697509268277,52.2684780465336,52.2678210617598,52.2667739721535,52.266363342028,52.2661580255391,52.2656857940064,52.2646797187518,52.2626674997566,52.2601623648371,52.2583347591011,52.2573079812638,52.2567861,52.2567233,52.2566225,52.25645,52.2563339,52.256248,52.2560752483866,52.256001865555,52.2559665,52.2559157217315,52.2558817,52.2558277,52.2557619,52.2556197,52.2555758570798,52.255181656879,52.2546432663156,52.2543090895756,52.2541327175033,52.254067738142,52.2540306070356,52.2539969645991,52.2539795517136,52.2539424205335,52.2538960065146,52.2538089800982,52.2537057085292,52.2535246926148,52.2534144582025,52.2532276392571,52.2529445084238,52.2526729796157,52.2524687517247,52.2523295048962,52.2521693705031,52.251815]}]],[[{"lng":[6.270168,6.27116311087526,6.28136195748705,6.28864205523296,6.292892],"lat":[52.303657,52.3037458829003,52.3040741102572,52.3054690493792,52.306799]}]],[[{"lng":[6.292892,6.30582262663077,6.30803749419327,6.30846161776906,6.31750958738607,6.3193710186354,6.32464900091199,6.32738224173379,6.33008406599443,6.33385405333485,6.33561338076038,6.33746695786941,6.34363507604582,6.34831614366018,6.35318571064156,6.35525920367879,6.3563692555068,6.35717561391017,6.35796102793942,6.35861030353694,6.35906060758038,6.36162629340928,6.36252690149616,6.36659011007417,6.38026678637025,6.39673822758708,6.39843472189027,6.40277020733175,6.41038086927523,6.41582378849796,6.4166249108078,6.41846277963625,6.41971158794277,6.42053627267348,6.42159658161298,6.42263332813159,6.42312813897002,6.42854749577188,6.42927793081908,6.43856152464487,6.44563025090816,6.44902323951453,6.45592702883168,6.4574821486096,6.45922576775454,6.46059238816545,6.46191188373459,6.46311356719935,6.46393825193007,6.46497499844868,6.46681286727714,6.46860361126384,6.47225578649987,6.47423],"lat":[52.306799,52.3041024614003,52.303670227613,52.3036270040022,52.3047940266835,52.3053847301424,52.3071207763975,52.3073320747525,52.3071399853805,52.3067942224108,52.3070247313573,52.3073128658528,52.3078891292179,52.3084525794775,52.3094514049396,52.3100660564585,52.3104053913547,52.3105526490852,52.3107831384626,52.3111032606069,52.3114489899224,52.3149317420348,52.3152134232415,52.3161288747827,52.3126974344136,52.3118963532466,52.3124293419309,52.3125589868566,52.3145468281689,52.3160304486461,52.3160592563185,52.3158720061128,52.3153534629471,52.3148493179335,52.3146332540269,52.3141002918809,52.3140138649822,52.3141435052669,52.314273145172,52.3173267753398,52.3180181340507,52.3182485845542,52.317917311578,52.3178308921329,52.3178596986334,52.3180037308544,52.3179893276533,52.3178885051151,52.3179749244477,52.31829179389,52.318954331756,52.3192999997484,52.3199625225201,52.320311]}]],[[{"lng":[6.47423,6.47455164988413,6.47601251997854,6.47746160886251,6.47856904264376,6.48101953441504,6.481797094304,6.48424758607527,6.48461280359887,6.4853785822774,6.48588517432627,6.48678054631961,6.48771126194428,6.48873622725246,6.48983187982327,6.49057409608091,6.49188181043962,6.49285965090604,6.49357830474281,6.49617017103935,6.49832613254965,6.49941000391002,6.50457017408222,6.50777466332158,6.50843441110615,6.50911379424146,6.50993062483188,6.512146],"lat":[52.320311,52.3201659582285,52.3196834720183,52.3194170221549,52.319056952223,52.3186104614386,52.318365609741,52.3181207566887,52.318063144009,52.3176238448593,52.3171053222189,52.3165147751456,52.3160034414362,52.3156577476941,52.3151536061466,52.3148943311146,52.3149375437254,52.3149519479197,52.3149663521092,52.3147142781155,52.3143541699198,52.3139580475205,52.3119269642569,52.310659290587,52.3101694978552,52.309600467141,52.3092547233878,52.308754]}]],[[{"lng":[6.521151,6.5210602997294,6.520834,6.5206807,6.5204773,6.520394,6.5200079,6.5195266,6.51918269540628,6.5191051,6.5189901,6.5177816,6.51758769162884,6.5168488906126,6.51611487659024,6.5150445,6.5145356,6.5135429,6.512146],"lat":[52.308009,52.3079748854129,52.3078754,52.307841,52.3078414,52.3078463,52.3078286,52.3078038,52.3077852789231,52.3077811,52.3077756,52.3077139,52.3077108229616,52.3077593161376,52.3078855786855,52.3081008,52.3082195,52.3084566,52.308754]}]],[[{"lng":[6.592483,6.59233130179254,6.59182240530042,6.591318,6.59115393963986,6.5907515,6.5903941,6.5894608,6.5892807,6.58892464025153,6.5887194,6.5883161,6.58725520304235,6.5857621,6.58566362223845,6.585656,6.5850214,6.5846479,6.5842291,6.5840938706453,6.5839018470529,6.5836801,6.5827609,6.5826716,6.5825398,6.5813441,6.5810542,6.5806738,6.5799885,6.579407,6.5790014,6.5786055,6.5782118,6.57772180455725,6.577655,6.577365,6.57685129029209,6.57304641922573,6.57156942596294,6.56812553221561,6.56600476856672,6.56459102331406,6.56335399621799,6.56232903090981,6.56089761384149,6.55939550951055,6.55821149786144,6.55674473716181,6.55584347456324,6.55411163662874,6.55170319471327,6.54844887375877,6.54792756839334,6.54588559190164,6.54510963219292,6.54447245829885,6.54414807099204,6.54373020250084,6.54250826572982,6.5407225092139,6.5356745,6.5346841,6.533273,6.5325827,6.5295124692325,6.52659957907593,6.52575821302515,6.5266643,6.5268658,6.526911,6.52686572547974,6.52681244447744,6.5267627,6.526292,6.5262571,6.525882,6.52559509103572,6.5252657693428,6.5248466,6.5246853,6.5243422,6.5243067012402,6.5242703,6.52424514870076,6.5242065,6.5241813,6.5240841,6.5240257,6.52397,6.5238797,6.5237754,6.52370416376345,6.52355851610069,6.5232899,6.5231896,6.5228952,6.5223999,6.5220519,6.52152246707192,6.521151],"lat":[52.357344,52.3572620697625,52.3569501240988,52.3565378,52.3564006153633,52.3560641,52.3557313,52.354948,52.3547846,52.3544615270482,52.3542753,52.3539956,52.3534211174614,52.3522348,52.3521736342682,52.3521689,52.3517716,52.3515596,52.3513328,52.3512595694258,52.3511555829344,52.3510355,52.3506048,52.350563,52.3505059,52.3499019,52.3497482,52.3495367,52.3491449,52.3487915,52.3484934,52.34815,52.3478904,52.3475602164574,52.3475152,52.3473217,52.3469639923389,52.3443483280745,52.3435845621717,52.3416965174964,52.3410032606667,52.3405389965175,52.3402042914327,52.3397292218011,52.3389734186864,52.3379584627545,52.3369434835276,52.336014865106,52.334827068937,52.332494576363,52.3309927320244,52.3297118898383,52.3294376378567,52.3283633641734,52.3275089126296,52.3259295275222,52.3252474312069,52.3248408337531,52.3241612493841,52.3235997826252,52.3228371,52.3226933,52.322477,52.3222474,52.3210895085777,52.320802837392,52.3201416199984,52.3174298,52.3169471,52.3168391,52.3160061632916,52.3154420117429,52.31531,52.313922,52.3137999,52.3125266,52.3120697732119,52.3115736214132,52.3110184,52.3108053,52.3103322,52.3102809021008,52.3102283,52.3101919528475,52.3101361,52.3100997,52.3099334,52.3098392,52.3097425,52.3096017,52.3094679,52.3093810175736,52.3092446455093,52.309114,52.3090627,52.3089208,52.3086804,52.3085094,52.3082438958902,52.308009]}]],[[{"lng":[6.665096,6.66479609925731,6.66436510176482,6.663634,6.663322,6.6632359,6.6629861,6.6627557,6.662604,6.6623645,6.66228427967779,6.6620867,6.6619443,6.6617999,6.6616876,6.6614516,6.66141652097895,6.66126936952313,6.66023082989508,6.65999473227879,6.65980585418576,6.65961482975077,6.65926281367296,6.6590917,6.6585677,6.658425,6.6582076,6.6580454,6.65786791189321,6.6578056,6.6576408,6.6574274,6.6565385,6.6562098,6.6547464,6.6540634,6.6539247,6.6537864,6.6535759,6.6533673,6.653231,6.6530314,6.6529489,6.650842,6.65063880452876,6.65031274906644,6.64947765664414,6.6484625,6.6474164,6.6458822,6.6442423,6.64239952911236,6.64036618844174,6.63885598881745,6.63839207128581,6.63728656567849,6.6369904481051,6.63522361325053,6.63483866040513,6.6334211,6.630629,6.6279743,6.62049368387042,6.62014145464934,6.61993462599233,6.61949207744331,6.61813247856892,6.61790856923791,6.6178736,6.61352088013236,6.60961931373806,6.60651994980324,6.6039413,6.6026097,6.60143235216434,6.60051959639382,6.59941209102983,6.5993068,6.5970246,6.596057,6.5956431,6.5943568,6.5938531,6.5934877,6.5934298,6.59309721696562,6.59284152867331,6.5927309,6.592483],"lat":[52.351456,52.35141398878,52.3512663681333,52.3509329,52.3507353,52.3506828,52.3505259,52.3504124,52.3503388,52.3502585,52.3502383149843,52.3501886,52.3501709,52.3501653,52.350167,52.3501956,52.3501999963276,52.3502184382858,52.3504215341959,52.3504661099127,52.3504962640487,52.3505093745361,52.3505109027384,52.3505062,52.3504861,52.3504709,52.3504335,52.3503934,52.350356540507,52.3503436,52.3503174,52.3502938,52.3502454,52.3502234,52.3501438,52.3501134,52.3501012,52.3500856,52.35004,52.3499723,52.3499229,52.34983,52.3497789,52.3481266,52.3479764937479,52.3477227314464,52.3475187716287,52.3474039,52.3473844,52.347371,52.3473613,52.3475275306512,52.3481485802054,52.3486128542698,52.3486068247678,52.3488057979013,52.3488540336776,52.3486430017677,52.3486550607612,52.3489816,52.3496303,52.3499946,52.3509859276075,52.3510351018789,52.3510566464706,52.3511193351458,52.3512789533365,52.351306444252,52.3513163,52.3519501083186,52.3542983086637,52.3564806536422,52.3565551,52.3566045,52.356669498529,52.356698401417,52.3566930732288,52.3566931,52.3567423,52.3567822,52.3568336,52.3570368,52.3571263,52.3571801,52.35719,52.3572455336486,52.3572882276122,52.3573067,52.357344]}]],[[{"lng":[6.665282,6.66550009942223,6.6657105,6.6657105,6.6658084,6.6658184,6.6658513,6.6659388,6.66606787336185,6.6662505,6.6663201,6.6663804,6.6664281,6.6664364,6.6664299,6.6663723,6.66626772960001,6.6661974,6.6659911,6.665882,6.6657258,6.66527752134428,6.6652236,6.665187,6.66517221886905,6.6651337,6.6650788,6.66505,6.66504669104278,6.66507619688389,6.665096],"lat":[52.35587,52.3558024161223,52.355694,52.355694,52.3555041,52.3554823,52.3554038,52.3552052,52.3549079227285,52.3544873,52.3543657,52.3542646,52.3541413,52.354124,52.3540307,52.3539136,52.3537765644901,52.3536844,52.3534287,52.3533261,52.3531901,52.3528534002161,52.3528129,52.3527471,52.3526888352669,52.352537,52.3520499,52.35177,52.3516935386517,52.3515110042808,52.351456]}]],[[{"lng":[6.894704,6.89441737886976,6.8940219,6.8935288,6.8933212,6.89312730854868,6.8930391,6.8929711705606,6.89292086380049,6.89283520804068,6.89266364467043,6.8925903,6.8923588,6.8921273,6.8918958,6.89176811042968,6.8916575,6.8915522,6.8914011,6.89126844499788,6.8911525425317,6.8908011,6.8905321,6.8894041,6.8888351,6.8885361,6.88844463552737,6.8883059,6.8880522,6.887755,6.8865408,6.88590934794338,6.885119701081,6.8842444,6.8838461,6.8835263,6.8832198,6.8831158,6.88296744385356,6.8828404,6.8825874,6.88197050144171,6.8819221,6.8815877,6.881526435092,6.88138555385213,6.88119516197346,6.88094868400451,6.88089999182094,6.87977167914924,6.87950863736476,6.87914805191443,6.87890766161421,6.87872736888904,6.87848135563841,6.8784399,6.8783283,6.87808399119714,6.8780564,6.877783,6.87768424655742,6.87753204505578,6.87712834412418,6.87657404991768,6.8762567,6.875916,6.8757293,6.8756028,6.8754467,6.8752657,6.8749448,6.8746307,6.87439048278333,6.87361370915443,6.8733640041222,6.8725432,6.8717761,6.8700283,6.8695903,6.8692802,6.8690584,6.8684066835443,6.86790802216554,6.866843,6.86676854500603,6.8664512,6.8658986,6.86581021773128,6.8630662,6.86289592203385,6.8625513,6.86205463940204,6.8614213,6.8604772,6.85997246816718,6.8591951,6.8577576,6.8576113,6.8571558,6.8570284,6.8567533,6.8565223,6.856058,6.8555614,6.85524089486331,6.8538625,6.8534417,6.8530452,6.8528214,6.8524654,6.8507854,6.84989,6.8495763,6.8490132,6.84872989082694,6.8486703,6.8484952,6.8481028,6.8477694,6.8473771,6.8460232,6.8456106,6.8452708,6.8450061,6.8445579,6.8432609,6.8426057,6.84252287803271,6.84188648220922,6.8412024,6.8408007,6.8388321,6.8383743,6.83826632140934,6.83805992915305,6.83778267636974,6.83744999336497,6.83723879022085,6.8355646,6.8347514,6.83464284266201,6.8336198,6.8332134,6.8328976,6.8315436,6.8300734,6.82982671719064,6.8279511,6.8266993,6.8257287,6.82564494386564,6.82509,6.8246778,6.8233886,6.82297,6.8224962,6.82238293892278,6.8190767,6.8179817,6.81760565539628,6.81411,6.81294,6.81152,6.81119237922499,6.8110391,6.81079,6.809156,6.8084982,6.80828,6.8080689,6.80752393151641,6.80743,6.8069232,6.80660999726326,6.80611769334155,6.80529,6.80475,6.80465766127169,6.80447,6.80397,6.80315499943576,6.80233154611307,6.80166389831385,6.80077554559368,6.79988719287351,6.79893961663867,6.79443862952313,6.79291855931306,6.79232632416628,6.79106288918648,6.7885469,6.7884924,6.78807576226385,6.7872092,6.787064,6.7859055,6.7858478,6.7853171,6.7851284,6.7848687,6.78286,6.77878328768204,6.7760431,6.7698081,6.76883573338767,6.7679353,6.7671937,6.76660104072589,6.76531573256301,6.7641778,6.7632861,6.7631362,6.7628513,6.7625451,6.76222103753827,6.76195453172222,6.76175712000663,6.76156957887682,6.7607503202571,6.7583715090842,6.75601243908286,6.75441357274338,6.75421,6.7529489,6.7524535,6.7521936,6.7519079,6.7516139,6.7502684,6.7483,6.7477563,6.74721,6.7466857,6.7457186,6.74478,6.74458120232302,6.74415,6.7436494,6.7433,6.7427556,6.7412966,6.7399899,6.7391482,6.738386,6.73743958350836,6.7356764,6.73174526746028,6.7304636,6.7297051,6.7258776,6.7255233,6.72318978306218,6.7200897,6.7190407,6.71873,6.7177413,6.7167898,6.7162241,6.715348796558,6.7144702,6.7138739,6.7134693,6.7133411,6.7130255,6.7120848,6.7107095,6.709259,6.7085906,6.7083196,6.7082065,6.7080196,6.70724433112564,6.7069733,6.7066367,6.7063132,6.7060888,6.7058618,6.7055174,6.7051606,6.7048959,6.7043399,6.7039767,6.7037311,6.703292,6.7030206,6.7028004,6.7025993,6.70207176859308,6.69986088796004,6.69684533883707,6.69639,6.6960311,6.6958057,6.6955883,6.6955074,6.6954424,6.695229,6.69364722796772,6.69136745522317,6.6912997,6.6907245,6.6906789,6.6906054,6.6905229,6.6904032,6.6901019,6.68991986829928,6.6898635,6.6897716,6.6896179,6.689484,6.6893257,6.68892098295802,6.68829563649127,6.6875495,6.6872827,6.686977,6.6861904,6.685739,6.685124,6.68493564192779,6.6848367,6.6845338,6.6840696,6.68405033300034,6.6836663,6.6833807,6.68318,6.68292,6.68277399762104,6.6826766,6.6818118,6.6816013,6.6813205,6.6811392,6.6808942,6.6803626,6.67967215039263,6.6795774,6.6795099,6.67938380710388,6.6791536,6.67907877212818,6.6790017,6.6788237,6.6786524,6.6781186,6.67738031263329,6.6770126,6.6765520996246,6.6762026,6.6758648,6.67575,6.6756262,6.675471,6.6753471,6.6740961,6.673688,6.67299323165023,6.6728871,6.6726834,6.672069,6.67202417875155,6.67192302502658,6.67183596804154,6.6717872,6.6716677,6.6716118,6.6715488,6.6714417,6.6713353,6.6708852,6.67068229938468,6.6705187,6.670419,6.6702834,6.6701419,6.6699634,6.6697014,6.6695848,6.66898570786532,6.6686058562474,6.66849639280712,6.66839666026884,6.6683773,6.6683453,6.6683185,6.6682967,6.6682214,6.6681655,6.6681405,6.6681256,6.66812222245899,6.6681214,6.6681204,6.6681546,6.6681896,6.6682264139776,6.66831,6.6683472,6.6683389,6.6682894,6.6681242,6.6680066,6.6679432,6.66786059749785,6.66774697413576,6.6677401,6.6676241,6.6674452,6.6673773,6.6672902,6.66715421330065,6.6670214,6.6669153,6.6665562,6.6661792,6.6657669,6.6653911,6.6651959,6.6651076,6.66499807991714,6.6649979,6.66499754767949,6.66495695277849,6.6649553,6.6649517,6.6649352,6.6649326,6.6649135,6.6648996,6.66488,6.6648212,6.66478,6.66474544357117,6.6647,6.6646985,6.664725,6.6649027,6.6649299,6.6650352499516,6.665282],"lat":[52.406696,52.4067962382318,52.406961,52.407225,52.4073581,52.4074822978295,52.4075388,52.4075763422981,52.4076041451183,52.4076442613151,52.4077143405976,52.4077443,52.4078149,52.4078827,52.4079575,52.4080302129126,52.4080932,52.4081782,52.4083681,52.4084871733746,52.4085447837804,52.40855,52.408543,52.4085245,52.4085204,52.4084994,52.408487559684,52.4084696,52.408422,52.4083402,52.4079603,52.407831085753,52.4078009786365,52.4079817,52.4081819,52.4083269,52.4084447,52.4084891,52.4085414610195,52.4085863,52.4086677,52.4088215305799,52.4088336,52.4089277,52.4089400449528,52.4089591328967,52.4089795130109,52.4089666990069,52.4089641675761,52.4089055083149,52.40898209668,52.4092832377727,52.4094613028429,52.4095529537018,52.4095746549478,52.4095716,52.4095613,52.409544946855,52.4095431,52.4095097,52.409494004564,52.4094698143186,52.4093932192245,52.4092840787095,52.4092278,52.409166,52.4091279,52.409089,52.4090361,52.4089521,52.4087951,52.408663,52.4085743384113,52.4083019030368,52.4082076905595,52.4078999,52.4076315,52.4064107,52.4060655,52.405821,52.4056462,52.4052091482831,52.4049148723416,52.4043486,52.4043174726144,52.4041848,52.4039786,52.4039462261971,52.4029411,52.402870760002,52.4027284,52.4025232314339,52.4022616,52.4018656,52.4016529764517,52.4013255,52.4006855,52.400618,52.4004246,52.4003784,52.4002786,52.4001965,52.4000512,52.3999175,52.3998342093746,52.399476,52.3993499,52.3992097,52.3991218,52.3989706,52.3982184,52.39781,52.397674,52.3974506,52.3973524456638,52.3973318,52.3972732,52.3971643,52.3970821,52.397002,52.3967595,52.3966732,52.3965896,52.396518,52.3963771,52.3959546,52.3957322,52.3957049514099,52.395486901831,52.395267,52.3951399,52.3946341,52.3945247,52.3944954709184,52.3944396018626,52.3943487455892,52.3941902658097,52.3940735203339,52.3931845,52.3927498,52.3927011912003,52.3922431,52.3920472,52.391895,52.391209,52.3904705,52.3903447946003,52.389389,52.3887563,52.388193,52.3881449915292,52.3878269,52.3876276,52.387049,52.3868567,52.3866307,52.3865735356196,52.3849048,52.384337,52.3841469272765,52.38238,52.38182,52.3813,52.3812147055469,52.3811748,52.38111,52.3807917,52.3806809,52.38064,52.3805978,52.3804887892669,52.38047,52.3803448,52.3803469582864,52.3804593036394,52.38061,52.38062,52.3806167021885,52.38061,52.38054,52.3803369999528,52.3801343827497,52.3794249219166,52.3788344387543,52.3785572704653,52.3784247111033,52.378629575404,52.3788344387543,52.3788223879953,52.3785331687927,52.37802,52.3780095,52.3779307961346,52.3777671,52.3777371,52.3774967,52.3774838,52.377365,52.377334,52.3772843,52.3769,52.3760753214798,52.375521,52.374261,52.3740914666443,52.373958,52.3738825,52.3738396466685,52.3737586367453,52.373739,52.3737643,52.3737646,52.3737495,52.3737152,52.3735558915683,52.3733570298562,52.3730135393354,52.3724470754482,52.372296418936,52.3718986832746,52.3716274978157,52.3715699182843,52.37156,52.3715073,52.3715138,52.3715226,52.3715064,52.3714631,52.3714141,52.37134,52.3713319,52.37139,52.371486,52.3716995,52.3718875,52.3719261551405,52.37201,52.3720547,52.3720602,52.3720163,52.3717962,52.3716094,52.3715749,52.3716364,52.3718053486596,52.3721201,52.3728228008135,52.3730519,52.3731926,52.3738797,52.3739457,52.3743653715152,52.3749229,52.3751204,52.37519,52.3753847,52.3756016,52.3757449,52.3759573009602,52.3761705,52.3762798,52.3763207,52.3763315,52.3763531,52.3764135,52.3765158,52.3766231,52.3766726,52.3766926,52.3767124,52.3767452,52.3769717864804,52.377051,52.3771322,52.377172,52.3771959,52.3771975,52.3771752,52.3771204,52.3770804,52.3769932,52.3769259,52.3768783,52.3767769,52.3767132,52.37665,52.3765922,52.3764369354645,52.3757791639033,52.3749406447355,52.3748,52.3746942,52.3746318,52.3745707,52.3745501,52.3745345,52.3744786,52.3739866386002,52.3734645845256,52.373453,52.3733556,52.3733479,52.3733354,52.3733215,52.3733012,52.3732575,52.3732299356386,52.3732214,52.3731993,52.3731473,52.3731036,52.3730581,52.3728190994418,52.3721689824097,52.3715755,52.3713674,52.3711294,52.3705992,52.3703146,52.3699268,52.3698078715972,52.3697454,52.3695562,52.3691983,52.369183175003,52.3688817,52.3686576,52.3685,52.3682785,52.3681550519245,52.3680727,52.3674154,52.3672617,52.3670938,52.3669899,52.3668583,52.3666058,52.3663367252527,52.3662998,52.3662733,52.3662198618684,52.3661223,52.3661018073158,52.3660807,52.366058,52.3660422,52.3660363,52.3660485825393,52.3660547,52.3660576562988,52.3660599,52.3660626,52.3660538,52.3660362,52.3660057,52.3659696,52.3655945,52.3654805,52.3652863570302,52.3652567,52.3652224,52.365161,52.365153110955,52.365135306754,52.3651199837331,52.3651114,52.3650785,52.365058,52.3650371,52.3649988,52.3649493,52.3647026,52.3645767076482,52.3644752,52.364422,52.3643669,52.3643258,52.3642911,52.3642713,52.3642701,52.3642695641927,52.3642011555693,52.3641146530851,52.363912953729,52.3638132,52.3636533,52.3634207,52.3633026,52.3627737,52.3624068,52.3622611,52.3621718,52.3620792393162,52.3620567,52.3619492,52.3618235,52.3617069,52.3615733121959,52.36127,52.361117,52.3609748,52.3608749,52.3605612,52.3603872,52.3602934,52.3601712656751,52.3600032640066,52.3599931,52.3599106,52.3598169,52.3597839,52.3597534,52.3597476327073,52.359742,52.3597433,52.3597216,52.3596772,52.3596249,52.3595582,52.3595276,52.3592988,52.3589630515621,52.3589625,52.3589604166834,52.3587203731464,52.3587106,52.3586595,52.358559,52.3585037,52.3582975,52.3581713,52.35814,52.3580283,52.35795,52.3578463308734,52.35771,52.3574804,52.3573691,52.3567545,52.3566394,52.3562108806459,52.35587]}]],[[{"lng":[6.897136,6.8970109829286,6.89689404410255,6.896395,6.89635537686017,6.89620861216206,6.89613205589342,6.89607661579888,6.89589321104987,6.89588,6.8957502,6.89565597132525,6.895459,6.8952656,6.89522671234751,6.894704],"lat":[52.408124,52.408046582215,52.4080503448529,52.4081581,52.4081691370088,52.4082100182291,52.4081951922064,52.4081538790251,52.4079170587383,52.4079,52.407755,52.4076529083964,52.4074395,52.4072212,52.4071855793584,52.406696]}]],[[{"lng":[6.902075,6.9007565,6.9004643,6.9000874,6.899557,6.8992537,6.8990803,6.89894,6.89876,6.8986059,6.898139,6.8978939,6.8978555,6.8977866,6.8977,6.8976516,6.8975472,6.8972065,6.89713870157602,6.897136],"lat":[52.410181,52.4099216,52.4098588,52.4097958,52.4097592,52.409745,52.4097133,52.40967,52.40959,52.4094968,52.4090907,52.4088901,52.4088743,52.4088613,52.40879,52.4087303,52.4086145,52.4082393,52.4081491474369,52.408124]}]],[[{"lng":[7.050764,7.05050739748131,7.0496431,7.0491232,7.048786,7.0483738,7.0474474,7.0463985,7.0453643,7.04493762438921,7.0445463,7.0438701,7.0423656,7.0421765,7.0408707,7.0401955,7.0393501,7.0382744,7.0376207,7.03659745274733,7.0363746,7.0358849,7.0354799,7.03507132139092,7.03441435484786,7.0340638,7.0323494,7.0319045,7.0317616,7.0313424,7.0309245,7.0305219,7.0300966,7.0296589,7.0277659,7.0275028,7.0257356,7.0248304,7.022825,7.02236213509019,7.0218635,7.0215546,7.0214824,7.0213896,7.0200954,7.0199455,7.01984639212486,7.0196633,7.0193678,7.015136,7.0142288,7.0128113,7.0127123,7.0126441,7.0125336,7.0117169,7.0073966,7.0049502,7.0038945,7.0036309,7.0033583,7.0030983,7.0028521,7.0019091,6.999313,6.99879,6.99754,6.9964443,6.9953148,6.9947208,6.99443,6.9943552,6.9942826,6.9941309,6.993986,6.9937672,6.9933892,6.9932008,6.99294,6.99227,6.9919992,6.99168,6.9914347,6.9910822,6.9908246,6.99074,6.9903213,6.98989,6.98964,6.9894456,6.98926,6.9883247,6.9880355,6.9877092,6.98728,6.9866421,6.9861532,6.9856187,6.9852827,6.98438,6.9839,6.98335,6.9830874,6.982846,6.9822058,6.98188,6.98134,6.98075,6.98051,6.97935,6.97835,6.97793,6.97658,6.97579,6.97505,6.97473,6.97442,6.97408,6.97382,6.97231,6.971777,6.9694111,6.9691492,6.9689464,6.9686838,6.96838,6.9678694,6.9677648,6.9673731,6.9673231,6.96672,6.9658846,6.9657979,6.9657,6.9648503,6.9647221,6.9644385,6.9643326,6.96254,6.9615,6.96126,6.9610511,6.9605189,6.95887701843502,6.95767,6.9569708,6.9553033,6.9546525,6.9543011,6.9539488,6.95340934879769,6.95277763130779,6.95240254904816,6.95094170235277,6.94894784402528,6.9437303,6.9434426,6.9431741,6.9428985,6.9425551,6.9423723,6.9421115,6.9418621,6.9416626,6.9414709,6.9412024,6.9405281,6.9402144,6.9399961,6.9397506,6.9395188,6.9393483,6.9391784,6.9389814,6.9385974,6.938157,6.93799971484078,6.9363039,6.9361125,6.9359125,6.935674,6.9341505,6.9337522,6.9333743,6.9330876,6.9327477,6.9322675,6.9319035,6.9316484,6.9314741,6.931322,6.9310536,6.9306509,6.9304643,6.9302419,6.929976,6.9297335,6.929418,6.929116,6.9288594,6.9286727,6.9284421,6.9282047,6.9275079,6.9271822,6.9269753,6.9267994,6.9264415,6.9262718,6.9260615,6.925777,6.9252972,6.9227329,6.9225564,6.9222235,6.9220378,6.9219506,6.9216421,6.9214284,6.9213481,6.920573,6.9192981,6.9174773,6.9171104,6.9167341,6.9161202,6.9150249,6.91463994105318,6.9144797,6.9142147,6.9139417,6.9135337,6.913416,6.9126423,6.9119023,6.911468,6.9109643,6.9104266,6.9098868,6.9097437,6.909589,6.9093345,6.9085751,6.9078217,6.9075265,6.9067575,6.9064792,6.9061917,6.9058295,6.9055157,6.9050212,6.9048297,6.9045151,6.9039149,6.90338551061045,6.90262724993855,6.902075],"lat":[52.428739,52.4286658853327,52.4283844,52.4282335,52.4281539,52.4280613,52.4279272,52.4277717,52.4276337,52.4275795570862,52.4275299,52.4274383,52.4272444,52.4272171,52.427043,52.4269448,52.4268261,52.4266981,52.4266111,52.4264749517475,52.4264453,52.426373,52.4263025,52.4262365994416,52.4261730925524,52.4261314,52.4258975,52.4258368,52.4258173,52.4257601,52.4257031,52.4256482,52.4255902,52.4255305,52.4252711,52.4252345,52.4249884,52.4248797,52.4246448,52.4245853472961,52.4245213,52.4244736,52.4243976,52.4243589,52.424196,52.424177,52.424160985409,52.4241314,52.4240617,52.4226693,52.4226601,52.4226662,52.4226378,52.4226499,52.4226544,52.4226664,52.4227593,52.4228051,52.4228053,52.4228093,52.4228064,52.4227885,52.4227354,52.4224792,52.4217815,52.4216497,52.42137,52.4211249,52.4208714,52.4207413,52.42068,52.4206681,52.42066,52.4206514,52.420646,52.4206326,52.4205965,52.4205674,52.4205,52.42024,52.4201348,52.42003,52.4199652,52.4198976,52.4198538,52.41984,52.4197663,52.4197,52.41968,52.4196739,52.41969,52.4197902,52.4198038,52.4197918,52.41971,52.4194953,52.4193059,52.4190718,52.4188921,52.41838,52.41815,52.418,52.4179646,52.4179524,52.417953,52.41796,52.41792,52.41786,52.41782,52.41757,52.41733,52.41727,52.41724,52.41714,52.41697,52.41688,52.41676,52.41666,52.41662,52.41655,52.416537,52.4162412,52.4162079,52.4161737,52.4161211,52.41605,52.4159067,52.4158769,52.4157667,52.4157532,52.41559,52.4153869,52.4153696,52.41535,52.4151627,52.4151367,52.4150812,52.4150607,52.41471,52.41448,52.41441,52.4143336,52.4141351,52.413497922627,52.4130295,52.412764,52.4121193,52.4118611,52.4117078,52.4115405,52.4109621136749,52.4090112685297,52.4085175224343,52.4085777356687,52.4085897783057,52.4080104,52.4079775,52.4079423,52.4079043,52.4078617,52.4078391,52.4078039,52.4077664,52.4077288,52.4076889,52.4076139,52.4073939,52.4073021,52.4072313,52.4071562,52.4070853,52.4070435,52.4070205,52.4069975,52.4069629,52.406914,52.406896939803,52.406713,52.4066944,52.4066633,52.4066235,52.4062835,52.4061912,52.406106,52.4060627,52.4060259,52.4060117,52.4060253,52.4060403,52.4060596,52.4060848,52.4061318,52.4062105,52.406245,52.406293,52.406338,52.4063718,52.4063931,52.4063933,52.4063806,52.4063597,52.4063243,52.4062807,52.4061522,52.4060954,52.4060657,52.4060468,52.4060306,52.4060286,52.4060306,52.4060427,52.4060932,52.4063605,52.4063851,52.4064447,52.4064976,52.4065192,52.4066107,52.4066828,52.4067098,52.4069637,52.4073933,52.4080214,52.4081352,52.408234,52.4083652,52.4085702,52.4086352306858,52.4086623,52.4087325,52.4087803,52.4088568,52.408879,52.4090559,52.4092744,52.4093977,52.4095239,52.4096865,52.4098284,52.4098637,52.4098959,52.409941,52.4100651,52.4101773,52.4102209,52.4103349,52.4103764,52.4104036,52.4104212,52.410424,52.4104078,52.4103975,52.4103807,52.4103486,52.4103205116619,52.410280279842,52.410181]}]],[[{"lng":[7.050764,7.0521707,7.0523273,7.0546294,7.0558306,7.0561839,7.0568072,7.0573196,7.05818606731069,7.06322061579349,7.0639677,7.0642417,7.0643081,7.0644186,7.0647729,7.0651893,7.0654571,7.065827,7.0662552,7.0665771,7.0669931,7.0672536,7.067398,7.06768696064175,7.067999],"lat":[52.428739,52.4292325,52.429282,52.4300196,52.4304011,52.4305178,52.430727,52.4308899,52.4310692579517,52.431488209721,52.4316924,52.4319154,52.4319694,52.4320475,52.432311,52.4325234,52.4326478,52.4327684,52.4329426,52.4330487,52.433171,52.4332613,52.4333151,52.4334177534791,52.433517]}]],[[{"lng":[7.06945,7.06892908021493,7.06849962397953,7.067999],"lat":[52.43356,52.4336151341945,52.4335949933457,52.433517]}]],[[{"lng":[7.070189,7.06992013306588,7.06966906634364,7.06945],"lat":[52.434269,52.4340622586682,52.4338326547754,52.43356]}]],[[{"lng":[7.070189,7.07036699181878,7.0704095,7.0704513,7.0705095,7.0705281,7.0705733,7.0706205,7.0706419,7.0706611,7.0706578,7.07064667433256,7.0706375,7.07063542363337,7.070657],"lat":[52.434269,52.434514401069,52.434615,52.4347214,52.4348553,52.4349116,52.4350183,52.4351773,52.4352815,52.4354189,52.4355645,52.4356644666817,52.4357469,52.4357644601376,52.435794]}]],[[{"lng":[7.070657,7.07064144741999,7.07061270023787,7.0705903,7.0704346,7.070404,7.0701989,7.0701907,7.0701871,7.0702254,7.0702731,7.070315,7.0703469,7.0704316,7.0704682,7.070518],"lat":[52.435794,52.4358370884012,52.4358980655585,52.4360138,52.4363538,52.4364244,52.4369626,52.436994,52.4371169,52.4372612,52.4373636,52.4374504,52.4375069,52.4376467,52.4377028,52.437788]}]],[[{"lng":[7.232169,7.23142284103533,7.2306107682218,7.22981200807734,7.22946587868141,7.2289212,7.228796,7.2286352,7.2281255,7.2271095,7.2260831,7.2256652953322,7.2250229,7.2223033,7.2218427,7.2217042,7.2215178,7.2206968,7.2203878,7.2201508,7.2196056,7.2191195,7.2185415,7.2182173,7.2177568,7.2171878,7.2158657,7.2151241,7.2149475,7.21466223641004,7.21456864908855,7.2144608,7.2140699,7.2139289,7.2136678,7.2133554,7.2126574,7.2120724,7.21174476418399,7.21157404079147,7.2112209,7.2103343,7.2098997,7.2097651,7.2085593,7.20822239407433,7.2074328,7.2068835,7.2063506,7.205821,7.2053047,7.2049198,7.2042162,7.2034222,7.202864,7.2025618,7.2015166,7.1976809,7.1970467,7.1967679,7.1965441,7.195823,7.1944046,7.1926913,7.1911977,7.19075,7.1901895,7.1899428,7.1895942,7.1891902,7.1888107,7.1886571,7.1876622,7.1864406,7.184126,7.1821318,7.1803064,7.1783445,7.17472232444242,7.1742919,7.1719501,7.1707421,7.1693798,7.1692143,7.1636123,7.1618131,7.1610142,7.1600887,7.1595388,7.158621,7.1582685,7.1576916,7.1570332,7.1562846,7.14787394244252,7.1467471,7.146133,7.1421442,7.1313077,7.1297677,7.128517,7.1273123,7.1260872,7.1248891,7.1247649,7.1234023,7.11992359170429,7.11891855104408,7.11764022068689,7.1173837,7.1172247,7.1164685,7.1157002,7.1151642,7.1146031,7.1141612,7.1135829,7.1129968,7.1124323,7.1118018,7.11037471898432,7.1094836,7.10875,7.1082973,7.106382,7.1060629,7.1049018,7.1047732,7.1037035,7.10356659950738,7.1032724,7.1028757,7.1023987,7.1018919,7.10062,7.0999945,7.0995719,7.0992329,7.0988581,7.0985781,7.0982159,7.0971268,7.0969275,7.0966593,7.0959296,7.0956864,7.0952053,7.0946941,7.0939333,7.0933032,7.0931124,7.0929936,7.0929415,7.0922579,7.0917991,7.0913636,7.0911266,7.0905957,7.0899389,7.0892562,7.0891795,7.0886017,7.0883934,7.0879648,7.0876256,7.087439,7.0870537,7.0869943,7.0868022,7.0861725,7.0857204,7.0848829,7.0844259,7.0839438,7.0838247,7.0837032,7.0835063,7.0832202,7.0831407,7.0827835,7.0824761,7.0817279,7.0812476,7.0810221,7.0804773,7.080158,7.0798469,7.0795772,7.0793245,7.0791481,7.0789195,7.0787504,7.0786935,7.0786032,7.0785228,7.0785194,7.0785135,7.0784912,7.0784376,7.078369,7.0782989,7.0781482,7.0779299,7.0776822,7.0775413,7.0774145,7.0772579,7.0771731,7.0769455,7.0767322,7.0759018,7.0753588,7.0742618,7.0739567,7.0736953,7.0729677,7.0728742,7.0726209,7.0725271,7.0720104,7.0715586,7.0712831,7.0710322,7.071013,7.0708214,7.070518],"lat":[52.489641,52.4893340988915,52.48891257419,52.4880532997873,52.4875182713959,52.4868972,52.4868221,52.4867263,52.4864227,52.4858189,52.4852541,52.485040233045,52.4847114,52.4833989,52.4831779,52.4831114,52.4830199,52.4826607,52.4825423,52.4824514,52.4822581,52.4821319,52.4820037,52.4819378,52.4818628,52.4817937,52.4816787,52.4816385,52.4816311,52.481615860936,52.4816108614073,52.4816051,52.4815854,52.4815775,52.4815606,52.4815396,52.4814963,52.4814497,52.4814158783082,52.4813982546021,52.4813618,52.4812461,52.4811894,52.481169,52.4809445,52.4808679374247,52.4806885,52.4805541,52.4804064,52.4802533,52.4800907,52.4799616,52.4797028,52.4793877,52.4791568,52.4790187,52.4785253,52.4763976,52.4760458,52.4758941,52.4757708,52.4753625,52.4745594,52.4736324,52.4728233,52.4726092,52.4723719,52.472274,52.4721539,52.4720241,52.4719163,52.4718697,52.4716035,52.4712972,52.4707215,52.4702255,52.4697424,52.4692685,52.4683556728425,52.4682472,52.467661,52.467355,52.4670099,52.4669688,52.4655672,52.4651171,52.4649342,52.4647146,52.4645896,52.4643953,52.4643439,52.4642597,52.464206,52.4641449,52.4635933906084,52.4635195,52.4634795,52.4632141,52.4624963,52.4623802,52.4622591,52.4621228,52.4619612,52.4617707,52.4617484,52.4614785,52.4605970560411,52.4602595234604,52.459658533883,52.459544,52.4594702,52.4591072,52.4587147,52.4584165,52.4580816,52.4578015,52.4574206,52.4570143,52.4565822,52.456066,52.4549232117453,52.4542096,52.453609,52.4532383,52.4516858,52.451431,52.4504882,52.4503859,52.4495235,52.449408956113,52.4491628,52.448794,52.4482922,52.4477549,52.4464066,52.4457516,52.4453179,52.4450357,52.4447833,52.4446118,52.4444111,52.4438591,52.4437669,52.4436364,52.4432762,52.4431529,52.4429188,52.4426562,52.442283,52.4420226,52.4419466,52.4419059,52.441888,52.4416538,52.441533,52.4414183,52.4413642,52.4412624,52.4411349,52.441007,52.4409926,52.4408912,52.4408581,52.4407859,52.4407287,52.4406993,52.4406671,52.4406659,52.4406641,52.4406727,52.4406815,52.4407063,52.4407185,52.4407334,52.4407382,52.4407415,52.4407463,52.4407521,52.4407533,52.4407572,52.4407651,52.4407926,52.4408114,52.4408186,52.4408405,52.4408441,52.4408351,52.4408055,52.440768,52.4407392,52.4406431,52.4405431,52.4405045,52.4404139,52.4403165,52.4402867,52.4402614,52.4402148,52.4401566,52.440102,52.4400559,52.4399643,52.4398299,52.439684,52.4395993,52.4395278,52.4394456,52.4394082,52.439328,52.4392718,52.4391158,52.4390105,52.4388025,52.4387545,52.4387055,52.4384614,52.4384362,52.4383652,52.4383393,52.4382341,52.4381044,52.4379984,52.4379018,52.4378945,52.4378212,52.437788]}]],[[{"lng":[7.289806,7.28968621995047,7.2894224,7.2880408,7.2876352,7.2856699,7.2853582,7.2843807,7.2821714,7.2815861,7.2811434,7.280754,7.2799674,7.2779832,7.2769714,7.27513510720601,7.2738187451706,7.271707,7.2704982,7.26924730995249,7.26825018299827,7.2663698,7.2649522,7.263606,7.2578347,7.2572382,7.2539554,7.2531095,7.2527948,7.251696,7.2467683,7.2464064,7.2440205,7.2427783,7.2414802,7.2394279,7.2387249,7.2382307,7.237812922191,7.23733366610433,7.23569620780818,7.23447144225335,7.23333986538203,7.232169],"lat":[52.507742,52.5076962794508,52.5076164,52.5071483,52.5070109,52.5063398,52.5062343,52.5059003,52.5051379,52.5049238,52.5047608,52.5046272,52.5043663,52.5036933,52.5033671,52.5026959945841,52.5022626415654,52.501541,52.5011279,52.5006389749289,52.5003065669349,52.4997392,52.499272,52.498815,52.4968246,52.496619,52.4954921,52.4951892,52.4950842,52.4947356,52.4930143,52.4928992,52.4920991,52.49165,52.4912037,52.4904538,52.4901984,52.4900175,52.4899339540015,52.4899420600956,52.4898609990873,52.4898204685271,52.4897961501731,52.489641]}]],[[{"lng":[7.289806,7.2908752,7.291577,7.29253,7.2930869,7.2938597,7.2941389,7.294477,7.2949022,7.2952104,7.2953826,7.29546176565548,7.295653],"lat":[52.507742,52.5081002,52.5083513,52.5086716,52.5088604,52.5090988,52.5091769,52.5092623,52.5093598,52.5094212,52.5094515,52.5094653905032,52.509446]}]],[[{"lng":[7.295653,7.29569491535742,7.29582944311861,7.2960468,7.296246,7.2965761,7.297004,7.2973984,7.2977869,7.2983466,7.2984968,7.2989819,7.2994336,7.299841],"lat":[52.509446,52.5094784794309,52.5095222972733,52.5095527,52.5095785,52.5096149,52.5096524,52.5096831,52.5097107,52.5097525,52.5097646,52.5097974,52.5098505,52.509959]}]],[[{"lng":[7.299841,7.3004576,7.3011991,7.3018969,7.3021154,7.302177,7.30253474416028,7.302806],"lat":[52.509959,52.5100655,52.5102422,52.5104269,52.5104847,52.510501,52.5105953616365,52.510661]}]],[[{"lng":[7.302806,7.30282615366397,7.30286812311531,7.3033246,7.3036003,7.3037209,7.3052496,7.3061399,7.3066323,7.3068154,7.3070841,7.30811667420095,7.3083507,7.3093509,7.3099258,7.3105176,7.311073,7.3113694,7.31141908349841,7.31188347606705,7.3120896,7.3132662,7.3138819,7.3140922,7.3145405,7.3147858,7.3148393,7.3151169,7.3152949,7.3155007,7.31589860673945,7.31595400268135,7.31609859700479,7.3161847,7.3162006,7.3162562,7.3163555,7.3164275,7.3164923,7.31657522860125,7.31655544433628,7.31648946274704,7.3164597,7.31640315626593,7.3166883,7.3167153,7.3167363,7.316753,7.3167598455266,7.316766,7.3167059,7.3167096,7.3160683,7.3163726,7.3162455,7.3162172,7.31619932756692,7.31624285294984,7.31630220574474,7.31653170321833,7.316653],"lat":[52.510661,52.5106678428101,52.5106832963172,52.5108037,52.5108822,52.5109156,52.51133,52.5115749,52.5117171,52.5117792,52.5118874,52.5123701816225,52.5124796,52.512968,52.5132198,52.5134791,52.5137273,52.5138597,52.5138832655991,52.5141035330739,52.5142013,52.5147756,52.5150605,52.5151433,52.5153327,52.5154508,52.5154753,52.5156051,52.5156922,52.5158018,52.5160811316503,52.5161533685404,52.5164343931368,52.5166023,52.5166559,52.5168411,52.5171871,52.5174439,52.5177009,52.5180411171644,52.5182530022532,52.5185761162792,52.5186983,52.5192732926845,52.5199888,52.5200806,52.5201854,52.5203913,52.5207021405543,52.5209816,52.5210884,52.5211309,52.5213934,52.5216281,52.5217706,52.5218537,52.5219391577104,52.5219945319529,52.5220523136968,52.5222425114003,52.522354]}]],[[{"lng":[7.323251,7.3226998,7.321939,7.32166502189835,7.3215488,7.3205165,7.3201418,7.3199935,7.319461,7.3189078,7.3184689,7.3181823,7.3179332,7.3176414,7.31725316639317,7.316653],"lat":[52.523037,52.5229977,52.5229157,52.522885299517,52.5228936,52.5230102,52.5229514,52.5229208,52.5228184,52.5227121,52.5226174,52.5224514,52.5223077,52.5224158,52.5224719708479,52.522354]}]],[[{"lng":[7.411333,7.41082154967486,7.41021436987839,7.4095519919186,7.40819963691737,7.40668168742619,7.40452895905689,7.39669081986608,7.39026023383983,7.3857063853663,7.38468521934496,7.38359505561948,7.3826980854656,7.380624,7.38044926608234,7.3801496,7.3795314,7.3790474,7.3787347,7.3784322,7.3780789,7.3777463,7.377472,7.3772885,7.3771522,7.3768836,7.3765779,7.3763556,7.3761561,7.3759285,7.3757199,7.3754579,7.3752606,7.3748604,7.3745987,7.374394,7.3740562,7.373915,7.3737081,7.3729328,7.3719111,7.3713762,7.3710054,7.37057892981905,7.370415,7.3700399,7.3694188,7.3683128,7.3679958,7.3672863,7.3664811,7.3654691,7.3645754,7.36346,7.3628248,7.3626858,7.361687,7.3605525,7.3588429,7.358822,7.3563332,7.3562615,7.3561978,7.3545812,7.3539499,7.3533738,7.3532359,7.3523897,7.3523253,7.3520755,7.3515036,7.351303,7.3512019,7.3511414,7.3504719,7.349703,7.3482247,7.3480948,7.3480584,7.3478294,7.347401,7.3473013,7.3471268,7.3469262,7.34680426119047,7.3466233,7.3465047,7.3463443,7.3461505,7.3460088,7.3457541,7.3454615,7.345204,7.3451363,7.3451004,7.3450482,7.344832,7.344746,7.3445591,7.3444705,7.3443187,7.3441322,7.3438729,7.3435274,7.3433819,7.3432688,7.343162,7.342894,7.34268908912496,7.342374,7.3418924,7.3417899,7.3401495,7.3399821,7.3396751,7.3394989,7.3390068,7.337929,7.3373311,7.3368085,7.3365242,7.3362866,7.335576,7.3349891,7.3336076,7.3334673,7.3331571,7.3328469,7.3322573,7.33119645539893,7.3309537,7.3308009,7.3306029,7.3297805,7.3293877,7.3290816,7.3288075,7.3285439,7.32765338077463,7.32720892846342,7.32401738261165,7.32347363776284,7.32335543236092,7.32324195517508,7.323251],"lat":[52.600366,52.5997732117828,52.5990859192018,52.5985662517989,52.597778357204,52.5968563348975,52.5953139996622,52.5899992330445,52.5855725631716,52.5824451196808,52.5817659402628,52.5811538312777,52.5807178032262,52.579934,52.5798732623074,52.5797662,52.579539,52.5793408,52.5792006,52.5790635,52.5788695,52.578683,52.5785173,52.5784042,52.5783028,52.5781046,52.5778541,52.5776783,52.5774756,52.577243,52.5769969,52.5766416,52.5763435,52.5757181,52.5752477,52.5749285,52.5744659,52.574272,52.5739932,52.5729878,52.5716095,52.5709014,52.5704616,52.5698786722277,52.5696546,52.5691846,52.5683951,52.5669213,52.5664988,52.5655872,52.5645044,52.5631955,52.5620092,52.5605462,52.5597582,52.559578,52.5582837,52.5568374,52.554598,52.5545714,52.5513938,52.5513028,52.5512236,52.5492139,52.5483922,52.5476531,52.5474802,52.5464186,52.5463329,52.5460005,52.5452774,52.5450146,52.5448821,52.5448043,52.5439437,52.5429923,52.5411252,52.5409703,52.5409094,52.5406138,52.5400609,52.5399379,52.5396924,52.5394448,52.5392936751661,52.5390694,52.5389081,52.5386978,52.5384622,52.5382679,52.5378987,52.537442,52.536902,52.5367301,52.5365835,52.536371,52.5354752,52.5351586,52.5345791,52.5343927,52.5340653,52.5337256,52.5333408,52.5328649,52.5326717,52.5325452,52.5324357,52.5321765,52.5319807713791,52.5316798,52.5312197,52.5311217,52.5295651,52.5294057,52.5292007,52.5290854,52.528856,52.5284329,52.5281962,52.5280059,52.5278989,52.5278412,52.5276499,52.527502,52.5271425,52.5271054,52.5270267,52.5269373,52.5267365,52.5263248087541,52.5262306,52.5261714,52.5260915,52.5257529,52.525583,52.5254532,52.5253626,52.5252936,52.5246713128758,52.5244296660443,52.5235666307939,52.5232962095933,52.5232156582756,52.5231207226043,52.523037]}]],[[{"lng":[7.48676,7.48676951473237,7.4865621,7.4864199,7.4863282,7.4861148,7.485599,7.4852465,7.4849251,7.4845141,7.4843737,7.4837713,7.4832059,7.4825284,7.4819276,7.4814618,7.4808851,7.4800574,7.4794971,7.4792331,7.4785146,7.4784077,7.4783605,7.4783226,7.478255,7.4781393,7.4777979,7.4773197,7.4771884,7.4763282,7.4757822,7.47464271805596,7.4741289,7.4738551,7.4727053,7.4724435,7.471744,7.4713698,7.4711873,7.4706469961695,7.4702047859201,7.46771232815075,7.46598368808492,7.46409424429203,7.46184299211326,7.45492843184992,7.45139074985472,7.44761186226894,7.44407418027374,7.43908926473505,7.43402394733283,7.42871742434003,7.4252776,7.4215515,7.4196863,7.419187,7.418774,7.4186466,7.4183526,7.4181353,7.4180516,7.4179829,7.4179504,7.4179162,7.41782614745652,7.4177972,7.4177175,7.4176172,7.4175604,7.417409,7.4172404,7.4171653,7.4168576,7.4165487,7.4158438,7.4155921,7.4153379,7.4151456,7.4150285,7.4147034,7.4142671,7.4141561,7.4138535,7.4136923,7.413384,7.4130957,7.4127416,7.4126615,7.412535,7.4124981,7.4124087,7.4120707,7.41155551661687,7.4113968,7.411333],"lat":[52.671096,52.6706112298451,52.6702662,52.6701143,52.670019,52.669797,52.6692665,52.6688992,52.6685926,52.6681846,52.6680306,52.6673904,52.6667552,52.6659708,52.6653213,52.6647566,52.6638223,52.6622525,52.6612259,52.6607411,52.6593587,52.659153,52.6590621,52.6589877,52.6588635,52.6586507,52.6580205,52.6571326,52.6568889,52.6552528,52.6542407,52.6520522549598,52.6510654,52.6505469,52.6483548,52.6478627,52.6465468,52.6459958,52.6457429,52.6445505603782,52.6422576301675,52.634304621679,52.6312303720798,52.6306447762373,52.6298639695896,52.6298639695896,52.6288391397208,52.6258133115014,52.6234217831227,52.6205420143219,52.6188824004866,52.6186383343216,52.6175006,52.6149486,52.6136684,52.6132722,52.6128809,52.6127319,52.6123568,52.6119097,52.6116942,52.6114782,52.6112696,52.6109114,52.6101392962207,52.6098911,52.6092977,52.6088552,52.6086738,52.6082441,52.6079743,52.6078327,52.607434,52.6071039,52.6064953,52.6062631,52.6060061,52.605766,52.6056197,52.6050696,52.6041658,52.603935,52.6034043,52.6031905,52.6027818,52.6024453,52.6020253,52.6019303,52.6017804,52.6017365,52.6016301,52.6012305,52.6005990405408,52.6004045,52.600366]}]],[[{"lng":[7.486634,7.48676],"lat":[52.671517,52.671096]}]],[[{"lng":[7.486447,7.48646317140647,7.4865251,7.4866101,7.4866797,7.4867383,7.4868588,7.4868747,7.4867356,7.4866588,7.486634],"lat":[52.673601,52.6735805352413,52.6735335,52.6734585,52.6732955,52.6731585,52.6728764,52.6726795,52.6723627,52.6719621,52.671517]}]],[[{"lng":[7.486447,7.4864605647527,7.48650184220482,7.4865586,7.4869059,7.4875184,7.4880052,7.4880799,7.4881202,7.4882115,7.4882812,7.4884378,7.4886482,7.4889046,7.4895188,7.4903006,7.4908481,7.4913053,7.4913586,7.491494,7.4916057,7.491703,7.4918003,7.4919153,7.4925785,7.4928576,7.493135,7.4933645,7.4940954,7.49479,7.4949236,7.4951056,7.4952298,7.4955989,7.496148,7.4964781,7.4970299,7.4972574,7.4973318,7.4974478,7.4975782,7.4976441,7.4977268,7.4978943,7.4980761,7.4983026,7.4994247,7.4996482,7.4999446,7.50030140930934,7.5004287,7.5007212,7.5020185,7.5022669,7.5029407,7.5030843,7.5032724,7.5039795,7.5047536,7.5050331,7.5051381,7.5056829,7.506063,7.5065942,7.5070592,7.508304,7.5085597,7.5086822,7.5090185,7.5093506,7.5100397,7.5115844,7.5116592,7.5131644,7.5137967,7.513925,7.51413988473458,7.5143442,7.5146552,7.5149294,7.5150039,7.5156553,7.5158285,7.5168148,7.51722063162459,7.5178927,7.5186847,7.5190155,7.5200656,7.522596,7.5261813,7.5280722,7.5291137,7.5298787,7.5318557,7.54102107122205,7.5425059,7.5429366,7.5458948,7.5489158,7.5516397,7.5524562,7.5525844,7.5535432,7.5543826,7.5556125,7.5561307,7.5571162,7.5628639,7.5667285,7.5706416,7.572629,7.57312304513284,7.57486815834443,7.57586108372053,7.5763339053282,7.58027723753616,7.5828872128105,7.58356807592554,7.58423002617628,7.60265115601107,7.60934630997566,7.61250475831489,7.6153227750966,7.61569195797266,7.61698843802203,7.61801356178201,7.62101858143134,7.62213415728778,7.62306882895128,7.62646580768531,7.62779243843351,7.63508890754861,7.63630498573446,7.63736026019325,7.6383049820897,7.6463652689083,7.65043561324937,7.65248586076931,7.65338033150106,7.65659640604215,7.65809389075034,7.65866675402797,7.6593602201009,7.66059639875263,7.66108886016673,7.6615712713479,7.6625561941761,7.66580241941602,7.66637528269365,7.66720945202774,7.66843558044653,7.66943055350768,7.67146070056175,7.67452602160872,7.67621446074279,7.67671697238984,7.6780335529051,7.68200004483912,7.68273706192145,7.68384258754495,7.68459300493787,7.68755447357779,7.68791628196367,7.69049584175184,7.69195647560592,7.6930821016953,7.69353771225529,7.69402012343645,7.69450253461761,7.6959564683164,7.6969547914552,7.70023116739394,7.70345394209033,7.70424456041502,7.70512898091382,7.70601340141262,7.70646901197261,7.70849245887138,7.70973868775605,7.71343717347831,7.71911220501228,7.72227467831102,7.72536344990153,7.73044886776964,7.73307532864486,7.73449940343181,7.73715975163811,7.73778484092541,7.74018649976612,7.74111865045771,7.74177663918119,7.74200693523441,7.74292811944728,7.7441782980219,7.745647806171,7.74654705742643,7.74678831995837,7.74707344840521,7.74768757121379,7.74857585599049,7.74962863794806,7.75064852046946,7.75222769340581,7.75250185537393,7.75268828551224,7.75285278269311,7.75323660944848,7.75348883845915,7.75382879929961,7.7543113243635,7.75488158125718,7.75521057561892,7.75550667054449,7.75590146377857,7.75643165568937,7.75657823038775,7.75737729890471,7.758713],"lat":[52.673601,52.6736068514084,52.6736254067377,52.6736435,52.6737108,52.6738393,52.6739544,52.6739821,52.6739971,52.6740576,52.6740937,52.674141,52.6742056,52.6742843,52.6744656,52.6746469,52.6747799,52.6749202,52.6749355,52.6748315,52.6747677,52.6748109,52.6748325,52.6748394,52.6748065,52.6748026,52.674801,52.674802,52.674828,52.6748583,52.6748717,52.6748975,52.6749155,52.6749883,52.6751022,52.6751476,52.6752183,52.675239,52.6752431,52.6752281,52.6751895,52.6751703,52.675141,52.6751166,52.6751253,52.6751832,52.6756219,52.6756543,52.6757703,52.675904886688,52.6759529,52.6760571,52.6764854,52.6765569,52.6767401,52.6767761,52.6768231,52.6770009,52.6771837,52.6772513,52.6772791,52.6774199,52.677526,52.6776743,52.6778045,52.6781569,52.6782293,52.6782653,52.678364,52.6784632,52.678669,52.6791305,52.6791522,52.6795882,52.6797792,52.6798204,52.6798894481763,52.6799551,52.6800809,52.6801962,52.6802275,52.6805346,52.6806163,52.681084,52.6812764309305,52.6815951,52.6819635,52.6821232,52.6826146,52.6837986,52.6854641,52.6863362,52.6868083,52.6871061,52.6877456,52.6905648816246,52.6910216,52.6911541,52.6920769,52.692995,52.6938199,52.6939935,52.6940208,52.6941906,52.694298,52.694379,52.6944016,52.6944267,52.6946272,52.6947373,52.6948694,52.6949309,52.694946638535,52.6950770646958,52.6952260739385,52.6954037381401,52.6976674606333,52.6992090171129,52.6994840846641,52.6996445399352,52.7029452032095,52.7022002836559,52.7022805063726,52.7029108225869,52.703034482576,52.7029979431538,52.7028700549349,52.7022366982324,52.7020357273578,52.7019565567591,52.7018164853482,52.7018408456258,52.7016276927352,52.7016155125385,52.7014145388035,52.7010491296416,52.7013049163762,52.7017616746737,52.7017068639304,52.7017312242692,52.7023463183194,52.7027238965117,52.7028761448586,52.7032232691037,52.7037713543957,52.7039418684161,52.7040332149386,52.704161099749,52.7046360971923,52.7047274422617,52.704934490378,52.7052937774248,52.705659151049,52.706523856439,52.7078330608618,52.7086002972917,52.7088803961189,52.7097024149039,52.7122028805454,52.7126737314184,52.7131486362032,52.7134246468651,52.7143703760377,52.7146057904131,52.7159127229794,52.7167812838343,52.7170166851994,52.7172196163911,52.7174712497577,52.7178162286269,52.7187212778681,52.7191555324962,52.7213145660881,52.7235262471965,52.7240537875906,52.7244027723441,52.7246381325778,52.7248166808732,52.7256850644315,52.7258717240598,52.7263748895128,52.7270403574849,52.728971780182,52.7306272172586,52.7313737663559,52.7319093263119,52.732091354238,52.7322682242653,52.7322881461953,52.7323744744536,52.7323943963351,52.7323943963351,52.7324076775843,52.7324541619249,52.7325205680397,52.7325869740534,52.7326135364305,52.732626817613,52.7326467393792,52.7327463480735,52.7329522053205,52.7331912641294,52.7334236811591,52.7337822650029,52.7338553094983,52.7339017922953,52.7339017922953,52.7339549154312,52.7339681962051,52.7339549154312,52.7339084326909,52.7338154670614,52.733749062919,52.733709220385,52.7335764116754,52.7334129717828,52.733381478219,52.7332712505665,52.733187]}]],[[{"lng":[7.864115,7.86403656402035,7.86393606169094,7.86380875874035,7.86258933047685,7.86208011867451,7.86178531184158,7.85796334484815,7.85732789952987,7.85658475161527,7.85579852266214,7.85474303721822,7.8533859845046,7.85174890504055,7.85040262258657,7.84816240858313,7.84701537593234,7.84193181338609,7.84070400378805,7.83945465367075,7.83531887397211,7.81955121387102,7.80761776619888,7.79633053410466,7.78997608092185,7.78179068360161,7.78028284725315,7.77951815881928,7.77887194324137,7.77869961908726,7.77656710768014,7.77629785118935,7.77212976071181,7.77088041059451,7.77037420839181,7.77004033034322,7.7697279928139,7.76920025009194,7.76896330438003,7.76817169029709,7.76800475127279,7.76784858250813,7.76760625166641,7.76732622491599,7.76656692161194,7.76482752468138,7.76451518715206,7.76429978195942,7.76403052546863,7.76369664742004,7.76345970170814,7.76307197236139,7.76272732405317,7.76231805418716,7.76211341925415,7.76198417613857,7.7619949463982,7.76202725717709,7.76217804081194,7.76244729730274,7.7627057835339,7.76279194561096,7.76279194561096,7.7629353,7.7629634,7.7629816,7.762973624868,7.762965,7.7629149,7.762893,7.76291112016881,7.76306522374057,7.76309202436175,7.76309202436175,7.76305852358528,7.76299822218763,7.76283741846058,7.76246220976412,7.76210710153354,7.76057276597122,7.7601783,7.7600786,7.7598105,7.7596146,7.7593801,7.7592253,7.7591936,7.758713],"lat":[52.795285,52.795196966486,52.7951199906666,52.7950835283888,52.7948890623912,52.7947999318517,52.7947026983276,52.7924985069993,52.7921598394272,52.7917430141804,52.7913717758326,52.7908767864407,52.7902775812205,52.7895936956126,52.7891182259501,52.7883561609953,52.7880011774637,52.7867375428558,52.7864248853306,52.7860470878234,52.7843013601931,52.7762492622036,52.7701505656357,52.7644159690698,52.7611182340415,52.7569077192795,52.7558778410228,52.7548935675281,52.7536224529571,52.7533617069221,52.7519015002847,52.7517580844933,52.7503369388605,52.7497502138409,52.7495155216207,52.7493199438049,52.7490330947535,52.748257288995,52.7476966140496,52.7455320802943,52.7451539278223,52.7447920719532,52.7444921530998,52.7442769926507,52.7438271082789,52.7428458226303,52.7426306540527,52.7424546062445,52.7421090288491,52.7416656425171,52.7413330998079,52.7409679519495,52.740557156952,52.7397355553411,52.7393443110352,52.7388878549061,52.7386139789341,52.7383922685535,52.7379292814752,52.7371402358263,52.7364555152169,52.7361816239615,52.7359403373821,52.7356707,52.7355543,52.7354421,52.7353310728635,52.735211,52.7350258,52.7349394,52.7348142187209,52.7344125761344,52.7341448123542,52.7338973322464,52.7336944786513,52.7334185962471,52.7330372265194,52.7329236263811,52.732805968783,52.7330169408021,52.7331052,52.7331209,52.7331694,52.7331966,52.7332343,52.733235,52.7332352,52.733187]}]],[[{"lng":[7.864115,7.8641839674368,7.86426101922268,7.86435147131916,7.86453907566739,7.86493438482973,7.86614041278264,7.86880707458963,7.86923588452844,7.86966469446725,7.87066971776134,7.87188914602484,7.87298797149305,7.87428780162008,7.87544022833064,7.87632464882944,7.87679365970001,7.87734307243412,7.87837489634938,7.87892430908349,7.87970152709759,7.88317220753985,7.88878693767618,7.89046197649966,7.89245862277726,7.89416046222192,7.89445526905486,7.89480367713014,7.89901137465474,7.90107502248527,7.90198624360525,7.90234805199113,7.90446530106401,7.90482710944988,7.90831119020274,7.91101805294149,7.91707499332722,7.92138989333652,7.92325253650824,7.92542338682348,7.92609340235287,7.92687062036697,7.92802304707753,7.92870646291751,7.92957748310573,7.93087731323275,7.93184213559508,7.93236474770801,7.93273995640447,7.93324916820681,7.93519221324205,7.93621063684673,7.9455669,7.9460467,7.9470275,7.94880511817918,7.9518366,7.952038,7.9579011,7.9587207,7.965203,7.96641713330193,7.9679364,7.9706126,7.9728028,7.9742875,7.9777604,7.981088,7.98211793998371,7.9825442,7.9837107,7.9847082,7.9849814,7.9855968,7.9866281,7.9871756,7.9905663,7.9946571,7.997846],"lat":[52.795285,52.795316481304,52.7953407893624,52.7953549690569,52.795367123077,52.7953549690569,52.7952415313718,52.7949984496218,52.7949903468734,52.7950713742895,52.7952901475588,52.7955818435395,52.7958573323919,52.7961571271016,52.7963434849337,52.7964731246939,52.7965460468891,52.796537944429,52.7964893296367,52.7964650222202,52.796481227166,52.79668378848,52.7970808059189,52.7971942388072,52.7972347504813,52.7972995690815,52.7973400806575,52.7974211036964,52.7987255538454,52.7998274191411,52.8003702395742,52.8005322742113,52.8010426793721,52.8011074922972,52.8044938332578,52.8055307461716,52.8078880108655,52.8096133504073,52.8103099469691,52.8111037294448,52.8113548208838,52.8117274055078,52.8124401671949,52.8128532396418,52.813282506849,52.8139142508779,52.8144002015756,52.8148132553971,52.8151615134124,52.815250602224,52.8155664610844,52.8157527357467,52.8171585,52.8172315,52.8173844,52.8176542729293,52.8181145,52.8181452,52.8190402,52.8191659,52.8201328,52.8203230002281,52.820561,52.82096,52.8212809,52.8215071,52.8220429,52.8225436,52.8226966553323,52.82276,52.8229716,52.8231704,52.823224,52.823347,52.8235697,52.8236925,52.8243881,52.8252263,52.825826]}]],[[{"lng":[7.997846,8.0034239,8.00415869977389,8.0058563,8.0076218,8.0099967,8.0117456,8.01187312408918,8.0119918,8.01229480169521,8.01542454066639,8.01697545805338,8.02207647701693,8.02692738944977,8.03017026461205,8.03143659396262,8.03498767626842,8.03522218170371,8.03566439195311,8.03587879692252,8.036633,8.03678196551643,8.0369145,8.0370237,8.0370392,8.037096,8.037128,8.0371624,8.03712076235489,8.03707],"lat":[52.825826,52.8276505,52.8278990599612,52.8284733,52.8290604,52.8298716,52.8304638,52.8305075684427,52.8305483,52.8306507512118,52.8318506459709,52.8324232511105,52.8340685441179,52.8355742750276,52.8369666249004,52.8375656454152,52.8391683894085,52.8392857597503,52.839573113386,52.8397228603018,52.840692,52.8408650474197,52.8411642,52.8417565,52.8418946,52.8424853,52.8426658,52.8428384,52.8429920167064,52.84306]}]],[[{"lng":[8.03707,8.03708183241946,8.03707866826464,8.0370532,8.0368428,8.0369631,8.0373848,8.03747816776591,8.0376927,8.0378583,8.0386196,8.0386949,8.0388879,8.0392005,8.0394707,8.0396269,8.0397493,8.0400309,8.0402943,8.0404614,8.0407696,8.0415098,8.0417059,8.0420407,8.0422752,8.0425376,8.04263590851446,8.04292809105439,8.04294],"lat":[52.84306,52.843239176965,52.8434826527549,52.8435922,52.8441274,52.8441543,52.8442485,52.8442703636564,52.8443206,52.8443587,52.8445168,52.8445316,52.8445731,52.8446687,52.8447723,52.8448334,52.8448905,52.8450227,52.8451291,52.8451903,52.8452872,52.8455306,52.8455794,52.8456652,52.845709,52.8457524,52.8457685654586,52.8458257092961,52.84583]}]],[[{"lng":[8.04294,8.04294128206754,8.04294443645321,8.0429398,8.0427775,8.0427296,8.0425984,8.0425941,8.0425828,8.0425798,8.04248042873659,8.0424639,8.0424636,8.0425103,8.04254711043845,8.04253289226605,8.04252],"lat":[52.84583,52.8458526683171,52.8458653321003,52.8459205,52.846455,52.8466237,52.8469487,52.8469593,52.8469946,52.847005,52.8473470980163,52.847404,52.8476804,52.8479494,52.8481041556193,52.8482065692125,52.84825]}]],[[{"lng":[8.04252,8.0427347667702,8.04306031405939,8.0431186,8.0432455,8.0434731,8.0441435,8.044939,8.0450972,8.0454009,8.0454558,8.0458258,8.0467212,8.0469788,8.047128,8.0473678,8.0475326,8.04852],"lat":[52.84825,52.8482163786906,52.8481541792707,52.8481512,52.8481692,52.8482258,52.848424,52.8486347,52.8486677,52.8487157,52.8487232,52.84877,52.8488797,52.84892,52.8489528,52.8490348,52.849097,52.849548]}]],[[{"lng":[8.04852,8.0504061,8.0504934,8.0507486,8.0508164,8.0510926,8.0511463,8.0513576,8.0514487,8.0515587,8.0520901,8.0528583,8.0539321,8.0540608,8.0544321,8.0550689,8.05537797425283,8.0556582,8.0558695,8.0563175,8.0572006,8.0572936,8.0579377,8.0585278,8.0586652,8.0586938,8.0591161,8.059417,8.0598873,8.06145920712876,8.0617839,8.0617839,8.0619298,8.062266,8.0623972,8.0625088,8.0632254,8.06342516709273,8.06363276613785,8.06368794806245,8.063696],"lat":[52.849548,52.8503255,52.8503663,52.8504857,52.850521,52.8506963,52.8507369,52.8509106,52.8509985,52.851101,52.8516277,52.8524117,52.8534845,52.8536204,52.8539992,52.8546489,52.8549589193151,52.85524,52.8554566,52.8559159,52.8568418,52.8569411,52.8575768,52.8581935,52.8583397,52.8583663,52.8587854,52.8591028,52.8595563,52.8618949402373,52.86231,52.86231,52.8625024,52.8629058,52.8629986,52.8630773,52.8635906,52.8637217537175,52.8638580489512,52.8639001247277,52.863906]}]],[[{"lng":[8.063696,8.06372043361484,8.06382202483189,8.0641388,8.0646874,8.0653387,8.0657721,8.0662802,8.0665042,8.0684513,8.06890956998752,8.07008441133817,8.0707508,8.0729228,8.0751728,8.07655,8.0803483,8.0816623,8.0834521,8.0838778,8.0868941,8.0874357,8.0879337,8.0887714,8.0890934,8.091027,8.0919123,8.09231300891807,8.0933213,8.0955615,8.0981482,8.1013469,8.1035544,8.1056585,8.1106639,8.1128378,8.1148275,8.1158807,8.1175009,8.1182933,8.11862199929777,8.12808633407822,8.13392886949456,8.13896738627563,8.14497072541903,8.15134927325889,8.15322531674121,8.157303,8.16075629129163,8.1636775589998,8.1664648236021,8.17010970808202,8.173414],"lat":[52.863906,52.8639216627534,52.8639843944609,52.8641979,52.8645319,52.8649951,52.8652342,52.8654552,52.8655402,52.8662974,52.8664684128565,52.8668304237366,52.8670783,52.8678964,52.8687147,52.869218,52.8706044,52.8710618,52.8715536,52.8716734,52.8724493,52.8725842,52.8727083,52.8729171,52.8729948,52.873478,52.8736992,52.8738016953325,52.8740596,52.8746311,52.8752863,52.8760954,52.8766647,52.8771971,52.8784593,52.8790262,52.8795757,52.8798916,52.8804599,52.8807466,52.8808683631462,52.8843183756555,52.8856768190579,52.8853857276262,52.8873263002543,52.8896872131549,52.8905927346714,52.8944234,52.8953464122978,52.8960901378202,52.8966721749786,52.8974805469531,52.898445]}]],[[{"lng":[8.173414,8.1743111,8.17548813293076,8.1761851,8.1768932,8.1787534,8.1794801,8.1801964,8.180818,8.1813907,8.1826052,8.1834301,8.1841356,8.1885725,8.1893319,8.1901644,8.1911727,8.1942096,8.19510746826501,8.1967594,8.1970398,8.1985999,8.1990521,8.2005779,8.2010801,8.2011075,8.2025315,8.2030508,8.2045115,8.20784538434848,8.210821],"lat":[52.898445,52.8987575,52.899167641226,52.8994105,52.8996494,52.9002854,52.900523,52.9007222,52.9008474,52.9009497,52.9010752,52.901104,52.9010916,52.9009581,52.9009231,52.9009037,52.9008687,52.9007765,52.9007541043633,52.9007129,52.9007076,52.9006772,52.9006617,52.9006092,52.9005935,52.9005926,52.9005465,52.9005303,52.9004733,52.9003763572199,52.900309]}]],[[{"lng":[8.210821,8.21355962198446,8.21523978248703,8.21631680845022,8.22092647957267,8.22578386666665,8.23045815934689,8.231068],"lat":[52.900309,52.9002072881341,52.9001813014241,52.9001163345808,52.899596596328,52.8990768518414,52.898583088805,52.898445]}]],[[{"lng":[8.231068,8.2321098,8.2327839,8.2331409,8.2334942,8.2336442,8.2338754,8.2356548,8.2358418,8.2362235,8.2366106,8.2369265,8.2371767,8.24804599332577,8.25106166602269,8.25196636783177,8.25683452518539,8.26277970850219,8.26566613808354,8.26980191778219,8.27311915774881,8.27927974625825,8.28350168803395,8.30887641972668,8.31602787212226,8.32541953852127,8.3286426,8.33343261168739,8.33552204205598,8.33937779500419,8.34273811600934,8.35340067304491,8.35421921277694,8.35473618523927,8.35564088704835,8.35635172418405,8.35792418209031,8.35960434259288,8.36034749050748,8.36130604361472,8.36227536698159,8.36294312307877,8.36578647162159,8.37140854714943,8.37655673125348,8.38291118443629,8.39010571787039,8.39562009080192,8.40147911204167,8.40466710889271,8.40613186420265,8.41108618363332,8.4159112599484,8.41866844641417,8.42026244483969,8.42123176820656,8.42327811753661,8.42452746765392,8.42549679102079,8.4262406,8.42667949945697,8.4284518,8.4290998,8.4295785,8.4299602,8.4300982,8.4304513,8.4305959,8.4306845,8.4309531,8.43201037443072,8.432223],"lat":[52.898445,52.8983912,52.8983162,52.8982775,52.8982427,52.8982253,52.8982016,52.8980109,52.8979917,52.8979617,52.8979426,52.8979331,52.8979379,52.8966404664621,52.89612068652,52.8956788686677,52.8935216932981,52.8917802806466,52.89087055965,52.8905066659053,52.8901427691058,52.889830855132,52.8910525053767,52.8953669945307,52.8959387620675,52.8974980897375,52.8988638,52.9006165767615,52.9010518477266,52.9012857228573,52.9017274835487,52.9031566784586,52.9038972427325,52.9044429135714,52.9047807063609,52.9048976340206,52.9048196822825,52.9045728341867,52.904293504382,52.9039946844577,52.9032281369666,52.9031241972797,52.9029293096945,52.9033450688165,52.9042025594059,52.9055277387544,52.9059954394352,52.9059434729423,52.9052938865206,52.9043064965059,52.9035009774587,52.9017859515381,52.9005126308097,52.9003826980193,52.8997590152017,52.8993562152778,52.8982127628356,52.8972511999282,52.8965884887484,52.8959679,52.8958793543053,52.8955218,52.8953911,52.8952945,52.8952559,52.8952432,52.8951681,52.8951335,52.8951114,52.8950583,52.8948440345867,52.894791]}]],[[{"lng":[8.43632,8.43619896457455,8.43604240514568,8.43604240514568,8.4359867,8.4358791,8.4355043,8.4352732,8.4351953,8.4350784,8.4349378,8.4346239,8.4340736,8.4338294,8.4336485,8.4335268,8.4332669,8.43272293309007,8.432223],"lat":[52.89731,52.8971673176709,52.8969718841368,52.8969718841368,52.8968827,52.8967189,52.8961529,52.8958896,52.8958008,52.8957162,52.8956144,52.8954525,52.8951687,52.8950631,52.8950172,52.8949882,52.8949166,52.8948215192765,52.894791]}]],[[{"lng":[8.43632,8.43638033319154,8.43643103076761,8.4364573,8.4365739,8.4367016,8.4368295,8.437692,8.4377876,8.4382342,8.4385731,8.43901271032573,8.4390571,8.4396963,8.4400098,8.4402963,8.4406128,8.4407345,8.4409477,8.4414843,8.4417325,8.44184964448538,8.4419926,8.4421796,8.4423013,8.4425461,8.4426725,8.4427825,8.4428717,8.4431972,8.4433199,8.4435855,8.4437697,8.44382749579056,8.4439004,8.444025],"lat":[52.89731,52.8974411718266,52.897536428865,52.8975686,52.8975916,52.8975826,52.8975699,52.8975341,52.8975301,52.8975089,52.8974928,52.8975025186582,52.8975035,52.897589,52.8976299,52.8976846,52.8977517,52.8977791,52.8978314,52.8979725,52.8980246,52.8980530191464,52.8980877,52.8981611,52.8982089,52.8983429,52.8984336,52.8985222,52.8986054,52.8989558,52.899088,52.899374,52.8995967,52.8996962397957,52.8998218,52.899991]}]],[[{"lng":[8.444025,8.4443866,8.4445619,8.4446955,8.4462376,8.4470989,8.4476924,8.4480415,8.4481893,8.4489541,8.4491123,8.4493208,8.4496902,8.4499458,8.4509115,8.452115,8.4527511,8.4529557,8.4541008,8.454991,8.4555425,8.4557833,8.4561219,8.4561634,8.4562871,8.4567223,8.4573816,8.4577394,8.4588565,8.4591594,8.4610545,8.4611015,8.4614101,8.46388036445575,8.465078,8.46511,8.4662449,8.4666983,8.4672847,8.4677857,8.468774,8.4690419,8.4691575,8.4696533,8.4700876,8.4733147,8.4754798,8.4768277,8.4784784,8.4807814,8.4816054,8.48269657199954,8.4835351,8.4839771,8.4858637,8.4879092,8.4903026,8.4958504,8.499698,8.4997847,8.5002567,8.5011163,8.5019929,8.50275146953224,8.5032825,8.5033995,8.5036111,8.5046987,8.5053879,8.5064826,8.508674,8.50966541898925,8.51119616646296,8.5129981,8.5149512,8.5168541,8.5184184,8.5230976,8.5235084,8.5260157,8.5295497,8.5323315,8.5337896,8.53549184593709,8.5421545,8.5437877,8.543843,8.5447757,8.5449619,8.5450283,8.5451716,8.5466114,8.5466765,8.5469852,8.5472785,8.5479235,8.5479662,8.5487265,8.5507151,8.5510641,8.5513579,8.5521966,8.5525451,8.5529681,8.5531465,8.5535303,8.5539424,8.554284,8.5546137,8.5551096,8.5570963,8.5584214,8.5590312,8.5593059,8.559653,8.5621505,8.5625715,8.5629792,8.5654031,8.5662375,8.5671903,8.5678078,8.5678842,8.56806456468719,8.56820902519121,8.5682845,8.5682845,8.569996,8.5701387,8.5717924,8.5723387,8.57278849841723,8.57324544601013,8.573448],"lat":[52.899991,52.9000819,52.9001669,52.9002316,52.9008595,52.9011999,52.9014394,52.9015803,52.9016311,52.9019248,52.9019728,52.9020282,52.9021218,52.902204,52.9023139,52.9024774,52.9025626,52.902585,52.9027564,52.9028777,52.9029203,52.9029343,52.9029357,52.9030061,52.9031383,52.9032548,52.9035546,52.9036745,52.9040544,52.9041608,52.9048338,52.9048477,52.9049553,52.9058165528719,52.9062341,52.9062452,52.9066386,52.9069092,52.9072723,52.9077798,52.90875,52.909013,52.9091293,52.9096359,52.9103088,52.9119819,52.9129641,52.9134472,52.9140386,52.9148939,52.9152373,52.9158503772228,52.9163215,52.9165689,52.9175074,52.9186457,52.9191411,52.9203772,52.9212462,52.9212658,52.9214029,52.9216542,52.9219275,52.9221937293031,52.9223801,52.9224001,52.9224613,52.9228121,52.9230479,52.9234302,52.9241773,52.924517100596,52.9250535580822,52.9256499,52.9263004,52.9269451,52.9274799,52.9290399,52.9291787,52.9300311,52.9311996,52.9321992,52.9327848,52.9335371397878,52.9364817,52.9371966,52.9372206,52.9376275,52.9377175,52.9377617,52.937824,52.9384952,52.9385218,52.9386481,52.938773,52.9390476,52.9390657,52.9394085,52.9402792,52.940466,52.9406364,52.9413111,52.9416047,52.9418181,52.941883,52.9420164,52.9421105,52.9421607,52.9421872,52.9421981,52.9421591,52.9421337,52.9421516,52.9421952,52.9422742,52.9431908,52.9432921,52.9432999,52.9431931,52.9431504,52.9431006,52.9430883,52.9430932,52.9426890215726,52.9425632121003,52.9425199,52.9425199,52.9422412,52.9422183,52.9419812,52.9420014,52.9420466142978,52.9421374256628,52.942256]}]],[[{"lng":[8.710165,8.7094759,8.7092491,8.7091466,8.7090343,8.7088213,8.7086117,8.7084428,8.7082327,8.7080251,8.7077852,8.7075842,8.7073513,8.7069369,8.706669,8.7062892,8.7061067,8.7054554,8.7051187,8.7048176,8.7045252,8.7042036,8.7038821,8.7030742,8.7030332,8.7022215,8.7013189,8.7012765,8.7012225,8.7011901,8.7010752,8.70041665356923,8.70033438380129,8.6995208272072,8.69868898844246,8.69836905045602,8.69807653572557,8.69757377603259,8.69740923649671,8.69725383804615,8.6967602194385,8.69537077446883,8.69291182251591,8.6917143403381,8.68947477443303,8.68742717131983,8.68080902554321,8.67563060070557,8.67248606735315,8.66744018825275,8.65954686107304,8.64455548113707,8.64230677414667,8.63910739428229,8.63663930124405,8.63446372293627,8.62822950274337,8.62710514924817,8.62688576320033,8.62664809498183,8.62549631823065,8.62305564844839,8.62084350579931,8.62057841432483,8.62027675850904,8.61941749648832,8.61846682361434,8.61758927942296,8.61706823755933,8.61662946546365,8.61289990265031,8.61237886078668,8.61105340341429,8.61069690108654,8.61034953984413,8.61003874294301,8.60963653518863,8.60767120184337,8.6065925537748,8.60511169795186,8.60260704057231,8.59904201729486,8.59211307461713,8.58787161102549,8.58494646372091,8.58450769162522,8.58236867765875,8.58070500012927,8.57637212568436,8.57483642334945,8.57386746830481,8.573448],"lat":[53.036234,53.0356428,53.0354649,53.0353994,53.035339,53.0352431,53.0351702,53.0351215,53.0350702,53.0350272,53.0349892,53.0349731,53.0349537,53.0349228,53.0349059,53.034884,53.0348708,53.0348246,53.0347992,53.0347739,53.0347405,53.0346826,53.0345968,53.0343425,53.0343296,53.0340633,53.033762,53.0337503,53.0337354,53.0337264,53.0337011,53.0336187236569,53.0330910124629,53.0321620051911,53.0313814037268,53.0311230325243,53.0308811516978,53.0298696353611,53.0295013055854,53.0287536414426,53.0275056731566,53.0253779963909,53.0215512128269,53.0197256734925,53.0162008443644,53.0131926891432,53.0082097988917,53.001567800806,52.9992352956897,52.9940747304824,52.9814705664833,52.9751106850196,52.9742189608724,52.9730519854274,52.9720281036607,52.9709381383512,52.9644913673583,52.963313117424,52.9631534455697,52.9630433267059,52.9627460043726,52.962217426287,52.960108555645,52.9599213402254,52.9597561494761,52.959458804537,52.9590623414374,52.9586548616859,52.9585116922129,52.9584841595676,52.9582638977741,52.9582749108905,52.9584290942244,52.9584566269048,52.9584290942244,52.9583740288111,52.9582363649711,52.9571625719769,52.956639432397,52.9558574645935,52.9546899933898,52.9534453903228,52.9508954943077,52.9492872752415,52.9472383615984,52.9469629625809,52.9457401697688,52.9449304636156,52.9431677855722,52.9425838826354,52.942341505517,52.942256]}]],[[{"lng":[8.720241,8.72036286688421,8.72048531346084,8.72051252381121,8.72043089276012,8.72013433347717,8.71963246392142,8.71887965958779,8.71820063507259,8.71774358080625,8.71719511568664,8.7160250567648,8.71525501521868,8.71479598818599,8.71444022372288,8.71161244952035,8.71100253743745,8.71083619777848,8.71114289851641,8.71153840995627,8.71188568829371,8.71200144773952,8.71201109436001,8.71245483890229,8.71238690480471,8.71198469705033,8.71099745983503,8.71043985363009,8.710165],"lat":[53.056729,53.056449414733,53.0560160267954,53.0550265776203,53.0547158368276,53.0544964546417,53.0541948023123,53.0539342826921,53.0535007581737,53.0531271357742,53.052594168804,53.0513029328352,53.0503495533777,53.0490953210073,53.048743159569,53.0467432600775,53.0459766073349,53.0453266084587,53.0449696532057,53.0447724720459,53.0445868889539,53.0444593001147,53.0442969137732,53.0440185357644,53.0438294882097,53.0425599729125,53.0394162503141,53.0376244457146,53.036234]}]],[[{"lng":[8.720241,8.72251979788952,8.7235368,8.7235724,8.7237732,8.7242486,8.7247022,8.7256338,8.7267687,8.7274616,8.7277528,8.7282547,8.72845,8.7285111,8.7287741,8.7289148,8.7293442,8.7298512,8.730645,8.7314079,8.7318505,8.7324461,8.7328399,8.7330242,8.7331214,8.73372313545842,8.73426353764331,8.7345083,8.73580994388021,8.73615422859567,8.736295],"lat":[53.056729,53.056879695976,53.0569414,53.0569436,53.056955,53.0569983,53.0570578,53.0572079,53.0573907,53.0575022,53.057549,53.0576296,53.0576609,53.0576707,53.057713,53.0577356,53.057806,53.057887,53.058016,53.0581347,53.0582049,53.0582998,53.0583614,53.0583887,53.0583984,53.0584914365565,53.0586208072177,53.0586777,53.0589857306737,53.0590331491867,53.059044]}]],[[{"lng":[8.736295,8.73810412584788,8.7395878655249,8.74216309186088,8.7427599986275,8.74361272257981,8.74415846590929,8.74470420923876,8.74568334865701,8.74718469576498,8.748328,8.74857495126616,8.74889343997917,8.7490574,8.7492263,8.7501842,8.7505053,8.7507589,8.7508883,8.7509821,8.751074,8.7524345,8.7527624,8.7536209,8.754446,8.7571881,8.758717,8.7603976,8.7605504,8.7606127,8.7608086,8.760915],"lat":[53.059044,53.0595282230611,53.0598562053377,53.0604609161143,53.0607068977303,53.0610656184028,53.0613935889752,53.0617932997343,53.0623424855374,53.0631662511147,53.063687,53.0637573684562,53.0638429271732,53.0638973,53.0639534,53.0643636,53.0645011,53.0646059,53.0646593,53.0646994,53.0647402,53.0653512,53.0654993,53.0658853,53.066269,53.0674681,53.0681367,53.0688938,53.0689764,53.0690092,53.0691125,53.06917]}]],[[{"lng":[8.760915,8.7619707,8.762258,8.7623861,8.7624579,8.7625438,8.7630486,8.7636012,8.7640001,8.7647148,8.7655146,8.7667037,8.7668821,8.7675678,8.76795606426551,8.76952782918287,8.76979292065735,8.77017684624108,8.77595401216766,8.77977498583428,8.78021375792997,8.78065253002566,8.78328516259979,8.78555215176086,8.7889709176731,8.78970220449925,8.79180922466709,8.79385225723764,8.7948533,8.79554372445518,8.7961107,8.7970658,8.7977597,8.7979931,8.7980198,8.7982486,8.7984617,8.79891036124039,8.79941286871935,8.799608],"lat":[53.06917,53.0698535,53.0700368,53.0701185,53.0701561,53.0701963,53.0703725,53.0705566,53.0706685,53.0708262,53.0709938,53.0712356,53.071271,53.071419,53.0714992726131,53.0718290020743,53.0719443357258,53.072317794954,53.070142898665,53.0686269969005,53.0683798339451,53.0681985802096,53.0697035123408,53.0710189117701,53.0730125303425,53.0734656126008,53.0750060566197,53.074459630913,53.0742115,53.0740171228693,53.0738575,53.0735971,53.0734278,53.0733647,53.0733575,53.0732956,53.0732373,53.07311269013,53.0729485720708,53.072859]}]],[[{"lng":[8.799608,8.80095030999394,8.80197411155055,8.80297963093651,8.804305],"lat":[53.072859,53.0735068016606,53.0739186900918,53.0728065822855,53.073386]}]],[[{"lng":[8.804305,8.80532431932284,8.80581336738783,8.80706112553495,8.807392],"lat":[53.073386,53.073869263688,53.0741520917872,53.0749895815741,53.075887]}]],[[{"lng":[8.807392,8.80752002683316,8.80772864586203,8.80772864586204,8.8079813,8.8081377,8.8082585,8.8083564,8.8083812,8.8083961,8.8084916,8.8086069,8.8086543,8.8086698,8.8086893,8.8087361,8.8088627,8.8088858,8.8090116,8.80908142363214,8.8092313021287,8.809272],"lat":[53.075887,53.0758416159592,53.0757521165576,53.0757521165576,53.0755993,53.0754877,53.075381,53.0752733,53.0752428,53.0752246,53.0751073,53.0749628,53.0749223,53.0749125,53.0749001,53.0748717,53.0748173,53.0748101,53.0747708,53.0747519465304,53.0746792616687,53.074656]}]],[[{"lng":[8.860087,8.85985951271213,8.85945745965384,8.859274,8.8587726,8.8586525,8.8586341,8.8585024,8.8581464,8.8579259,8.8576697,8.8573763335119,8.85718933292488,8.85683574239385,8.85620595764141,8.8554471,8.8554471,8.8549543,8.8547291,8.8545062,8.8535213,8.8524333,8.8522435,8.8520849,8.8520206,8.8519875,8.8517234,8.8514575,8.8507674,8.8506956,8.8505921,8.8505283,8.8503893,8.8499292,8.84959203574631,8.8492709,8.848682,8.8484703,8.8482394,8.8480002,8.8477449,8.847552,8.8469902,8.8469214,8.8468389,8.8467339,8.846054,8.845785,8.8454438,8.8453307,8.8452335,8.8450633,8.8449452,8.844889,8.8448481,8.8443321,8.8441483,8.8439901,8.8436012,8.8433159,8.8431169,8.8429511,8.8427867,8.8426221,8.8424211,8.8418814,8.8417759,8.8416946,8.8415423,8.8413186,8.8411419,8.8410463,8.8409538,8.8407171,8.8405052,8.8404713,8.8399618,8.8396483,8.8389963,8.8380303,8.8378858,8.8378297,8.8372441,8.83338049605247,8.8329509310293,8.8324857435408,8.83209816001473,8.8320207,8.8314025,8.8309417,8.8307311,8.8303264,8.8301347,8.8298901,8.829652,8.8295338,8.82939670522694,8.8289142,8.8286695,8.8284745,8.8283504,8.8281491,8.8273673,8.827219,8.8271665,8.82707483226633,8.8270167,8.8267112,8.8265384,8.8264201,8.8263323,8.826223,8.8261191,8.8260151,8.8258657,8.8257277,8.8256031,8.8252172,8.82497094550647,8.8246599,8.824252,8.8240934,8.8239596,8.8235109,8.8234623,8.8232417,8.8230696,8.8225491,8.8221396,8.8219055,8.8216757,8.821497,8.8212967,8.8205033,8.8200953,8.8197468,8.8194257,8.8191971,8.8189904,8.8186251,8.8184177,8.8181588,8.8180094,8.8179098,8.8177171,8.8175103,8.8172769,8.8171966,8.817133,8.8167859,8.8166113,8.8164552,8.8162913,8.8161392,8.8159664,8.815839,8.8156614,8.8155683,8.8153578,8.8152974,8.815196,8.8151047,8.8145437,8.8144555,8.8141588,8.8140754,8.813797,8.8133909,8.8131191,8.8128696,8.8127178,8.8125468,8.8124679,8.8119125,8.8116341,8.8113132,8.8111389505097,8.8109921,8.8106641,8.8104992,8.8104035,8.8101563,8.8101481,8.8100996,8.8099814,8.80965249510668,8.8095812,8.80946515490872,8.80930390036383,8.809272],"lat":[53.068596,53.0686965222489,53.068865130349,53.0689353,53.0691255,53.0691802,53.0691885,53.0692676,53.069498,53.0696726,53.0699192,53.0701402569276,53.0702313989095,53.0703627500748,53.0705004340255,53.0705897,53.0705897,53.0706243,53.0706433,53.0706494,53.0707447,53.0708508,53.0708687,53.0708865,53.0708981,53.0709046,53.0709373,53.0709845,53.0711329,53.0711478,53.0711693,53.0711811,53.0712068,53.071281,53.0713343173622,53.0713851,53.0714789,53.0715117,53.0715453,53.0715707,53.071587,53.0715897,53.0715967,53.0715975,53.0715985,53.071599,53.0716077,53.071609,53.0716087,53.0716086,53.0716109,53.0716265,53.0716515,53.0716689,53.0716816,53.0718843,53.071952,53.0720055,53.07212,53.0722068,53.0722615,53.0723049,53.0723428,53.0723796,53.0724247,53.0725322,53.0725504,53.0725599,53.0725737,53.0725759,53.0725626,53.0725497,53.0725349,53.0724885,53.0724432,53.0724375,53.072333,53.0722706,53.0721396,53.0719425,53.0719144,53.0719031,53.0717836,53.0709715824245,53.0709130392171,53.0709105212279,53.0710115612013,53.0710373,53.0712484,53.0714075,53.0714796,53.0716176,53.071684,53.0717707,53.0718641,53.07191,53.0719652717102,53.0721598,53.0722534,53.0723202,53.0723587,53.072417,53.0726536,53.0727007,53.0727162,53.0727431863008,53.0727603,53.0728539,53.0728992,53.072928,53.0729452,53.0729648,53.0729793,53.0729932,53.0730065,53.0730128,53.073015,53.073025,53.0730293303323,53.0730348,53.0730411,53.0730398,53.0730325,53.0729885,53.0729842,53.0729645,53.0729439,53.0728836,53.0728423,53.0728211,53.0728011,53.072786,53.0727703,53.0726978,53.072654,53.072613,53.0725737,53.072543,53.072515,53.0724638,53.0724383,53.072413,53.0724019,53.0723949,53.0723858,53.0723825,53.0723802,53.0723803,53.0723804,53.0723933,53.0724022,53.0724133,53.0724321,53.0724648,53.0725143,53.0725509,53.0726036,53.0726313,53.0726909,53.072708,53.0727364,53.0727625,53.0729297,53.0729562,53.0730332,53.0730515,53.0731126,53.0732001,53.0732587,53.0733134,53.0733515,53.0734008,53.0734244,53.0735824,53.0736651,53.0737766,53.073840363089,53.0738941,53.0740196,53.0740803,53.0741229,53.0742454,53.0742495,53.0742735,53.0743322,53.0744936116624,53.0745286,53.0745890811461,53.0746432785395,53.074656]}]],[[{"lng":[8.886329,8.88631639629589,8.88626254401257,8.8862364,8.8860785,8.8857527,8.8855075,8.8854345,8.8853224,8.8851625,8.8849988,8.8848883,8.8847572,8.8846497,8.8845523,8.8845221,8.8845035,8.8844807,8.8844764,8.8845242,8.8845597,8.88458856739937,8.8846072,8.8846249,8.8847746,8.8847886,8.8848245,8.88477406244045,8.88466436941652,8.88399707018767,8.88332977095881,8.88231511048753,8.8815198360641,8.88041376473955,8.87894205000193,8.87686702363275,8.87418868563199,8.87339341120856,8.87186684995898,8.87055967475724,8.86948102668868,8.86924335847018,8.8689505,8.8686405,8.8681818,8.867815,8.8677464,8.8671632,8.8668786,8.86639025703059,8.8661362,8.8654386,8.86484089540149,8.8637805990093,8.8636641,8.863547,8.863446,8.8634122,8.8629547,8.8624975,8.8624283,8.8613638,8.8609115,8.86070082405261,8.86011987102495,8.860087],"lat":[53.051802,53.0518089115867,53.0518346436407,53.0518461,53.0518974,53.0519867,53.0520643,53.0521004,53.0521559,53.052284,53.0524321,53.0525505,53.052694,53.0528348,53.0529999,53.0530932,53.0531675,53.0532816,53.0534308,53.053779,53.054007,53.054184641096,53.0542993,53.0544109,53.0552065,53.0552861,53.055477,53.0556586388658,53.0558124747375,53.056433306782,53.0570156720729,53.0577958094671,53.05840562534,53.0588066707138,53.0591527753641,53.0597625720306,53.0607514131429,53.0610755283463,53.0625092968569,53.0638496349884,53.0650745796743,53.0652338748541,53.0654058,53.0655234,53.0656985,53.0658407,53.0658658,53.0660792,53.0661689,53.0663261118541,53.0664079,53.0665945,53.0668011240546,53.0671409138673,53.0671771,53.0672159,53.0672494,53.0672622,53.0674389,53.0676056,53.0676332,53.0680667,53.0682409,53.0683225886589,53.0685776983576,53.068596]}]],[[{"lng":[8.918209,8.91819277305493,8.9181513,8.9181513,8.9179145,8.917715,8.9174825,8.9171676,8.9170919,8.9167923,8.9165149,8.9161775,8.916068,8.9157835,8.9153492,8.9148378,8.9142468,8.9135037,8.9130778,8.9126969,8.9123243,8.9111047,8.9109102,8.9107267,8.9092356,8.9090114,8.9089189,8.9087475,8.9083941,8.90767689484994,8.907447,8.9072754,8.9060319,8.9060163,8.905896,8.9054206,8.9045152,8.9031198,8.9030727,8.9028473,8.9023486,8.9022941,8.901831,8.9013009,8.9008726,8.9005574,8.9004183,8.9002778,8.8999167,8.8996221,8.8993824,8.8986532,8.89842525572772,8.8981711,8.8977132,8.8971718,8.8968851,8.8961306,8.895993,8.8952553,8.8945493,8.8933747,8.8924891,8.8920492,8.88917208694532,8.8875609,8.8875609,8.8874922,8.8874087,8.8873398,8.8871113,8.8870381,8.8868014,8.8865795,8.8864361,8.886361,8.886329],"lat":[53.038871,53.0388755539018,53.038887,53.038887,53.0389339,53.0389622,53.0389814,53.0390015,53.0390064,53.0390223,53.039037,53.0390507,53.0390496,53.0390455,53.039034,53.0390204,53.0390062,53.0389941,53.0390183,53.0390753,53.0391384,53.0393476,53.0393814,53.0394179,53.0396997,53.0397412,53.0397579,53.0397892,53.0398538,53.0399848068117,53.0400268,53.0400581,53.040287,53.0402899,53.040312,53.0404286,53.0406959,53.0411163,53.0411305,53.0411982,53.0413562,53.0413735,53.0414993,53.0416519,53.0418099,53.0419331,53.0420053,53.0420765,53.0422952,53.0424925,53.0427218,53.0434694,53.0436371079625,53.0438241,53.0441279,53.044487,53.0446772,53.0451979,53.0452889,53.0457758,53.0462621,53.0470279,53.0476157,53.0479168,53.0494976485665,53.0504845,53.0504845,53.050541,53.0506186,53.0506845,53.0509032,53.0510067,53.0513948,53.0516625,53.0517494,53.0517915,53.051802]}]],[[{"lng":[8.941825,8.94133113586961,8.94099431083797,8.9406316,8.9405792,8.9401806,8.939918,8.9394206,8.9392944,8.9389499,8.9388534,8.9385688,8.9382278,8.9380497,8.9377525,8.9374342,8.9364599,8.936215,8.9361231,8.9353106,8.9351371,8.9348746,8.93453862777655,8.93425745720162,8.933976,8.933976,8.933737,8.9335918,8.9332957,8.9329737,8.93128088590698,8.92998518173193,8.9296846,8.9288929,8.9279976,8.926982,8.9265087,8.9261111,8.9256645,8.9255135,8.9255056,8.9253543,8.9252807,8.9251643,8.9249425,8.9247234,8.924593,8.9244302,8.9239958,8.9238463438277,8.9237512,8.9235079,8.9230401,8.9224515,8.9222383,8.9216686,8.9210137,8.9206918,8.9203773,8.9201417,8.9199381,8.9197437,8.9196189,8.9194871,8.9191935,8.9190496,8.9188787,8.9187241,8.9185418,8.9183094,8.918209],"lat":[53.038019,53.0381029336738,53.038166887166,53.0382094,53.0382126,53.0382563,53.0382888,53.0383514,53.0383677,53.0384084,53.0384198,53.0384534,53.0384969,53.0385098,53.0385037,53.0384642,53.0383294,53.0383391,53.038341,53.0383681,53.0383738,53.0383752,53.0383868274545,53.0383935094858,53.0383888,53.0383888,53.0384052,53.0384145,53.0384273,53.038437,53.0385017071333,53.0385293199054,53.0385134,53.0384814,53.0384444,53.0384041,53.0383848,53.0383691,53.0383495,53.0383442,53.0383437,53.0383331,53.038328,53.0383153,53.0382845,53.0382322,53.0381901,53.0381338,53.0379835,53.037931440879,53.0378983,53.0378244,53.0377237,53.0376834,53.0376698,53.0376333,53.0375899,53.0375775,53.0375812,53.0376145,53.0376747,53.0377856,53.0378569,53.0379817,53.0383194,53.0384562,53.0385813,53.038672,53.0387557,53.0388408,53.038871]}]],[[{"lng":[8.970547,8.9704001153028,8.97021505707998,8.9689267,8.9688475,8.9685199,8.9676109,8.9674941,8.9672354,8.9668384,8.9666049,8.9663422,8.9661234,8.965911,8.9656019,8.9648773,8.9646292,8.9643658,8.9635775,8.9634169,8.9632477,8.962695,8.96223876842883,8.96200250301107,8.9615588,8.9604265,8.9600595,8.9598834,8.9597228,8.9595894,8.9587102,8.9582664,8.957592,8.9573981,8.9570098,8.9563313,8.9563091,8.9556271,8.9554993,8.9552425,8.95465737053915,8.9545528,8.9544774,8.9539559,8.9538737,8.9536666,8.9529745,8.9527484,8.9524252,8.9522179,8.9520515,8.9515975,8.9515344,8.9509421,8.9503382,8.9500669,8.9499298,8.9496466,8.9492636,8.9488637,8.9484484,8.9480567,8.9473081,8.9470464,8.9469862,8.9464493,8.9458613,8.9456843,8.9454453,8.945265,8.9451119,8.9446787,8.9443307,8.9442826,8.9441752,8.9441167,8.9439552,8.9437773,8.9432822,8.9431045,8.9429884,8.9428478,8.9423904,8.94202786938279,8.941825],"lat":[53.028939,53.0289865209996,53.0290393179281,53.0292113,53.0292241,53.0292643,53.0293757,53.0293958,53.0294403,53.0295274,53.0295926,53.0296774,53.0297567,53.0298482,53.0299984,53.0304079,53.0305336,53.030648,53.030914,53.030966,53.0310239,53.0312128,53.0313592024394,53.0314350186402,53.0315774,53.0319435,53.0320655,53.0321241,53.032175,53.0322172,53.0325001,53.0326429,53.0328676,53.0329298,53.0330544,53.0332754,53.0332824,53.0335052,53.0335469,53.0336308,53.0338218558311,53.033856,53.0338806,53.0340472,53.0340777,53.0341482,53.0343701,53.0344426,53.0345462,53.0346127,53.034666,53.0348116,53.0348318,53.0350217,53.0352233,53.0353207,53.0353754,53.0355137,53.0357051,53.0359152,53.0361317,53.0363409,53.0367484,53.0368793,53.0369123,53.0371815,53.0374846,53.0375611,53.0376526,53.0377151,53.0377597,53.0378614,53.0379282,53.0379379,53.0379592,53.0379676,53.0379909,53.038012,53.0380528,53.0380601,53.0380649,53.0380659,53.0380647,53.0380355261054,53.038019]}]],[[{"lng":[9.032515,9.0308066,9.0307533,9.0303882,9.0301268,9.0299843,9.029894,9.029625,9.0294181,9.0290243,9.028613,9.0284716,9.0280836,9.027403,9.026898,9.026094,9.0257179,9.0255197,9.0252883,9.0250378,9.0239077,9.0237889,9.0219542,9.0211185,9.0206761,9.0204229,9.0197311,9.0192808,9.0192171,9.0189685,9.0183838,9.0181415,9.0180325,9.0173646,9.0172696,9.0163765,9.0160982,9.0158085,9.0153933,9.0142137,9.0134133,9.0129198,9.0123102,9.0103988,9.0103791,9.0099804,9.0098604,9.0086986,9.0078321,9.0066891,9.0051725,9.0041787,9.0032715,9.0017619,8.9999282,8.99975547186457,8.9986584,8.9982251,8.9979973,8.9970871,8.9969136,8.9958517,8.9952035,8.9948669,8.9938279,8.9936124,8.9924807,8.992177,8.991745,8.9909776,8.988935,8.987684,8.9871944,8.9868982,8.9865694,8.9863134,8.9859202,8.9852761,8.985065,8.9848896115037,8.9847979,8.984503,8.9838155,8.9835434,8.9834071,8.98305,8.982613,8.9822854,8.9819386,8.9815433,8.9811907,8.9793153,8.9785651,8.9785122,8.9783918,8.9781729,8.9775108,8.97729,8.9770461,8.9769376,8.9764334,8.9757702,8.9756508,8.9755544,8.9750947,8.9745771,8.974098,8.9737232,8.9733422,8.9730178,8.9713948,8.97129528803482,8.971042,8.970547],"lat":[53.01297,53.0126223,53.0126107,53.0125315,53.0124699,53.0124376,53.0124199,53.0123766,53.0123504,53.0123435,53.0123841,53.0124028,53.0124516,53.012549,53.0126179,53.0127212,53.0127802,53.012824,53.0128953,53.012991,53.0134348,53.0134828,53.0142105,53.014542,53.0147171,53.0148173,53.0150911,53.0152693,53.0152945,53.0153912,53.0156227,53.0157182,53.0157621,53.0160308,53.0160682,53.0164102,53.0165167,53.0166186,53.0167706,53.0170432,53.0172381,53.0173583,53.0175041,53.0179493,53.0179539,53.018043,53.018071,53.0183468,53.0185537,53.0188219,53.0191796,53.0194124,53.0196285,53.019981,53.0204129,53.020453790088,53.0207135,53.0208161,53.0208682,53.0210757,53.0211163,53.0213624,53.0215189,53.0215995,53.0218436,53.0218931,53.0221487,53.0222169,53.0223249,53.0224987,53.022976,53.0232833,53.0234161,53.0235117,53.0236313,53.0237301,53.0238983,53.0241793,53.0242726,53.0243501491993,53.0243907,53.0245211,53.0248148,53.0249282,53.0249834,53.0251166,53.0252608,53.0253565,53.0254486,53.0255414,53.0256214,53.0259939,53.0261459,53.026156,53.0261804,53.0262244,53.0263585,53.0264005,53.026448,53.0264671,53.0265793,53.0267371,53.0267736,53.0268031,53.0269582,53.0271512,53.0273512,53.0275283,53.0277084,53.0278617,53.0286074,53.0286532353634,53.0287699,53.028939]}]],[[{"lng":[9.054039,9.0539232464372,9.05373652471005,9.05353107878106,9.0512047,9.0499818,9.0489883,9.04568944229566,9.0445361,9.0438315,9.04354529487415,9.0435074,9.0430235,9.04290297914774,9.04280899032416,9.0426033,9.0420357,9.0413645,9.0410954,9.0395732,9.0389438,9.0388885,9.0387887,9.0386987,9.0385158,9.0381135,9.0375041,9.0367557,9.0365687,9.0365258,9.0362627724292,9.03380823118458,9.032515],"lat":[53.029817,53.0297816613471,53.0297013615593,53.0295795341023,53.0279793,53.027138,53.0265228,53.0244797213154,53.0237654,53.0233093,53.0230250377667,53.0229874,53.0225087,53.0223745558194,53.0222699424817,53.022041,53.0214165,53.020678,53.0203663,53.018639,53.0180105,53.017954,53.0178504,53.0177444,53.0175484,53.0171032,53.0164337,53.0155946,53.0152977,53.0152127,53.0146114245583,53.0136579007547,53.01297]}]],[[{"lng":[9.085822,9.08579227697187,9.08572653163695,9.0856129,9.0854135,9.0852623,9.085035,9.084982,9.0848785,9.0847548,9.084784,9.08528476573549,9.0857914,9.0858371,9.0860027,9.0860275,9.0860988,9.0864844,9.0865604,9.0866856,9.0867675,9.0866592,9.08662766546173,9.08660483359212,9.0865686,9.0864682,9.0863373,9.0862453,9.085982,9.0858108,9.0855928,9.0853617,9.0851273,9.0837842,9.082788,9.0815999,9.0803908,9.0796573,9.0794994,9.0770513,9.0757635,9.0752457,9.0745649,9.0738078,9.0734664,9.06979523498886,9.0692805,9.068985,9.0682378,9.067887,9.0668393,9.0644248,9.0623267,9.0612391,9.0604702,9.05877089496952,9.0575562,9.0561637,9.0549434,9.05438276562692,9.05418994986646,9.054039],"lat":[53.06366,53.0636354483843,53.0635753755123,53.0634817,53.0633112,53.0631698,53.0629478,53.0628919,53.0627878,53.0626634,53.0623643,53.0616162098576,53.0607895,53.0607104,53.0604188,53.0603789,53.0602645,53.0596384,53.0595149,53.0591778,53.0587328,53.0580518,53.0578583124051,53.0577182215434,53.0574959,53.0568792,53.0565246,53.0562755,53.0556998,53.055418,53.0551334,53.0549148,53.0547389,53.0540371,53.0535645,53.0530246,53.0524392,53.0520024,53.0519068,53.0503782,53.0495926,53.0491873,53.048417,53.0475331,53.0471609,53.0428944357972,53.0422962,53.0419409,53.0410975,53.040726,53.0399152,53.0381268,53.0365923,53.0357771,53.0351898,53.033852014581,53.0330673,53.0317414,53.030528,53.0301552983868,53.0300000935982,53.029817]}]],[[{"lng":[9.143259,9.14322466689481,9.14311245929506,9.14280388839573,9.1426355769961,9.1409299,9.1406622,9.1403606,9.1400092,9.1399129,9.1395597,9.1378861,9.1377283,9.1368583,9.1364714,9.1361632,9.131835,9.129994,9.129919,9.1296268,9.1291713,9.1287649,9.1276096,9.1269482,9.1267646,9.1265502,9.126079,9.1257112,9.1253614,9.1242542,9.12405821816028,9.1240021,9.1236234,9.1221336,9.1217192,9.1216164,9.1212234,9.1210377,9.1208572,9.12052642191024,9.1195334,9.1194476,9.1183561,9.1169238,9.1164746,9.1160169,9.114777,9.1140314,9.1107682,9.1064727,9.1050263,9.1047501,9.104517,9.1040379,9.1037844,9.1034404,9.1030226,9.10291703299516,9.1006495,9.0970947,9.0958896,9.0926606,9.0914011,9.09127559463064,9.09078957369922,9.09048352003454,9.089999,9.0898787,9.089642,9.0891561,9.088609,9.0883085,9.0880313,9.087741,9.0876033,9.0866133,9.0865873,9.08618530568662,9.085822,9.085822],"lat":[53.110276,53.1097395657521,53.1093606840054,53.1086534291468,53.108249278291,53.1071113,53.1069665,53.1068262,53.1067117,53.1066827,53.1065999,53.1062986,53.1062617,53.105996,53.1058019,53.105607,53.1026744,53.1014771,53.1014252,53.1012416,53.1009406,53.1006881,53.0999648,53.0995316,53.0993937,53.0992261,53.0988088,53.0984238,53.0979699,53.0965112,53.0962531043439,53.0961792,53.0956803,53.0937175,53.0931715,53.0930361,53.0925182,53.0922736,53.0920358,53.0915999581431,53.0902915,53.0901785,53.0887404,53.086853,53.0863341,53.0858932,53.0849796,53.08443,53.0820248,53.0788585,53.0777922,53.0775886,53.0774168,53.0770635,53.0768767,53.0766231,53.0763151,53.0762372752603,53.0745656,53.0719448,53.0710563,53.0686755,53.0677468,53.067654257835,53.0672869428197,53.0670702782869,53.066713,53.0666242,53.0664497,53.0660914,53.065688,53.0654664,53.065262,53.0650479,53.0649464,53.0642164,53.0641972,53.0639114283474,53.06366,53.06366]}]],[[{"lng":[9.146686,9.14904543613205,9.15044803112898,9.15120543242732,9.15140179572689,9.1482132,9.1480033,9.1473363,9.1468655,9.1458365,9.1444917,9.1440738,9.1433882,9.1433041,9.1431838,9.1430846,9.1430349,9.1429899,9.1429049,9.1428083,9.1422157,9.1421576,9.1420451,9.1419435,9.1418534,9.1417161,9.1414762,9.1413528,9.1413129,9.1412872,9.1412184,9.1412158,9.141159,9.1410963,9.1410554,9.1410539,9.1409967,9.1409895,9.140986,9.14100168731322,9.1410851,9.1412157,9.1414062,9.1415908,9.1417176,9.1420058,9.1422902,9.1424234,9.143259],"lat":[53.140233,53.1398373637291,53.1393830235048,53.1392484032562,53.1380536300634,53.1342638,53.1336533,53.1319445,53.1307384,53.1284671,53.125895,53.1251426,53.123292,53.1229713,53.1225188,53.1220407,53.1218702,53.1217331,53.1215755,53.1214097,53.1206048,53.1205259,53.1203564,53.1201759,53.1198612,53.118978,53.1168812,53.1154243,53.1150812,53.1145635,53.1140661,53.1139018,53.1134016,53.1128166,53.1124358,53.1124117,53.1114874,53.1113445,53.1112754,53.1111906610556,53.1111738,53.1111498,53.1111148,53.1110738,53.1110334,53.1109195,53.1107559,53.110675,53.110276]}]],[[{"lng":[9.203015,9.20103612618065,9.20034885463216,9.19978781663339,9.19658990004039,9.19591665444187,9.19191925870063,9.18412083051772,9.1832799,9.1828754,9.1817071,9.1813706,9.1795441,9.1782245,9.17786525683142,9.17789330873136,9.17881902142933,9.17884707332927,9.17859460622982,9.169842413449,9.16355878786277,9.16038892316972,9.1584583,9.156605,9.1561521,9.1557417,9.15519826257128,9.1551191,9.1549283,9.1547917,9.1543433,9.1538617,9.1528783,9.1526933,9.1521434,9.1514815,9.1512031,9.15104348542341,9.1506949,9.1505533,9.1503126,9.1498667,9.148605,9.1479531,9.1478381,9.1475683,9.147135,9.1469835,9.1469179,9.1468433,9.146865,9.1468783,9.14689150428439,9.1469117,9.1469504,9.1469733,9.1470035,9.1470141,9.14701,9.1469421,9.146686],"lat":[53.218914,53.2177906303544,53.2173791073985,53.2171355511337,53.2165728468814,53.2161445148093,53.2108026021674,53.1948737021786,53.1930447,53.1927634,53.1919072,53.1916606,53.1901567,53.1891024,53.1885710790696,53.1881004460395,53.1871759732635,53.1866717069853,53.1862682896933,53.1809898971529,53.1778628996455,53.1761984370481,53.175025,53.1714733,53.1698776,53.1684317,53.1657057096671,53.1653086,53.1645583,53.1640633,53.161945,53.1599033,53.1573567,53.15684,53.1556099,53.153761,53.1530482,53.1527002671718,53.1519404,53.1516317,53.1512742,53.1504867,53.148365,53.1472552,53.1470593,53.1466,53.1459367,53.1456515,53.1455022,53.1449817,53.1447283,53.1441583,53.1438150302378,53.14329,53.1428303,53.1425117,53.1419021,53.1412709,53.1410317,53.1407676,53.140233]}]],[[{"lng":[9.257674,9.25766739821405,9.25764065117808,9.2572673,9.2567278,9.2566213,9.2565714,9.2565604,9.2556789,9.2548007,9.2542385,9.253705,9.2535507,9.2527485,9.2527279,9.2523339,9.251316,9.2504878,9.2498176,9.2495337,9.2482609,9.2477533,9.2473199,9.2473038,9.2468786,9.2465095,9.2462554,9.2461318,9.2460901,9.2460152,9.2459291,9.2458432,9.2457022,9.2450316,9.2441131,9.2440209,9.2438045,9.2435408,9.2433477,9.2431311,9.2297277,9.2278768,9.2241971,9.2211088,9.216424,9.215836,9.2153542,9.2148856,9.2145042,9.2141769,9.2139783,9.2137844,9.2134734,9.2131978,9.2129158,9.212523,9.2121556,9.2119067,9.2116908,9.2116286,9.2116165,9.2116481,9.2117148,9.2117813,9.2120409,9.2126858,9.2127239,9.21279728953294,9.2128639,9.2130669,9.2131083,9.2130188,9.2129761,9.2128967,9.2122167,9.2122017,9.2114097,9.211215,9.2108384,9.2106565,9.2087187,9.2075937,9.2072525,9.2069425,9.20593,9.2056964,9.2053136,9.2047168,9.204404,9.2039144,9.2036372,9.203329,9.20315986687131,9.203015],"lat":[53.273983,53.2739509844833,53.2738760007209,53.2732267,53.2722886,53.2721034,53.2720167,53.2719975,53.270739,53.2696464,53.268853,53.2681751,53.267979,53.2669231,53.266896,53.2663772,53.2650366,53.263991,53.2631598,53.2628489,53.2617542,53.2613233,53.2609361,53.2609187,53.2604572,53.2598411,53.2593732,53.2589066,53.2582055,53.2572527,53.2564386,53.2558224,53.2551065,53.254591,53.2539028,53.2538404,53.2537083,53.2535902,53.253521,53.2534657,53.2510313,53.2506951,53.2500079,53.2494408,53.2486131,53.2485063,53.2484171,53.248308,53.2481817,53.2480298,53.2479121,53.2477702,53.2474554,53.2470467,53.2466042,53.2459049,53.2452216,53.2447467,53.2442231,53.243831,53.2433786,53.2428639,53.2421752,53.2415779,53.2400095,53.2359921,53.2357219,53.2351509328897,53.2346327,53.2333296,53.232945,53.2325305,53.2323326,53.231965,53.23056,53.2305283,53.2288517,53.2285612,53.2280523,53.2278302,53.2256921,53.2243679,53.2239663,53.2236014,53.2224095,53.2221345,53.2216839,53.2208971,53.2204864,53.2198841,53.219543,53.2192447,53.2191013058134,53.218914]}]],[[{"lng":[9.274875,9.27455402332585,9.27284565475866,9.27192327732532,9.26346562949386,9.26228744969644,9.25880901410406,9.25023915867284,9.25099655997118,9.25265162206755,9.25303032271672,9.25426460631402,9.25575135701076,9.25616185371333,9.2565818,9.2573555,9.2575383,9.25761775024945,9.25771672977242,9.257674],"lat":[53.296491,53.2963937313464,53.2959384554424,53.2957154634123,53.2946340296509,53.2930747048652,53.2885976174035,53.2773525257535,53.2764383651475,53.2762706271337,53.2762119186733,53.2758596662178,53.2751048297523,53.2749413132214,53.2747879,53.2745271,53.2743903,53.2742498335895,53.2740748391786,53.273983]}]],[[{"lng":[9.278606,9.27784924118734,9.27746352756318,9.27712690476392,9.2767271651898,9.27632742561567,9.27607010798505,9.27584084640244,9.27547106965629,9.274875],"lat":[53.295483,53.2953885212091,53.2953298390103,53.2953088810626,53.295342413774,53.2954178622783,53.29551411625,53.2956599833229,53.2960047580605,53.296491]}]],[[{"lng":[9.334932,9.33491678305797,9.33489362606441,9.3348252,9.3347427,9.3346439,9.3344352,9.3342187,9.3339173,9.3337801,9.3336191,9.3334408,9.3327883,9.332488,9.3322723,9.3313915,9.33076,9.3295092,9.3293703,9.3287867,9.3287308,9.3281502,9.328104,9.3272086,9.3262182,9.3251674,9.3228075,9.3221911,9.3215421,9.320981,9.3207986,9.3200656,9.319766,9.317483,9.3153864,9.3143897,9.3110682,9.3064315,9.3018844,9.301007,9.3001457,9.2991934,9.2981018,9.2980743,9.2971314,9.2954791,9.2951577,9.2944595,9.2935668,9.291556,9.2914171,9.2897413,9.2890891,9.2889121,9.2887491,9.2871126,9.2870454,9.2853883,9.2848244,9.2841586,9.2840538,9.2839256,9.2837082,9.2831841,9.28274429772661,9.28247780467719,9.28228144137763,9.28187468882851,9.28135572867965,9.28075261283097,9.28024767863208,9.27971469253325,9.27908352478463,9.2787328760354,9.27856456463577,9.27866274628555,9.278606],"lat":[53.314912,53.3148484847608,53.3147876127377,53.3146678,53.3145364,53.3144107,53.3141746,53.3139762,53.3137581,53.3136774,53.3135864,53.3134945,53.3131959,53.3130741,53.3129802,53.312505,53.3121389,53.3114142,53.3113337,53.31099,53.3109568,53.3106095,53.3105846,53.3100683,53.3094846,53.3088761,53.3075034,53.3071633,53.3068723,53.3066745,53.3066049,53.3063995,53.3063208,53.3057249,53.3051954,53.3049359,53.3040711,53.3028654,53.3016878,53.3014679,53.3012355,53.3009688,53.3006341,53.3006257,53.3003394,53.2998399,53.2997432,53.2995291,53.2992516,53.2986289,53.2985859,53.2980874,53.2978936,53.2978377,53.2977885,53.2972826,53.2972624,53.296769,53.296613,53.2964373,53.2964117,53.2963873,53.2963506,53.2962722,53.296034020075,53.2941645615108,53.2940471936696,53.294022043376,53.2940723439484,53.2942567788741,53.2945082797621,53.2946843295027,53.2947933123118,53.2948519952477,53.2948436119761,53.2952711567309,53.295483]}]],[[{"lng":[9.373245,9.37322826918041,9.37319727199731,9.3729354,9.3725954,9.3724351,9.3723068,9.372184,9.3720749,9.3718297,9.3715628,9.3710243,9.37086688183151,9.370502,9.3698665,9.3675747,9.3653553,9.3642284,9.3631121,9.3620725,9.3616969,9.3615141,9.3612095,9.3598971,9.3596616,9.3593958,9.358705,9.3574664,9.3558501,9.3554684,9.354449,9.3534995,9.3531908,9.3515208,9.3498593,9.3487649,9.34750364589164,9.3454495,9.345363,9.3444528,9.3426351,9.3423123,9.3420357,9.3411708,9.3409788,9.3404502,9.3404073,9.3400456,9.3394634,9.3378621,9.3369104,9.3364063,9.3362003,9.3360472,9.3358268,9.3355853,9.33516928082808,9.334932],"lat":[53.336027,53.3359962344443,53.335943985219,53.3356263,53.3352608,53.3350885,53.3349623,53.3348748,53.3348142,53.334707,53.3346153,53.3344545,53.3344082963336,53.3343012,53.3341208,53.3334642,53.3328111,53.3324919,53.332162,53.3318475,53.331715,53.3316236,53.3314308,53.3303992,53.3302449,53.3300834,53.3296804,53.3289706,53.3281019,53.3278968,53.3273264,53.3267833,53.3266125,53.3256756,53.3247409,53.3241178,53.3234087773785,53.322254,53.3222068,53.3216986,53.3206642,53.3204805,53.3203269,53.3198467,53.3197286,53.3194331,53.3194091,53.3192068,53.3188729,53.3179624,53.3174212,53.3171481,53.3170275,53.3169167,53.3164573,53.3159947,53.315268906321,53.314912]}]],[[{"lng":[9.398899,9.39870058614849,9.39843318464667,9.3980717,9.3979082,9.3975088,9.3971127,9.395699,9.3949427,9.3938958,9.3937236,9.3918868,9.3911034,9.38823846147598,9.3878882,9.3868096,9.3849192,9.3847324,9.3838668,9.3837167,9.3824417,9.3817927,9.3811456,9.379195,9.3785526,9.37838495756261,9.3770303,9.376501,9.376299,9.3758858,9.3757791,9.3753939,9.3748113,9.3737265,9.3735143,9.3733696,9.373245],"lat":[53.346258,53.3462746210852,53.3462225458669,53.3461192,53.3460693,53.3459513,53.3458112,53.345223,53.3449022,53.3444429,53.344376,53.3436106,53.3432852,53.3420857451867,53.3419391,53.3414778,53.3406991,53.3406314,53.3403178,53.340265,53.3398551,53.3396565,53.3394698,53.3388598,53.3386577,53.3386061399883,53.3381895,53.338019,53.3379535,53.3378117,53.3377725,53.337603,53.3373191,53.3367198,53.3365058,53.3362497,53.336027]}]],[[{"lng":[9.426945,9.42705729696038,9.42655236276149,9.42507963801472,9.4245465,9.42380327656751,9.4228074341197,9.42213418852117,9.4209197,9.4206999,9.4204769,9.4203125,9.4200607,9.4194554,9.4189052,9.4180696,9.4172376,9.4155924,9.4150742,9.4145373,9.4139164,9.41346,9.40841680945123,9.40698616255436,9.4051908409583,9.4020305,9.4010131,9.4007863,9.4005183,9.4000756,9.4000574,9.3999801,9.3998689,9.3996694,9.3992747,9.39898659224488,9.398899],"lat":[53.365284,53.3651297430717,53.3639245220963,53.3641839822693,53.3646648,53.3648786581799,53.3648702886581,53.364451810468,53.3636966,53.3635452,53.3633283,53.363162,53.3629419,53.3623954,53.3618389,53.3610611,53.3602779,53.3586899,53.3582245,53.3577906,53.3574008,53.3571465,53.3525820577186,53.3519457858035,53.3515271806797,53.3515995,53.3502821,53.3498553,53.3492824,53.3482549,53.3474392,53.3471369,53.346996,53.3468349,53.3465395,53.346352755891,53.346258]}]],[[{"lng":[9.465251,9.4649287718509,9.46368875279296,9.4613974,9.4581267,9.4536552,9.4530482,9.44971,9.446154,9.4430594,9.4373442,9.4345286,9.4335824,9.43185305552393,9.43098511456417,9.430928,9.4303497,9.4300497,9.4296651,9.4289529,9.4282746,9.4280944,9.4278769,9.4275959,9.4273228,9.427004,9.426945],"lat":[53.378304,53.3781783625478,53.3777878207096,53.3771011,53.3762574,53.3750404,53.3748332,53.3737895,53.3726444,53.3717229,53.3700465,53.3692305,53.3689379,53.368454331369,53.3681054100662,53.3680543,53.3675942,53.3673705,53.3670802,53.3666194,53.3661619,53.3660458,53.3659123,53.3657165,53.3655448,53.3653463,53.365284]}]],[[{"lng":[9.549767,9.54959425168456,9.54920392265072,9.54745565263786,9.546555,9.54578553292418,9.545354,9.5419381,9.5410015,9.53217002622787,9.5153469,9.514266,9.513562,9.5124814,9.51152088360561,9.5104086,9.5100507,9.508676,9.5079752,9.5072868,9.506594,9.5060823,9.5055535,9.5045982,9.5035551,9.5030715,9.5023784,9.4990633,9.4974484,9.4965969,9.49624703536197,9.4957249,9.4951085,9.4948495,9.494651,9.4944288074056,9.4942476,9.492428,9.4916717,9.4914152,9.49141465007653,9.4911547,9.4904453,9.4893981,9.4888463,9.4882057,9.4880992,9.4876197,9.4869085,9.4868039,9.4861071,9.4853447,9.485282,9.4850463,9.4835387,9.4834292,9.4818014,9.47995,9.4796854,9.4794271,9.4789343,9.4784862,9.4762307,9.4747173,9.4713858,9.4712686,9.4711506,9.4706279,9.4685746,9.4682217,9.468015,9.4676683,9.4669908,9.4664115,9.465833,9.465251],"lat":[53.414627,53.4145529238833,53.4144144872684,53.4138703333587,53.41359,53.4133397474239,53.4131994,53.4121268,53.4119428,53.4104553314664,53.4076217,53.4073425,53.4070661,53.4066418,53.4062254457497,53.4057433,53.4055881,53.4050491,53.4047948,53.4046134,53.4044911,53.4044222,53.4043699,53.4042917,53.4042131,53.4041752,53.4041148,53.4037278,53.403531,53.4033665,53.4032662751916,53.4031167,53.4028707,53.4027348,53.4026199,53.4024747644839,53.4023564,53.401072,53.4004338,53.4002174,53.4002167153954,53.3998931,53.3990097,53.3977814,53.3971292,53.3963421,53.3962114,53.3956231,53.3947569,53.3946295,53.3937808,53.3928825,53.3928196,53.3926558,53.3916086,53.3915325,53.3904015,53.389143,53.3889631,53.3887915,53.3884197,53.3880789,53.3859056,53.3844381,53.3812013,53.3810927,53.3810051,53.380675,53.379512,53.3792924,53.3791848,53.379045,53.3788159,53.3786292,53.3784428,53.378304]}]],[[{"lng":[9.68523,9.68511670539378,9.68492982966922,9.6843515,9.6827387,9.6824895,9.6822365,9.6819348,9.68132585412475,9.6812631,9.6808414,9.6806143,9.6803739,9.6799359,9.6786342,9.6779266,9.67772295111862,9.67759762003358,9.6774722,9.6774058,9.6772254,9.677125,9.6755867,9.6755332,9.6752836,9.6748912,9.6728757,9.6723241,9.6720585,9.671833,9.6700363,9.6690796,9.6686664,9.6683156,9.6680193,9.6675549,9.6673662,9.6667675,9.6665064,9.6656595,9.6652676,9.6643312,9.6640269,9.6636733,9.6628186,9.6627531,9.6626455,9.6622896,9.6617098,9.6610951,9.6609348,9.6608892,9.660842,9.6595948,9.659325,9.6591327,9.65738519074302,9.6569378,9.6566172,9.655453,9.6543621,9.65368980318985,9.6501615852236,9.64961412855881,9.64863193498964,9.64824495343225,9.64807409339908,9.64737203637629,9.64658057388449,9.64611137503289,9.64587777714997,9.64581126024423,9.64561247556666,9.64106586248814,9.63890443162603,9.63770676336864,9.63562854953036,9.63421903531841,9.63120578689681,9.63085510837634,9.62946064125195,9.62773098666798,9.62748828919914,9.62699145291042,9.62621170481019,9.62571612296165,9.62488219263891,9.62411263382919,9.62367879785179,9.62320631659805,9.62269482669901,9.62232218391653,9.62147520325342,9.6212005233097,9.61960907058225,9.61765906685863,9.61443596031484,9.61346053666825,9.61181378207602,9.61176038099225,9.60902897719129,9.60758782516566,9.60509424268922,9.60413137093594,9.60315252890264,9.60201044893192,9.60004433280091,9.59641296366956,9.59525423625229,9.59453039192489,9.59275295936387,9.59022928343444,9.58826490366199,9.58817579037504,9.58718942548974,9.58684565568782,9.58561465025777,9.58520694431524,9.58492538903809,9.58428501427974,9.5841083,9.58353571982899,9.5820447,9.57935632941583,9.5781585,9.5744532,9.57413251271752,9.5728596,9.5710198,9.5704413,9.56877645465016,9.5677668,9.5634251,9.5626259,9.5594904230188,9.5573861,9.5569862,9.5565959,9.55634050156367,9.5562121,9.55600392263607,9.5558754,9.5554043,9.55522965529618,9.5550882,9.55459644854342,9.5541753,9.5541304,9.5537934,9.5535351609305,9.5534984,9.5529525,9.5526079,9.5521492,9.5514083,9.5511991,9.55009633631431,9.54996448996556,9.549767],"lat":[53.468829,53.4687759870364,53.468706444658,53.4685539,53.468138,53.4680657,53.467972,53.4678398,53.4674784396912,53.4674412,53.4672253,53.467109,53.4670099,53.4668819,53.466612,53.4664798,53.4664391957355,53.4664142067426,53.4663892,53.4663752,53.4663371,53.4663207,53.466059,53.4660513,53.4660019,53.4659186,53.4655246,53.4654067,53.4653502,53.4653025,53.4648941,53.4646366,53.4644928,53.4643707,53.4642634,53.4641031,53.4640353,53.4638364,53.46376,53.4634973,53.4633496,53.4629967,53.4628417,53.4626617,53.4621439,53.4621042,53.462036,53.4618095,53.4614406,53.4610653,53.4609748,53.4609467,53.4609176,53.4601494,53.4599846,53.4598672,53.4588359267288,53.4585719,53.4583827,53.4576613,53.4570103,53.4566032260308,53.4544668352531,53.4541353321073,53.4537219903941,53.4535591337943,53.4534872291766,53.4531917747087,53.4528586922778,53.4526612314103,53.4525536965075,53.4525230759362,53.4524315667315,53.4503385098008,53.4495253266908,53.449074727921,53.4479401624794,53.4471706449601,53.4457195269446,53.4455506442138,53.4447911697912,53.4438491219645,53.4437169363155,53.4435571631795,53.4433064097543,53.4431470384833,53.4428788583539,53.4426313776856,53.4424918607244,53.4423399153388,53.4421754245389,53.4420403571883,53.4417333606902,53.441633799758,53.4410569546631,53.4403501368884,53.4390547823776,53.4386627554644,53.4380921236763,53.4380736190567,53.4371271183786,53.4366277141616,53.4354657456383,53.4350170546956,53.4347646679281,53.4346102296357,53.4344583772802,53.4342598782332,53.4340985263067,53.4339079620283,53.4332130415947,53.431801353442,53.4299718231625,53.4298888254005,53.4288903728804,53.4285423847611,53.4279549842182,53.4277604368272,53.4276260849554,53.4274073885684,53.4273491,53.4271592584246,53.4266649,53.42579430689,53.4254064,53.4241208,53.4240155951256,53.423598,53.4229784,53.4227892,53.4222352497201,53.4218993,53.4204464,53.4201808,53.4191278107649,53.4184211,53.4182757,53.418112,53.4179900237112,53.4179287,53.4178234057381,53.4177584,53.4174565,53.4173376577562,53.4172414,53.4168581919418,53.41653,53.416497,53.4162493,53.4160821887303,53.4160584,53.4157543,53.4155902,53.4154024,53.4151284,53.415053,53.4146953627955,53.4146636001092,53.414627]}]],[[{"lng":[9.68523,9.68561363088715,9.68642852370572,9.687005,9.6874282,9.6879931,9.6887152,9.6891267,9.6892017,9.6893753,9.68969648910201,9.689763],"lat":[53.468829,53.4688521838624,53.4689741506244,53.4690814,53.4691747,53.4693294,53.4696393,53.4697861,53.46981,53.4698653,53.4700893486894,53.470173]}]],[[{"lng":[9.689763,9.6901408,9.6902229,9.690245,9.6903745,9.6904701,9.6906035,9.6908626,9.6910414,9.6910609,9.6910904,9.6914334,9.691759,9.6921269,9.6922955,9.6924616,9.6925634,9.6933301,9.6940884,9.6941578,9.6942262,9.6943739,9.6945307,9.6946436,9.6947661,9.6948713,9.6949382,9.6950852,9.6954417,9.6954822,9.6956423,9.695729,9.6957799,9.6960265,9.6960524,9.6964603,9.6965168,9.6965884,9.6967965,9.696934,9.6970539,9.6977576,9.6978379,9.6979164,9.698046,9.6984258,9.6990435,9.7000891,9.7001062,9.70031984201682,9.700347],"lat":[53.470173,53.4705519,53.470647,53.4706726,53.4707966,53.4708783,53.4709945,53.4712202,53.471404,53.4714219,53.4714408,53.4717848,53.4719808,53.4722069,53.4723105,53.4724066,53.4724655,53.4729291,53.4734011,53.4734382,53.4734739,53.4735607,53.4736528,53.4737218,53.4737975,53.4738627,53.4739052,53.4739981,53.4742256,53.4742512,53.4743525,53.4744069,53.4744387,53.4745912,53.474607,53.4748558,53.4748838,53.4749254,53.4750325,53.4750984,53.475161,53.4754094,53.4754343,53.4754571,53.4754934,53.4755999,53.4758356,53.4762378,53.4762481,53.4763578587173,53.476376]}]],[[{"lng":[9.700347,9.70036581927904,9.70041371264451,9.7010497,9.7011359,9.7014693,9.7015047,9.70198580165638,9.70208818146746,9.702158],"lat":[53.476376,53.4763853456869,53.4764110309629,53.4767932,53.4768411,53.4770874,53.4771067,53.47730202438,53.4773564806571,53.477427]}]],[[{"lng":[9.702158,9.70208818146746,9.70206794872633,9.70218934517314,9.70240178895506,9.70284690926004,9.70325156408275,9.70337296052957,9.7033931932707,9.70345389149411,9.70363598616433,9.70448576129202,9.70527483819631,9.70594251865378,9.70642810444103,9.70701485393396,9.70788486180279,9.70826928388436,9.70847161129572,9.70871440418935,9.70966534302272,9.71047465266814,9.71156722068945,9.71185047906535,9.71225513388806,9.71274071967532,9.71332746916825,9.71359049480301,9.71367142576755,9.71379282221436,9.71379282221436,9.71387375317891,9.7141570115548,9.71510795038817,9.71555307069315,9.7163219148563,9.71686819886696,9.71745494835989,9.71794053414714,9.71838565445212,9.71889147298051,9.71972101536707,9.72036846308341,9.72055055775362,9.72135986739904,9.7218454531863,9.72241196993809,9.72312011587784,9.72441501131051,9.72524455369707,9.72585153593113,9.72619549253043,9.72649898364747,9.72660014735315,9.72651921638861,9.72613479430703,9.7261752597893,9.72635735445952,9.72666084557655,9.72710596588153,9.72757131892765,9.72795574100923,9.72908877451281,9.72963505852348,9.72993854964051,9.73024204075754,9.73066692832139,9.7311322813675,9.73123344507318,9.73117274684977,9.73127391055545,9.73139530700226,9.73171903086043,9.7320427547186,9.73242717680018,9.73307462451651,9.73347927933922,9.73364114126831,9.73374230497398,9.73360067578603,9.73339834837468,9.73309485725765,9.7323260130945,9.73109181588523,9.73072762654479,9.7305050663923,9.73044436816889,9.73054553187457,9.7308490229916,9.73103111766182,9.73103111766182,9.73076809202706,9.73070739380366,9.73074785928593,9.73082879025047,9.73082879025047,9.73068716106252,9.73044436816889,9.73010041156959,9.72943273111212,9.72856272324329,9.72821876664399,9.72779387908014,9.72744992248084,9.72724759506948,9.7270857331404,9.72706550039926,9.72706550039926,9.72700480217586,9.7268024747645,9.72668107831769,9.72653944912974,9.72655968187087,9.72666084557655,9.72678224202336,9.72694410395245,9.72747015522197,9.72795574100923,9.7286841196901,9.72912923999508,9.72961482578234,9.72999924786391,9.73038366994549,9.73068716106252,9.73143577248453,9.73254857324699,9.73289252984629,9.73311508999878,9.73319602096332,9.73303415903424,9.73303415903424,9.73309485725765,9.733297184669,9.73366137400944,9.73426835624351,9.73633209583933,9.73801141335358,9.73867909381105,9.73922537782171,9.73993352376145,9.74047980777211,9.74080353163028,9.74122841919413,9.74149144482889,9.74187586691046,9.74246261640339,9.74284703848497,9.74353495168357,9.74428356310559,9.74517380371555,9.74592241513757,9.74687335397094,9.74816824940361,9.74956430854196,9.75057594559873,9.75124362605621,9.7520934011839,9.75330736565203,9.75504738138968,9.75640297504576,9.7576776377373,9.75860834382954,9.75966044636858,9.7607530143899,9.76172418596441,9.76295838317367,9.76356536540774,9.76425327860635,9.76484002809928,9.76556840678015,9.76593259612059,9.76653957835466,9.76698469865964,9.76734888800008,9.76771307734052,9.76807726668096,9.7687854126207,9.76925076566682,9.77003984257111,9.77093008318107,9.77173939282649,9.77279149536554,9.77345917582301,9.77390429612799,9.77447081287978,9.7749361659259,9.7752801225252,9.7755633809011,9.77566454460678,9.77532058800748,9.77536105348975,9.77568477734791,9.77742479308557,9.77811270628418,9.77902317963527,9.78027760958568,9.78151180679495,9.78317089156806,9.78428369233051,9.78565951872773,9.78626650096179,9.78707581060721,9.78778395654696,9.78863373167465,9.78984769614278,9.79069747127047,9.79496657965006,9.79541169995504,9.79460239030962,9.7940763390401,9.79431913193372,9.79484518320325,9.795553],"lat":[53.477427,53.4776063373574,53.4777207290906,53.4779434910545,53.4781843134837,53.4785937084765,53.4790030995191,53.4794606495393,53.4799422758089,53.4803757347771,53.4807128664685,53.4815075234274,53.4823864448491,53.4834700215388,53.4840599572114,53.4845294919854,53.4852518430283,53.4856852477497,53.4860704926735,53.4864918503028,53.4874669761928,53.4880929711219,53.4886346900459,53.488935642014,53.4895014259331,53.4903079559186,53.4909218714477,53.4913311435302,53.4916802254206,53.4922218985275,53.493088561108,53.4934376285327,53.4937987297416,53.4943163027788,53.4944968500265,53.494629250853,53.4947977603978,53.4949783055957,53.4952190313304,53.495628261943,53.496013416559,53.4964226395052,53.4966392853485,53.4968800016538,53.4979872790573,53.4982761292765,53.4984927656496,53.4986371892834,53.4986732951149,53.4987695771822,53.498913999873,53.499070457233,53.4994315104755,53.4997564557649,53.5001175031655,53.5008756926989,53.5012006269197,53.5014894552473,53.5016459031032,53.5017060752018,53.5016338686733,53.5014653862952,53.5009479005188,53.5008516233983,53.5008516233983,53.5009479005188,53.5012487651109,53.5020550716891,53.50253644144,53.503053907826,53.503787977212,53.5041850914175,53.5047266047933,53.505063542958,53.5052801446497,53.5055328452244,53.5056892781586,53.5058938434322,53.5061585734972,53.5066398966522,53.5068805561801,53.5070610499293,53.5072896742416,53.5074942317916,53.5076145593009,53.5077950499245,53.5080116376582,53.5082522893976,53.5086252968934,53.5089742364175,53.5093472375596,53.5101654221152,53.5105504446758,53.5109595273139,53.5114889225134,53.5118017438409,53.5120664370069,53.5122950343197,53.5124995677123,53.512692068827,53.5127401939691,53.5127401939691,53.5127883190565,53.5129086315359,53.5131131619669,53.5133658158423,53.5137628403174,53.5141959536842,53.5145568781083,53.5150140446323,53.5153990231427,53.5161208484223,53.5166381489753,53.5170110826655,53.5172998032681,53.5175043125045,53.5177930297462,53.5179614472285,53.5181659532714,53.5183824880056,53.5186591696666,53.5190080265768,53.5194290569534,53.5198621124042,53.5211131366191,53.5225084662001,53.5230497454753,53.5236752151353,53.5241683673941,53.5253831813936,53.5261770014227,53.526465659563,53.5266821518771,53.5268024249069,53.5268505340232,53.5268745885608,53.5268745885608,53.5269467520918,53.5270670243701,53.5274278391561,53.5279450016556,53.5285824258155,53.529869271772,53.5302901942055,53.5305547718796,53.5309516352914,53.5313605209774,53.5317212991865,53.5319257388073,53.531997893732,53.5319858679198,53.5318776354559,53.5316250919646,53.5313605209774,53.5312402608914,53.5312402608914,53.5313003909771,53.5314687547629,53.5318054803264,53.5321301774412,53.5322143577683,53.5321422032124,53.5319497904625,53.5317092732957,53.5314206508924,53.5311681046758,53.5310839222689,53.5310839222689,53.5311560786279,53.5312643129359,53.5313725469672,53.5314447028345,53.5313965989366,53.5313003909771,53.5311681046758,53.5309275830693,53.5305186932032,53.5301097993891,53.5293401062056,53.5287868805895,53.5283539163708,53.5280532441703,53.5279690555717,53.5279690555717,53.5280652710993,53.5282817552373,53.5285824258156,53.529027414354,53.5295686103295,53.5313965989366,53.5318535837598,53.532298537928,53.5336814737382,53.5341624843029,53.534451088019,53.5345352637323,53.5345232386406,53.5339941312267,53.5335732456081,53.5332245086406,53.5332124831767,53.5332846359088,53.5335131187496,53.5339340049659,53.5348118398978,53.5356054832127,53.5407998721194,53.5417617260551,53.5437334583012,53.5455127475876,53.5483017532506,53.5512829009002,53.555098]}]],[[{"lng":[9.936447,9.92633270965771,9.88910446596836,9.87235175630815,9.85139063649175,9.795553],"lat":[53.541838,53.5414851952869,53.5420142090918,53.5433126694939,53.5459094708117,53.555098]}]],[[{"lng":[9.98846,9.98824489753239,9.98797175552707,9.98743558788697,9.98646441631247,9.98492672798617,9.98402637100564,9.98336880691874,9.98277699924052,9.98223577341515,9.98185135133357,9.97968644803207,9.97669200234402,9.97367732391482,9.96558422746061,9.96153767923351,9.95765299293549,9.9524734112048,9.94349007414063,9.936447],"lat":[53.54469,53.5445329455108,53.5444367673974,53.5443225556039,53.5442684551731,53.5441903099843,53.5441211814278,53.5440099743826,53.5438356492373,53.5436012108411,53.5434930080664,53.5434569404133,53.5436132333545,53.5439618847591,53.5443946894041,53.5444668230813,53.5441782876347,53.543360759855,53.5424470336513,53.541838]}]],[[{"lng":[9.9931,9.99300687482168,9.99283004772496,9.9926839731668,9.9924148884544,9.99197666477992,9.99060817681399,9.98960871931079,9.98897060413567,9.98830173642199,9.98788657715142,9.98758673990046,9.98747141788086,9.98751754668871,9.9878250720743,9.98811722119063,9.98846],"lat":[53.54798,53.5474780765493,53.5468613981424,53.5466695408055,53.5465370673748,53.546463978408,53.5464913867853,53.5465553395968,53.5464913867853,53.5462538469252,53.5458427170939,53.5453630605767,53.5448559892051,53.5446869640646,53.5445910305766,53.5446001671086,53.54469]}]]],null,null,{"interactive":true,"className":"","stroke":true,"color":"#03F","weight":5,"opacity":0.5,"fill":false,"fillColor":"#03F","fillOpacity":0.2,"smoothFactor":1,"noClip":false},null,null,null,{"interactive":false,"permanent":false,"direction":"auto","opacity":1,"offset":[0,0],"textsize":"10px","textOnly":false,"className":"","sticky":true},null]}],"limits":{"lat":[52.091844,53.555098],"lng":[5.11879988899074,9.9931]}},"evals":[],"jsHooks":[]}</script>

# Constructing an itinerary using the node information

The paths object gives both the roads (edges/lines) and towns or other points of interest passed (nodes). We can use the additional data from the list of nodes to get more information on the route along the way.

First, it’s worth taking a closer look at the nodes data.

``` r
nodes_lookup_table %>% glimpse()
```

    ## Rows: 12,567
    ## Columns: 45
    ## $ WKT                     <POINT [°]> POINT (22.47944 58.24702), POINT (22.80…
    ## $ ID                      <dbl> 287, 7743, 6695, 3479, 5911, 7263, 10815, 386…
    ## $ Parent_ID               <dbl> NA, NA, NA, NA, NA, NA, 3865, NA, NA, NA, NA,…
    ## $ Name                    <chr> "Kuressaare", "Tõlluste", "Sakla", "Kahtla", …
    ## $ Alternative_Name        <chr> "Arensburg; Arnsborch; Arnburg; Arensburgk", …
    ## $ Geonames_ID             <dbl> 590939, 588249, 588839, 591760, 589398, 59043…
    ## $ Ready                   <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    ## $ Is_Settlement           <chr> "y", "y", "y", NA, "y", "y", NA, "y", "y", "y…
    ## $ Settlement_From         <dbl> 1384, 1528, 1645, NA, 1250, 1345, NA, 1532, 1…
    ## $ Settlement_To           <dbl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    ## $ Settlement_Description  <chr> "In the second half of the 14th century the c…
    ## $ Is_Town                 <chr> "y", NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, …
    ## $ Town_From               <dbl> 1563, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA,…
    ## $ Town_To                 <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    ## $ Town_Description        <chr> "The settlement near the castle was granted R…
    ## $ Is_Toll                 <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    ## $ Toll_From               <dbl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    ## $ Toll_To                 <dbl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    ## $ Toll_Description        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    ## $ Is_Staple               <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    ## $ Staple_From             <dbl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    ## $ Staple_To               <dbl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    ## $ Staple_Duration_Of_Stay <dbl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    ## $ Staple_Description      <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    ## $ Is_Fair                 <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    ## $ Fair_From               <dbl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    ## $ Fair_To                 <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    ## $ Fair_Description        <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    ## $ Is_Ferry                <chr> NA, NA, NA, NA, NA, "y", "y", NA, NA, NA, NA,…
    ## $ Ferry_From              <dbl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    ## $ Ferry_To                <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    ## $ Ferry_Description       <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    ## $ Is_Bridge               <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    ## $ Bridge_From             <dbl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    ## $ Bridge_To               <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    ## $ Bridge_Description      <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    ## $ Is_Harbour              <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    ## $ Harbour_From            <dbl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    ## $ Harbour_To              <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    ## $ Harbour_Description     <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    ## $ Is_Lock                 <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    ## $ Lock_From               <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    ## $ Lock_To                 <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    ## $ Lock_Description        <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    ## $ node_id                 <int> 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14…

There are a large number of types of node, including town, settlement, bridge, harbour, ferry, and lock. Using the types will help to plan out a proper itinerary for the route.

There’s also dates from/to fields for each type, which means we can draw up an itinerary for a specific time period. From what I can tell, the official Viabundus map takes into account the infrastructure types in the node list (locks, bridges and ferries etc) and draws up a route which would have used the correct network at a specified date, but I haven’t figured out how to do that yet.

The Viabundus data format is quite ‘wide,’ which makes it difficult to easily filter. Each place can be multiple types, with multiple start and end points. To make it easier to work with, I used `pivot_longer` to make the data long, in a couple of steps.

First, I turned all the From/To columns into a pair: one column for the type of node, and a second for the value:

``` r
nodes_to_draw = nodes_to_draw %>% map_df(rev) # The route is in reverse order - switch this around

long_data = nodes_to_draw %>% mutate(order = 1:nrow(.)) %>% 
  pivot_longer(names_to = 'data_type' , values_to = 'value_of', cols =matches("From$|To$", ignore.case = F))

long_data %>% select(Name, data_type, value_of) %>% head(10) %>% kbl()%>%
  kable_styling(bootstrap_options = c("striped", "hover", "condensed"))
```

<table class="table table-striped table-hover table-condensed" style="margin-left: auto; margin-right: auto;">
<thead>
<tr>
<th style="text-align:left;">
Name
</th>
<th style="text-align:left;">
data\_type
</th>
<th style="text-align:right;">
value\_of
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
Hamburg
</td>
<td style="text-align:left;">
Settlement\_From
</td>
<td style="text-align:right;">
831
</td>
</tr>
<tr>
<td style="text-align:left;">
Hamburg
</td>
<td style="text-align:left;">
Settlement\_To
</td>
<td style="text-align:right;">
NA
</td>
</tr>
<tr>
<td style="text-align:left;">
Hamburg
</td>
<td style="text-align:left;">
Town\_From
</td>
<td style="text-align:right;">
1150
</td>
</tr>
<tr>
<td style="text-align:left;">
Hamburg
</td>
<td style="text-align:left;">
Town\_To
</td>
<td style="text-align:right;">
NA
</td>
</tr>
<tr>
<td style="text-align:left;">
Hamburg
</td>
<td style="text-align:left;">
Toll\_From
</td>
<td style="text-align:right;">
1188
</td>
</tr>
<tr>
<td style="text-align:left;">
Hamburg
</td>
<td style="text-align:left;">
Toll\_To
</td>
<td style="text-align:right;">
NA
</td>
</tr>
<tr>
<td style="text-align:left;">
Hamburg
</td>
<td style="text-align:left;">
Staple\_From
</td>
<td style="text-align:right;">
1458
</td>
</tr>
<tr>
<td style="text-align:left;">
Hamburg
</td>
<td style="text-align:left;">
Staple\_To
</td>
<td style="text-align:right;">
1748
</td>
</tr>
<tr>
<td style="text-align:left;">
Hamburg
</td>
<td style="text-align:left;">
Fair\_From
</td>
<td style="text-align:right;">
1000
</td>
</tr>
<tr>
<td style="text-align:left;">
Hamburg
</td>
<td style="text-align:left;">
Fair\_To
</td>
<td style="text-align:right;">
NA
</td>
</tr>
</tbody>
</table>

Next I used `ifelse` to filter the value column in a different way depending on whether it was a to date or a from date:

``` r
long_data  = long_data %>% 
  mutate(date_type = ifelse(str_detect(data_type, "From$"), 'from', 'to')) %>% 
  filter(date_type == 'from' & value_of < 1600 | data_type == 'to' & value_of >1600| is.na(value_of))
```

Using, this, we can easily list each place visited:

``` r
library(htmltools)
long_data %>% distinct(order, .keep_all = T) %>% select(order, Name) %>% filter(!is.na(Name)) %>% distinct(Name) %>% kbl() %>%
  kable_styling(bootstrap_options = c("striped", "hover", "condensed"))%>%
  scroll_box(height = "400px")
```

<div style="border: 1px solid #ddd; padding: 0px; overflow-y: scroll; height:400px; ">

<table class="table table-striped table-hover table-condensed" style="margin-left: auto; margin-right: auto;">
<thead>
<tr>
<th style="text-align:left;position: sticky; top:0; background-color: #FFFFFF;">
Name
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
Hamburg
</td>
</tr>
<tr>
<td style="text-align:left;">
Ottensen
</td>
</tr>
<tr>
<td style="text-align:left;">
Blankenese
</td>
</tr>
<tr>
<td style="text-align:left;">
Buxtehude
</td>
</tr>
<tr>
<td style="text-align:left;">
Kammerbusch
</td>
</tr>
<tr>
<td style="text-align:left;">
Ahrenswohlde
</td>
</tr>
<tr>
<td style="text-align:left;">
Wangersen
</td>
</tr>
<tr>
<td style="text-align:left;">
Steddorf
</td>
</tr>
<tr>
<td style="text-align:left;">
Boitzen
</td>
</tr>
<tr>
<td style="text-align:left;">
Heeslingen
</td>
</tr>
<tr>
<td style="text-align:left;">
Zeven
</td>
</tr>
<tr>
<td style="text-align:left;">
Oldendorf (Zeven)
</td>
</tr>
<tr>
<td style="text-align:left;">
Steinfeld
</td>
</tr>
<tr>
<td style="text-align:left;">
Otterstedt
</td>
</tr>
<tr>
<td style="text-align:left;">
Ottersberg
</td>
</tr>
<tr>
<td style="text-align:left;">
Bassen
</td>
</tr>
<tr>
<td style="text-align:left;">
Borstel (Achim)
</td>
</tr>
<tr>
<td style="text-align:left;">
Achim (Weser)
</td>
</tr>
<tr>
<td style="text-align:left;">
Uphusen (Bremen)
</td>
</tr>
<tr>
<td style="text-align:left;">
Mahndorf (Bremen)
</td>
</tr>
<tr>
<td style="text-align:left;">
Arbergen
</td>
</tr>
<tr>
<td style="text-align:left;">
Hemelingen
</td>
</tr>
<tr>
<td style="text-align:left;">
Hastedt
</td>
</tr>
<tr>
<td style="text-align:left;">
Bremen
</td>
</tr>
<tr>
<td style="text-align:left;">
Neustadt
</td>
</tr>
<tr>
<td style="text-align:left;">
Warturm
</td>
</tr>
<tr>
<td style="text-align:left;">
Kirchhuchting
</td>
</tr>
<tr>
<td style="text-align:left;">
Mittelshuchting
</td>
</tr>
<tr>
<td style="text-align:left;">
Varrel
</td>
</tr>
<tr>
<td style="text-align:left;">
Horstedt
</td>
</tr>
<tr>
<td style="text-align:left;">
Wildeshausen
</td>
</tr>
<tr>
<td style="text-align:left;">
Ahlhorn
</td>
</tr>
<tr>
<td style="text-align:left;">
Lethe
</td>
</tr>
<tr>
<td style="text-align:left;">
Bethen
</td>
</tr>
<tr>
<td style="text-align:left;">
Cloppenburg
</td>
</tr>
<tr>
<td style="text-align:left;">
Krapendorf
</td>
</tr>
<tr>
<td style="text-align:left;">
Lastrup
</td>
</tr>
<tr>
<td style="text-align:left;">
Löningen
</td>
</tr>
<tr>
<td style="text-align:left;">
Haselünne
</td>
</tr>
<tr>
<td style="text-align:left;">
Bawinkel
</td>
</tr>
<tr>
<td style="text-align:left;">
Lingen
</td>
</tr>
<tr>
<td style="text-align:left;">
Schepsdorf
</td>
</tr>
<tr>
<td style="text-align:left;">
Südlohne (Bentheim)
</td>
</tr>
<tr>
<td style="text-align:left;">
Nordhorn
</td>
</tr>
<tr>
<td style="text-align:left;">
Ootmarsum
</td>
</tr>
<tr>
<td style="text-align:left;">
Almelo
</td>
</tr>
<tr>
<td style="text-align:left;">
Wierden
</td>
</tr>
<tr>
<td style="text-align:left;">
Rijssen
</td>
</tr>
<tr>
<td style="text-align:left;">
Heeten
</td>
</tr>
<tr>
<td style="text-align:left;">
Deventer
</td>
</tr>
<tr>
<td style="text-align:left;">
Twello
</td>
</tr>
<tr>
<td style="text-align:left;">
Apeldoorn
</td>
</tr>
<tr>
<td style="text-align:left;">
Hoog-Soeren
</td>
</tr>
<tr>
<td style="text-align:left;">
Garderen
</td>
</tr>
<tr>
<td style="text-align:left;">
Voorthuizen
</td>
</tr>
<tr>
<td style="text-align:left;">
Terschuur
</td>
</tr>
<tr>
<td style="text-align:left;">
Hoevelaken
</td>
</tr>
<tr>
<td style="text-align:left;">
Amersfoort
</td>
</tr>
<tr>
<td style="text-align:left;">
De Bilt
</td>
</tr>
<tr>
<td style="text-align:left;">
Utrecht
</td>
</tr>
</tbody>
</table>

</div>

Another useful task is to make a nice summary of the places visited and their types. To do this easily, the data needs to be made ‘longer’ again. This time, we want a pair of columns where one is the node type, and the other is whether it is a ‘y’ or simply empty.

``` r
longer_data = long_data %>% pivot_longer(names_to = 'node_type', values_to = 'type', cols = matches('^Is_'))

longer_data %>% select(Name, node_type, type) %>% head(10) 
```

    ## # A tibble: 10 x 3
    ##    Name    node_type     type 
    ##    <chr>   <chr>         <chr>
    ##  1 Hamburg Is_Settlement y    
    ##  2 Hamburg Is_Town       y    
    ##  3 Hamburg Is_Toll       y    
    ##  4 Hamburg Is_Staple     y    
    ##  5 Hamburg Is_Fair       y    
    ##  6 Hamburg Is_Ferry      <NA> 
    ##  7 Hamburg Is_Bridge     <NA> 
    ##  8 Hamburg Is_Harbour    y    
    ##  9 Hamburg Is_Lock       <NA> 
    ## 10 Hamburg Is_Settlement y

You’ll notice that Hamburg now has multiple repeated entries for each of the types, because we had already duplicated the data in the previous step, when making the dates long. This will need to be fixed for counts.

First, filter out any entries where the type is NA (this means that particular place was not that type):

``` r
longer_data = longer_data %>% filter(!is.na(type))
```

Now we can add all the node types associated with each place, using `summarise`. We need to use `unique` too, and then it’ll just print one instance of each type for each place:

``` r
longer_data %>% 
  group_by(order) %>% 
  summarise(Name = max(Name),types = paste0(unique(node_type), collapse = '; ')) %>% filter(!is.na(Name)) %>% kbl()%>%
  kable_styling(bootstrap_options = c("striped", "hover", "condensed"))%>%
  scroll_box(height = "400px")
```

<div style="border: 1px solid #ddd; padding: 0px; overflow-y: scroll; height:400px; ">

<table class="table table-striped table-hover table-condensed" style="margin-left: auto; margin-right: auto;">
<thead>
<tr>
<th style="text-align:right;position: sticky; top:0; background-color: #FFFFFF;">
order
</th>
<th style="text-align:left;position: sticky; top:0; background-color: #FFFFFF;">
Name
</th>
<th style="text-align:left;position: sticky; top:0; background-color: #FFFFFF;">
types
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:right;">
1
</td>
<td style="text-align:left;">
Hamburg
</td>
<td style="text-align:left;">
Is\_Settlement; Is\_Town; Is\_Toll; Is\_Staple; Is\_Fair; Is\_Harbour
</td>
</tr>
<tr>
<td style="text-align:right;">
3
</td>
<td style="text-align:left;">
Ottensen
</td>
<td style="text-align:left;">
Is\_Settlement
</td>
</tr>
<tr>
<td style="text-align:right;">
4
</td>
<td style="text-align:left;">
Blankenese
</td>
<td style="text-align:left;">
Is\_Settlement; Is\_Ferry
</td>
</tr>
<tr>
<td style="text-align:right;">
6
</td>
<td style="text-align:left;">
Buxtehude
</td>
<td style="text-align:left;">
Is\_Settlement; Is\_Town; Is\_Toll; Is\_Fair
</td>
</tr>
<tr>
<td style="text-align:right;">
7
</td>
<td style="text-align:left;">
Buxtehude
</td>
<td style="text-align:left;">
Is\_Settlement; Is\_Town; Is\_Toll; Is\_Fair
</td>
</tr>
<tr>
<td style="text-align:right;">
8
</td>
<td style="text-align:left;">
Buxtehude
</td>
<td style="text-align:left;">
Is\_Settlement; Is\_Town; Is\_Toll; Is\_Fair
</td>
</tr>
<tr>
<td style="text-align:right;">
9
</td>
<td style="text-align:left;">
Kammerbusch
</td>
<td style="text-align:left;">
Is\_Settlement
</td>
</tr>
<tr>
<td style="text-align:right;">
10
</td>
<td style="text-align:left;">
Ahrenswohlde
</td>
<td style="text-align:left;">
Is\_Settlement
</td>
</tr>
<tr>
<td style="text-align:right;">
11
</td>
<td style="text-align:left;">
Wangersen
</td>
<td style="text-align:left;">
Is\_Settlement
</td>
</tr>
<tr>
<td style="text-align:right;">
12
</td>
<td style="text-align:left;">
Steddorf
</td>
<td style="text-align:left;">
Is\_Settlement
</td>
</tr>
<tr>
<td style="text-align:right;">
13
</td>
<td style="text-align:left;">
Boitzen
</td>
<td style="text-align:left;">
Is\_Settlement
</td>
</tr>
<tr>
<td style="text-align:right;">
14
</td>
<td style="text-align:left;">
Heeslingen
</td>
<td style="text-align:left;">
Is\_Settlement
</td>
</tr>
<tr>
<td style="text-align:right;">
15
</td>
<td style="text-align:left;">
Zeven
</td>
<td style="text-align:left;">
Is\_Settlement; Is\_Fair
</td>
</tr>
<tr>
<td style="text-align:right;">
16
</td>
<td style="text-align:left;">
Zeven
</td>
<td style="text-align:left;">
Is\_Settlement; Is\_Fair
</td>
</tr>
<tr>
<td style="text-align:right;">
17
</td>
<td style="text-align:left;">
Oldendorf (Zeven)
</td>
<td style="text-align:left;">
Is\_Settlement
</td>
</tr>
<tr>
<td style="text-align:right;">
18
</td>
<td style="text-align:left;">
Steinfeld
</td>
<td style="text-align:left;">
Is\_Settlement
</td>
</tr>
<tr>
<td style="text-align:right;">
19
</td>
<td style="text-align:left;">
Otterstedt
</td>
<td style="text-align:left;">
Is\_Settlement
</td>
</tr>
<tr>
<td style="text-align:right;">
20
</td>
<td style="text-align:left;">
Ottersberg
</td>
<td style="text-align:left;">
Is\_Settlement; Is\_Toll; Is\_Fair
</td>
</tr>
<tr>
<td style="text-align:right;">
21
</td>
<td style="text-align:left;">
Bassen
</td>
<td style="text-align:left;">
Is\_Settlement
</td>
</tr>
<tr>
<td style="text-align:right;">
22
</td>
<td style="text-align:left;">
Borstel (Achim)
</td>
<td style="text-align:left;">
Is\_Settlement
</td>
</tr>
<tr>
<td style="text-align:right;">
23
</td>
<td style="text-align:left;">
Achim (Weser)
</td>
<td style="text-align:left;">
Is\_Settlement
</td>
</tr>
<tr>
<td style="text-align:right;">
24
</td>
<td style="text-align:left;">
Uphusen (Bremen)
</td>
<td style="text-align:left;">
Is\_Settlement
</td>
</tr>
<tr>
<td style="text-align:right;">
25
</td>
<td style="text-align:left;">
Mahndorf (Bremen)
</td>
<td style="text-align:left;">
Is\_Settlement
</td>
</tr>
<tr>
<td style="text-align:right;">
26
</td>
<td style="text-align:left;">
Arbergen
</td>
<td style="text-align:left;">
Is\_Settlement
</td>
</tr>
<tr>
<td style="text-align:right;">
27
</td>
<td style="text-align:left;">
Hemelingen
</td>
<td style="text-align:left;">
Is\_Settlement
</td>
</tr>
<tr>
<td style="text-align:right;">
28
</td>
<td style="text-align:left;">
Hastedt
</td>
<td style="text-align:left;">
Is\_Settlement
</td>
</tr>
<tr>
<td style="text-align:right;">
29
</td>
<td style="text-align:left;">
Bremen
</td>
<td style="text-align:left;">
Is\_Settlement; Is\_Town; Is\_Toll; Is\_Staple; Is\_Fair
</td>
</tr>
<tr>
<td style="text-align:right;">
30
</td>
<td style="text-align:left;">
Bremen
</td>
<td style="text-align:left;">
Is\_Settlement; Is\_Town; Is\_Toll; Is\_Staple; Is\_Fair
</td>
</tr>
<tr>
<td style="text-align:right;">
33
</td>
<td style="text-align:left;">
Warturm
</td>
<td style="text-align:left;">
Is\_Settlement; Is\_Toll; Is\_Bridge
</td>
</tr>
<tr>
<td style="text-align:right;">
34
</td>
<td style="text-align:left;">
Kirchhuchting
</td>
<td style="text-align:left;">
Is\_Settlement
</td>
</tr>
<tr>
<td style="text-align:right;">
35
</td>
<td style="text-align:left;">
Mittelshuchting
</td>
<td style="text-align:left;">
Is\_Settlement
</td>
</tr>
<tr>
<td style="text-align:right;">
36
</td>
<td style="text-align:left;">
Varrel
</td>
<td style="text-align:left;">
Is\_Settlement; Is\_Toll
</td>
</tr>
<tr>
<td style="text-align:right;">
37
</td>
<td style="text-align:left;">
Horstedt
</td>
<td style="text-align:left;">
Is\_Settlement
</td>
</tr>
<tr>
<td style="text-align:right;">
38
</td>
<td style="text-align:left;">
Wildeshausen
</td>
<td style="text-align:left;">
Is\_Settlement; Is\_Town; Is\_Toll; Is\_Fair
</td>
</tr>
<tr>
<td style="text-align:right;">
39
</td>
<td style="text-align:left;">
Wildeshausen
</td>
<td style="text-align:left;">
Is\_Settlement; Is\_Town; Is\_Toll; Is\_Fair
</td>
</tr>
<tr>
<td style="text-align:right;">
40
</td>
<td style="text-align:left;">
Wildeshausen
</td>
<td style="text-align:left;">
Is\_Settlement; Is\_Town; Is\_Toll; Is\_Fair
</td>
</tr>
<tr>
<td style="text-align:right;">
41
</td>
<td style="text-align:left;">
Ahlhorn
</td>
<td style="text-align:left;">
Is\_Settlement
</td>
</tr>
<tr>
<td style="text-align:right;">
42
</td>
<td style="text-align:left;">
Ahlhorn
</td>
<td style="text-align:left;">
Is\_Settlement
</td>
</tr>
<tr>
<td style="text-align:right;">
43
</td>
<td style="text-align:left;">
Lethe
</td>
<td style="text-align:left;">
Is\_Settlement
</td>
</tr>
<tr>
<td style="text-align:right;">
44
</td>
<td style="text-align:left;">
Bethen
</td>
<td style="text-align:left;">
Is\_Settlement
</td>
</tr>
<tr>
<td style="text-align:right;">
45
</td>
<td style="text-align:left;">
Cloppenburg
</td>
<td style="text-align:left;">
Is\_Settlement; Is\_Town; Is\_Toll; Is\_Fair
</td>
</tr>
<tr>
<td style="text-align:right;">
46
</td>
<td style="text-align:left;">
Cloppenburg
</td>
<td style="text-align:left;">
Is\_Settlement; Is\_Town; Is\_Toll; Is\_Fair
</td>
</tr>
<tr>
<td style="text-align:right;">
47
</td>
<td style="text-align:left;">
Cloppenburg
</td>
<td style="text-align:left;">
Is\_Settlement; Is\_Town; Is\_Toll; Is\_Fair
</td>
</tr>
<tr>
<td style="text-align:right;">
48
</td>
<td style="text-align:left;">
Krapendorf
</td>
<td style="text-align:left;">
Is\_Settlement
</td>
</tr>
<tr>
<td style="text-align:right;">
49
</td>
<td style="text-align:left;">
Krapendorf
</td>
<td style="text-align:left;">
Is\_Settlement
</td>
</tr>
<tr>
<td style="text-align:right;">
50
</td>
<td style="text-align:left;">
Lastrup
</td>
<td style="text-align:left;">
Is\_Settlement
</td>
</tr>
<tr>
<td style="text-align:right;">
51
</td>
<td style="text-align:left;">
Löningen
</td>
<td style="text-align:left;">
Is\_Settlement
</td>
</tr>
<tr>
<td style="text-align:right;">
52
</td>
<td style="text-align:left;">
Haselünne
</td>
<td style="text-align:left;">
Is\_Settlement; Is\_Town; Is\_Toll; Is\_Fair
</td>
</tr>
<tr>
<td style="text-align:right;">
55
</td>
<td style="text-align:left;">
Bawinkel
</td>
<td style="text-align:left;">
Is\_Settlement
</td>
</tr>
<tr>
<td style="text-align:right;">
56
</td>
<td style="text-align:left;">
Lingen
</td>
<td style="text-align:left;">
Is\_Settlement; Is\_Town; Is\_Fair
</td>
</tr>
<tr>
<td style="text-align:right;">
57
</td>
<td style="text-align:left;">
Lingen
</td>
<td style="text-align:left;">
Is\_Settlement; Is\_Town; Is\_Fair
</td>
</tr>
<tr>
<td style="text-align:right;">
60
</td>
<td style="text-align:left;">
Schepsdorf
</td>
<td style="text-align:left;">
Is\_Settlement
</td>
</tr>
<tr>
<td style="text-align:right;">
61
</td>
<td style="text-align:left;">
Schepsdorf
</td>
<td style="text-align:left;">
Is\_Settlement
</td>
</tr>
<tr>
<td style="text-align:right;">
62
</td>
<td style="text-align:left;">
Südlohne (Bentheim)
</td>
<td style="text-align:left;">
Is\_Settlement
</td>
</tr>
<tr>
<td style="text-align:right;">
63
</td>
<td style="text-align:left;">
Nordhorn
</td>
<td style="text-align:left;">
Is\_Settlement; Is\_Town; Is\_Toll
</td>
</tr>
<tr>
<td style="text-align:right;">
64
</td>
<td style="text-align:left;">
Nordhorn
</td>
<td style="text-align:left;">
Is\_Settlement; Is\_Town; Is\_Toll
</td>
</tr>
<tr>
<td style="text-align:right;">
69
</td>
<td style="text-align:left;">
Ootmarsum
</td>
<td style="text-align:left;">
Is\_Settlement; Is\_Town
</td>
</tr>
<tr>
<td style="text-align:right;">
70
</td>
<td style="text-align:left;">
Ootmarsum
</td>
<td style="text-align:left;">
Is\_Settlement; Is\_Town
</td>
</tr>
<tr>
<td style="text-align:right;">
71
</td>
<td style="text-align:left;">
Ootmarsum
</td>
<td style="text-align:left;">
Is\_Settlement; Is\_Town
</td>
</tr>
<tr>
<td style="text-align:right;">
72
</td>
<td style="text-align:left;">
Almelo
</td>
<td style="text-align:left;">
Is\_Settlement; Is\_Town
</td>
</tr>
<tr>
<td style="text-align:right;">
73
</td>
<td style="text-align:left;">
Almelo
</td>
<td style="text-align:left;">
Is\_Settlement; Is\_Town
</td>
</tr>
<tr>
<td style="text-align:right;">
74
</td>
<td style="text-align:left;">
Wierden
</td>
<td style="text-align:left;">
Is\_Settlement
</td>
</tr>
<tr>
<td style="text-align:right;">
75
</td>
<td style="text-align:left;">
Rijssen
</td>
<td style="text-align:left;">
Is\_Settlement; Is\_Town
</td>
</tr>
<tr>
<td style="text-align:right;">
76
</td>
<td style="text-align:left;">
Rijssen
</td>
<td style="text-align:left;">
Is\_Settlement; Is\_Town
</td>
</tr>
<tr>
<td style="text-align:right;">
77
</td>
<td style="text-align:left;">
Rijssen
</td>
<td style="text-align:left;">
Is\_Settlement; Is\_Town
</td>
</tr>
<tr>
<td style="text-align:right;">
78
</td>
<td style="text-align:left;">
Heeten
</td>
<td style="text-align:left;">
Is\_Settlement
</td>
</tr>
<tr>
<td style="text-align:right;">
79
</td>
<td style="text-align:left;">
Heeten
</td>
<td style="text-align:left;">
Is\_Settlement
</td>
</tr>
<tr>
<td style="text-align:right;">
80
</td>
<td style="text-align:left;">
Deventer
</td>
<td style="text-align:left;">
Is\_Settlement; Is\_Town
</td>
</tr>
<tr>
<td style="text-align:right;">
81
</td>
<td style="text-align:left;">
Deventer
</td>
<td style="text-align:left;">
Is\_Settlement; Is\_Town
</td>
</tr>
<tr>
<td style="text-align:right;">
83
</td>
<td style="text-align:left;">
Twello
</td>
<td style="text-align:left;">
Is\_Settlement
</td>
</tr>
<tr>
<td style="text-align:right;">
84
</td>
<td style="text-align:left;">
Twello
</td>
<td style="text-align:left;">
Is\_Settlement
</td>
</tr>
<tr>
<td style="text-align:right;">
85
</td>
<td style="text-align:left;">
Apeldoorn
</td>
<td style="text-align:left;">
Is\_Settlement; Is\_Toll
</td>
</tr>
<tr>
<td style="text-align:right;">
86
</td>
<td style="text-align:left;">
Hoog-Soeren
</td>
<td style="text-align:left;">
Is\_Settlement
</td>
</tr>
<tr>
<td style="text-align:right;">
87
</td>
<td style="text-align:left;">
Garderen
</td>
<td style="text-align:left;">
Is\_Settlement
</td>
</tr>
<tr>
<td style="text-align:right;">
88
</td>
<td style="text-align:left;">
Voorthuizen
</td>
<td style="text-align:left;">
Is\_Settlement
</td>
</tr>
<tr>
<td style="text-align:right;">
89
</td>
<td style="text-align:left;">
Voorthuizen
</td>
<td style="text-align:left;">
Is\_Settlement
</td>
</tr>
<tr>
<td style="text-align:right;">
90
</td>
<td style="text-align:left;">
Terschuur
</td>
<td style="text-align:left;">
Is\_Settlement
</td>
</tr>
<tr>
<td style="text-align:right;">
91
</td>
<td style="text-align:left;">
Hoevelaken
</td>
<td style="text-align:left;">
Is\_Settlement
</td>
</tr>
<tr>
<td style="text-align:right;">
92
</td>
<td style="text-align:left;">
Hoevelaken
</td>
<td style="text-align:left;">
Is\_Settlement
</td>
</tr>
<tr>
<td style="text-align:right;">
93
</td>
<td style="text-align:left;">
Amersfoort
</td>
<td style="text-align:left;">
Is\_Settlement; Is\_Town
</td>
</tr>
<tr>
<td style="text-align:right;">
94
</td>
<td style="text-align:left;">
Amersfoort
</td>
<td style="text-align:left;">
Is\_Settlement; Is\_Town
</td>
</tr>
<tr>
<td style="text-align:right;">
95
</td>
<td style="text-align:left;">
Amersfoort
</td>
<td style="text-align:left;">
Is\_Settlement; Is\_Town
</td>
</tr>
<tr>
<td style="text-align:right;">
97
</td>
<td style="text-align:left;">
De Bilt
</td>
<td style="text-align:left;">
Is\_Settlement
</td>
</tr>
<tr>
<td style="text-align:right;">
98
</td>
<td style="text-align:left;">
De Bilt
</td>
<td style="text-align:left;">
Is\_Settlement
</td>
</tr>
<tr>
<td style="text-align:right;">
99
</td>
<td style="text-align:left;">
Utrecht
</td>
<td style="text-align:left;">
Is\_Settlement; Is\_Town; Is\_Harbour
</td>
</tr>
</tbody>
</table>

</div>

Tidy data also allows us to count the totals for each type of node passed. Again, we want to remove the duplicated data which is easy to do with `distinct`

``` r
longer_data %>% distinct(order, node_type) %>% group_by(node_type) %>% tally() %>% kbl()%>%
  kable_styling(bootstrap_options = c("striped", "hover", "condensed"))
```

<table class="table table-striped table-hover table-condensed" style="margin-left: auto; margin-right: auto;">
<thead>
<tr>
<th style="text-align:left;">
node\_type
</th>
<th style="text-align:right;">
n
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
Is\_Bridge
</td>
<td style="text-align:right;">
4
</td>
</tr>
<tr>
<td style="text-align:left;">
Is\_Fair
</td>
<td style="text-align:right;">
18
</td>
</tr>
<tr>
<td style="text-align:left;">
Is\_Ferry
</td>
<td style="text-align:right;">
3
</td>
</tr>
<tr>
<td style="text-align:left;">
Is\_Harbour
</td>
<td style="text-align:right;">
10
</td>
</tr>
<tr>
<td style="text-align:left;">
Is\_Settlement
</td>
<td style="text-align:right;">
85
</td>
</tr>
<tr>
<td style="text-align:left;">
Is\_Staple
</td>
<td style="text-align:right;">
3
</td>
</tr>
<tr>
<td style="text-align:left;">
Is\_Toll
</td>
<td style="text-align:right;">
20
</td>
</tr>
<tr>
<td style="text-align:left;">
Is\_Town
</td>
<td style="text-align:right;">
31
</td>
</tr>
</tbody>
</table>

# Estimate Travel Times

We can use this info on the nodes passed plus the road lengths to estimate the total travel time.

First, get the total distance (note that everything is in metres for now):

``` r
distance = line_to_draw %>% tally(Length) %>% pull(n)
```

Viabundus suggests 5km per hour for wagons, up to a maximum of 35km per day.

``` r
time_taken_in_hours = distance / 5000

time_taken_in_days = time_taken_in_hours/7
```

Passing through certain nodes slow you down, according to the Viabundus documentation. [Staples](https://en.wikipedia.org/wiki/Staple_right) forced merchants to stop for three days, and ferries add an hour for loading/unloading. We can use the lists generated above to add the correct time penalties:

``` r
staples = longer_data %>% 
  distinct(order, node_type) %>% 
  filter(node_type == 'Is_Staple') %>% 
  tally() %>% pull(n)

ferries = longer_data %>% 
  distinct(order, node_type) %>% 
  filter(node_type == 'Is_Ferry') %>% 
  tally() %>% pull(n)

ferries = ferries /24

total_time_taken = time_taken_in_days + staples+ ferries

paste0("Total time taken (in days): ", total_time_taken)
```

    ## [1] "Total time taken (in days): 15.3426"

# Calculating all daily travel rates

`sfnetworks` also has some functions for calculating the ‘neighbourhood’ of a node - the number of nodes reachable within a specified distance.

Using the code from the package vignette, I visualised the maximum one could travel from Utrecht in steps of a day for up to 10 days, ignoring staple stops:

Download a map of the world to use as a backdrop:

``` r
library(rnaturalearth)
library(rnaturalearthdata)

map = rnaturalearth::ne_coastline(returnclass = 'sf', scale = 'medium')
map = map %>% st_set_crs(4326)
```

Get a set of coordinates for Utrecht, in sf form:

``` r
utrecht_point = sf_net %>% activate(nodes) %>% filter(Name == 'Utrecht') %>%
  st_geometry() %>%
  st_combine() %>% 
  st_centroid()
```

Make a list of the ‘thresholds’: a sequence of distances which will be fed to a for loop to calculate ten separate ‘neighborhood’ objects. The thresholds are each 35,000 metres long, corresponding to the maximum speed per day:

``` r
thresholds = rev(seq(0, 350000, 35000))
palette = sf.colors(n = 10)
```

Run the for loop, which will generate 10 separate network objects calculating the neighborhood of nodes from our starting point, and put them in a single list, with an additional ‘days’ column.

``` r
nbh = list()


for (i in c(1:10)) {
  nbh[[i]] = convert(sf_net, to_spatial_neighborhood, utrecht_point, thresholds[i])
   nbh[[i]] =  nbh[[i]] %>% st_set_crs(4326)
   nbh[[i]] = nbh[[i]] %>% activate(edges) %>% as_tibble() %>% mutate(days = 10-(i-1))
   
}
```

Use `rbindlist()` from `data.table` to merge them into one large spatial object:

``` r
library(data.table)
```

    ## 
    ## Attaching package: 'data.table'

    ## The following objects are masked from 'package:dplyr':
    ## 
    ##     between, first, last

    ## The following object is masked from 'package:purrr':
    ## 
    ##     transpose

``` r
all_neighborhoods = rbindlist(nbh)
all_neighborhoods = st_as_sf(all_neighborhoods)
all_neighborhoods = all_neighborhoods %>% st_set_crs(4326)
```

Plot the results:

``` r
p = ggplot() + 
  geom_sf(data = map) + 
  geom_sf(data = all_neighborhoods, aes(color = days)) + 
  coord_sf(xlim = c(0, 11), ylim = c(50, 54))+ theme_void() + 
  labs(title  = "Maximum Distance Reached From Utrecht,\nby Cart or Wagon", color = 'Days:', caption = 'Made with data from Viabundus.eu') + 
  theme(title = element_text(size = 14, face = 'bold')) + 
  scale_color_viridis_c(direction = -1, option = 'A', breaks = 1:10) + 
  guides(color = guide_colorsteps(barwidth = '20', barheight = .5)) + 
  theme(legend.position = 'bottom') 

p
```

<img src="{{< blogdown/postref >}}index.en_files/figure-html/unnamed-chunk-34-1.png" width="672" />

# Calculating elevation data

A few more R packages allow us to estimate the elevation travelled during the trip, which can also come in handy for understanding likely routes and travel times. Load the `elevatr` and `slopes` packages.

``` r
library(elevatr)
library(slopes)
```

Take the spatial lines data created above, and reverse it, because the path is reversed by default:

``` r
line_to_draw = line_to_draw %>% map_df(rev)

line_to_draw = line_to_draw %>% st_as_sf()
line_to_draw = line_to_draw %>% st_set_crs(4326)
```

The `get_elev_raster` function takes a different spatial object, called a SpatialLinesDataFrame. Create this using `as('Spatial')`:

``` r
sp_edges = line_to_draw  %>% as('Spatial')
```

Now, download elevation data using `get_elev_raster`. This function takes the spatial object and downloads the relevant elevation data as a raster file from an online service. We can specify the zoom level (higher numbers mean a larger download) and the src, which is the data source, in this case Mapzen tiles hosted on Amazon Web Services.

``` r
elevation = get_elev_raster(sp_edges, z=9, src = "aws")
```

    ## Mosaicing & Projecting

    ## Note: Elevation units are in meters.
    ## Note: The coordinate reference system is:
    ##  GEOGCRS["WGS 84 (with axis order normalized for visualization)",
    ##     DATUM["World Geodetic System 1984",
    ##         ELLIPSOID["WGS 84",6378137,298.257223563,
    ##             LENGTHUNIT["metre",1]]],
    ##     PRIMEM["Greenwich",0,
    ##         ANGLEUNIT["degree",0.0174532925199433]],
    ##     CS[ellipsoidal,2],
    ##         AXIS["geodetic longitude (Lon)",east,
    ##             ORDER[1],
    ##             ANGLEUNIT["degree",0.0174532925199433,
    ##                 ID["EPSG",9122]]],
    ##         AXIS["geodetic latitude (Lat)",north,
    ##             ORDER[2],
    ##             ANGLEUNIT["degree",0.0174532925199433,
    ##                 ID["EPSG",9122]]]]

With this info, create a new ‘slope’ column using `slope_raster` and the raster we’ve just downloaded.

``` r
line_to_draw$slope = slope_raster(line_to_draw, e = elevation)
```

    ## Loading required namespace: geodist

Use the function `slope_3d` to estimate elevation data using the slopes, and add it as a Z axis to the lines sf object.

``` r
line_to_draw_3d = slope_3d(line_to_draw, elevation)
```

Plot the result using `plot_slope`:

``` r
plot_slope(line_to_draw_3d, title = "Elevation Data, Utrecht to Hamburg")
```

<img src="{{< blogdown/postref >}}index.en_files/figure-html/unnamed-chunk-41-1.png" width="672" />

# Using Slope Data in the Shortest-Path Routing

We can also supply elevation data as part of the weight used by the shortest-path algorithm - letting us help an early modern trader avoid pesky hills. To do this, first download elevation data for the entire Viabundus dataset. I’ve reduced the zoom level a bit because it takes a long time to download and process.

``` r
all_elevation = get_elev_raster(edges_sf, z=7, src = "aws")
```

    ## Mosaicing & Projecting

    ## Note: Elevation units are in meters.
    ## Note: The coordinate reference system is:
    ##  GEOGCRS["WGS 84 (with axis order normalized for visualization)",
    ##     DATUM["World Geodetic System 1984",
    ##         ELLIPSOID["WGS 84",6378137,298.257223563,
    ##             LENGTHUNIT["metre",1]]],
    ##     PRIMEM["Greenwich",0,
    ##         ANGLEUNIT["degree",0.0174532925199433]],
    ##     CS[ellipsoidal,2],
    ##         AXIS["geodetic longitude (Lon)",east,
    ##             ORDER[1],
    ##             ANGLEUNIT["degree",0.0174532925199433,
    ##                 ID["EPSG",9122]]],
    ##         AXIS["geodetic latitude (Lat)",north,
    ##             ORDER[2],
    ##             ANGLEUNIT["degree",0.0174532925199433,
    ##                 ID["EPSG",9122]]]]

Use the full raster to get slope data for all the roads:

``` r
edges_sf$slope = slope_raster(edges_sf, e = all_elevation)
```

Create a new weight column in the roads dataset, taking the slope of each road into account using some kind of appropriate formula. I’ve done a very exaggerated one here, so we can easily see changes in the routing:

``` r
edges_sf  = edges_sf %>% mutate(weight =  Length + Length * (slope*1000))
```

Now, create an sfnetworks object and run the pathing algorithm as above:

``` r
sf_net_elev = as_sfnetwork(edges_sf, directed = F)

sf_net_elev = sf_net_elev %>% 
  st_join(nodes_sf, join = st_nearest_feature)
```

    ## although coordinates are longitude/latitude, st_nearest_points assumes that they are planar

``` r
paths = st_network_paths(sf_net_elev, from =utrecht_node_id, to = hamburg_node_id)

 node_list = paths %>%
  slice(1) %>%
  pull(node_paths) %>%
  unlist() %>% as_tibble()

edge_list = paths %>%
  slice(1) %>%
  pull(edge_paths) %>%
  unlist() %>% as_tibble()

line_to_draw = edge_list %>% inner_join(sf_net %>% 
  activate(edges) %>% 
  as_tibble() %>% 
  mutate(edge_id = 1:nrow(.)) , by = c('value' = 'edge_id'))

line_to_draw = line_to_draw %>% st_as_sf()
line_to_draw = line_to_draw %>% st_set_crs(4326)

nodes_to_draw = node_list %>% inner_join(sf_net %>% 
  activate(nodes) %>% 
  as_tibble() %>% 
  mutate(node_id = 1:nrow(.)) , by = c('value' = 'node_id'))

nodes_to_draw = nodes_to_draw %>% st_as_sf()
nodes_to_draw = nodes_to_draw %>% st_set_crs(4326)
```

And plot it - it now takes a totally different route, avoiding the hills around Amerongen in the Netherlands, for example.

``` r
library(leaflet)

leaflet() %>% 
  addTiles() %>% 
  addCircles(data = nodes_to_draw, label = ~Name) %>% 
  addPolylines(data  = line_to_draw)
```

<div id="htmlwidget-2" style="width:672px;height:480px;" class="leaflet html-widget"></div>
<script type="application/json" data-for="htmlwidget-2">{"x":{"options":{"crs":{"crsClass":"L.CRS.EPSG3857","code":null,"proj4def":null,"projectedBounds":null,"options":{}}},"calls":[{"method":"addTiles","args":["//{s}.tile.openstreetmap.org/{z}/{x}/{y}.png",null,null,{"minZoom":0,"maxZoom":18,"tileSize":256,"subdomains":"abc","errorTileUrl":"","tms":false,"noWrap":false,"zoomOffset":0,"zoomReverse":false,"opacity":1,"zIndex":1,"detectRetina":false,"attribution":"&copy; <a href=\"http://openstreetmap.org\">OpenStreetMap<\/a> contributors, <a href=\"http://creativecommons.org/licenses/by-sa/2.0/\">CC-BY-SA<\/a>"}]},{"method":"addCircles","args":[[52.091844,52.093432,52.092846,52.092856,52.126164,52.138258,52.171829,52.209477,52.229807,52.255641,52.276948,52.284801,52.30635,52.329933,52.373362,52.373074,52.377087,52.377573,52.55894,52.559609,52.560243,52.567599,52.583394,52.59888,52.622679,52.623892,52.634715,52.640124,52.702662,52.697755,52.764274,52.770038,52.817696,52.859378,52.919006,52.987559,52.991676,53.074117,53.094072,53.096955,53.163259,53.164275,53.162237,53.17594,53.143014,53.142255,53.128703,53.138163,53.159927,53.182414,53.173999,53.160188,53.165115,53.216324,53.215579,53.226324,53.216849,53.208149,53.209537,53.206121,53.215949,53.217324,53.218204,53.195142,53.197398,53.187145,53.140346,53.138916,53.140241,53.235771,53.16807,53.075981,53.07518,53.075689,53.075887,53.074656,53.068596,53.051802,53.038871,53.038019,53.028939,53.01297,53.029817,53.06366,53.110276,53.140233,53.218914,53.273983,53.296491,53.295483,53.314912,53.336027,53.346258,53.365284,53.378304,53.414627,53.468829,53.470173,53.476376,53.477427,53.555098,53.541838,53.54469,53.547316,53.54798],[5.118906,5.116996,5.11162,5.109726,5.066822,5.042248,5.001748,5.022115,5.033258,5.043433,5.048368,5.062793,5.045592,5.069393,4.893505,4.89257,4.896699,4.897774,5.917383,5.918985,5.920472,5.923852,5.933873,6.015805,6.040893,6.040333,6.04485,6.070683,6.182969,6.190474,6.360496,6.357712,6.464505,6.516947,6.542643,6.645967,6.639621,6.668757,6.68729,6.688743,6.743031,6.795179,6.862903,6.971194,7.038785,7.039675,7.050126,7.140995,7.190474,7.275917,7.297466,7.30937,7.354946,7.42508,7.428082,7.451264,7.643661,7.664067,7.672318,7.68488,7.74356,7.797488,7.805967,7.896441,7.911292,8.004598,8.212924,8.214479,8.217053,8.464983,8.62331,8.801675,8.8031,8.806253,8.807392,8.809272,8.860087,8.886329,8.918209,8.941825,8.970547,9.032515,9.054039,9.085822,9.143259,9.146686,9.203015,9.257674,9.274875,9.278606,9.334932,9.373245,9.398899,9.426945,9.465251,9.549767,9.68523,9.689763,9.700347,9.702158,9.795553,9.936447,9.98846,9.993701,9.9931],10,null,null,{"interactive":true,"className":"","stroke":true,"color":"#03F","weight":5,"opacity":0.5,"fill":true,"fillColor":"#03F","fillOpacity":0.2},null,null,["Utrecht","Utrecht",null,null,"Oud-Zuilen","Maarssen","Breukelen","Loenen","Vreeland","Nederhorst den Berg","Hinderdam",null,null,"Muiden",null,"Amsterdam",null,null,null,"IJsselbrug","IJsselbrug","IJsselmuiden","Grafhorst","Afsched","Genemuiden",null,null,"Zwartsluis",null,"Meppel","Ruinen","Ruinen","Spier","Beilen","Hooghalen","Rolde","Rolde","Schipborg","Zuidlaren","Zuidlaren","Martenshoek","Sappemeer",null,null,null,"Winschoten","Winschoterhogebrug","Oudeschans","Booneschans","Bunde","Bunde","Stapelmoor","Weener",null,"Leerort","Leer","Stickhausen",null,"Detern","Detern","Vreschen-Bokel",null,"Apen","Howiek","Howiek","Bad Zwischenahn","Oldenburg (Oldenburg)","Oldenburg (Oldenburg)","Stau","Elsfleth","Vegesack","Schlachte","Schlachte","Stintbrücke","Bremen","Bremen","Hastedt","Hemelingen","Arbergen","Mahndorf (Bremen)","Uphusen (Bremen)","Achim (Weser)","Borstel (Achim)","Bassen","Ottersberg","Otterstedt","Steinfeld","Oldendorf (Zeven)","Zeven","Zeven","Heeslingen","Boitzen","Steddorf","Wangersen","Ahrenswohlde","Kammerbusch","Buxtehude","Buxtehude","Buxtehude",null,"Blankenese","Ottensen",null,"Hamburg","Hamburg"],{"interactive":false,"permanent":false,"direction":"auto","opacity":1,"offset":[0,0],"textsize":"10px","textOnly":false,"className":"","sticky":true},null,null]},{"method":"addPolylines","args":[[[[{"lng":[5.116996,5.11712109983706,5.11715008351816,5.11715732943843,5.11709936207623,5.11704864063431,5.11704139471403,5.11707762431541,5.11715732943843,5.11725877232228,5.1173964448075,5.11763556017657,5.11786018370509,5.11808480723361,5.11830943076214,5.11852680837038,5.118906],"lat":[52.093432,52.0930359705775,52.0928712552828,52.0927154429578,52.0925329192564,52.0923503948083,52.0922034843443,52.0920921882162,52.0920120548321,52.0919230175698,52.091865143254,52.091811720742,52.0917805575805,52.0917582981661,52.0917627500499,52.0917850094621,52.091844]}]],[[{"lng":[5.116996,5.11652331141438,5.11472424471323,5.1144631996865,5.1138554,5.113654,5.11304439913333,5.11251031839938,5.11238774194287,5.11211791673303,5.11162],"lat":[52.093432,52.0933631734808,52.093097446012,52.0930586116005,52.0930228,52.0930086,52.0929773864443,52.0928855779068,52.0928702805667,52.0928308364641,52.092846]}]],[[{"lng":[5.11162,5.109726],"lat":[52.092846,52.092856]}]],[[{"lng":[5.109726,5.10870595024178,5.10791625335814,5.10783312737039,5.10841500928465,5.10766687539489,5.10539476506302,5.10418943824062,5.10256848147947,5.10211128854684,5.1008228357367,5.1001301191721,5.09938198528234,5.09843989075449,5.09805196947832,5.09716529227564,5.09659726469267,5.09614007176004,5.09546120952674,5.09328607951391,5.09154043377113,5.09082000854395,5.08890811082567,5.08724559107065,5.08541681934012,5.08411451219868,5.08076176402604,5.07772766547312,5.07778308279828,5.07765839381666,5.0769005,5.0764459,5.076159,5.0759678,5.07541322087754,5.0751429,5.0730842,5.073037,5.0728111,5.07086,5.0702633,5.07014,5.0692,5.06906,5.06873,5.06819,5.06732256118158,5.066822],"lat":[52.092856,52.0931490788279,52.0937959722899,52.0947365046035,52.0956557387451,52.0961323712133,52.0972090311692,52.0978133111425,52.0980345947188,52.0981707686817,52.0989622716104,52.0992686560703,52.0994218475111,52.0995495066431,52.0996856759813,52.1003154537629,52.1007579949924,52.1019749607336,52.103089774333,52.1053959033299,52.1069531111778,52.1077444582615,52.1096334235587,52.1115563420688,52.1141087606563,52.1151637176569,52.1168141648178,52.1184985788086,52.1189239258108,52.1194003096346,52.1198344,52.1201416,52.1203241,52.1204457,52.1207911276777,52.1209595,52.122239,52.1222668,52.1224049,52.12357,52.1239181,52.12399,52.12408,52.12412,52.12434,52.12479,52.1256215150991,52.126164]}]],[[{"lng":[5.066822,5.0655316,5.0653828,5.0650027,5.0646818,5.06425,5.06259282421401,5.0618182745434,5.0617011,5.0615504,5.0614027,5.06047902301472,5.05716783783595,5.05471562119729,5.05306695577355,5.05175166009379,5.0514723,5.0509282,5.0506805,5.0501154,5.0490087,5.0487501,5.04868726911451,5.04858422570479,5.04850430944426,5.0484613,5.0483639,5.0483038,5.04827636056377,5.04793526820469,5.0477717,5.04759878704952,5.04734,5.0472284,5.0471201,5.0471023,5.04690976876108,5.04653877905838,5.04634003285798,5.046323,5.046302,5.0462038,5.0460095,5.0459481,5.04593685800574,5.0459267,5.0459238,5.04592,5.04591052355388,5.04587,5.0457452,5.04573378943858,5.0455865,5.0452789,5.0451835,5.0451435,5.0449983,5.0449772,5.0446361,5.04439564443082,5.0442848,5.044172,5.0440603,5.043915,5.04385,5.04375530898955,5.0436189,5.0430765,5.04230755903458,5.042248],"lat":[52.126164,52.126972,52.1270523,52.1271877,52.1272352,52.127267,52.1274524551672,52.1275805522797,52.1276202,52.12768,52.1277052,52.12741300295,52.1274555293257,52.1276001186995,52.128033884006,52.1286030632351,52.1287654,52.1291279,52.1293076,52.1297175,52.1305203,52.1307079,52.1307501064146,52.130819325357,52.1308730086678,52.1309019,52.1309766,52.1310293,52.1310518921313,52.1313327279713,52.1314674,52.1316372693341,52.1318915,52.1320011,52.1321053,52.1321219,52.1323018460666,52.132719315714,52.1330474859292,52.1330858,52.1331331,52.1333641,52.1338125,52.1339614,52.1341006118077,52.1342264,52.1344814,52.13485,52.1349049635325,52.13514,52.1355069,52.1355328057386,52.1358672,52.1364326,52.1366076,52.1366811,52.13696,52.1370005,52.1375958,52.1380290043008,52.1382287,52.1384264,52.1385225,52.1386204,52.1386,52.1385784886356,52.1385475,52.1384481,52.1382756425716,52.138258]}]],[[{"lng":[5.042248,5.04214117541594,5.04165037478443,5.0412368,5.0410144,5.0405104,5.0404139,5.0402651,5.0401002,5.0397045,5.039375,5.0392346,5.0387335,5.038426,5.038006,5.03774412464251,5.03638261017427,5.03582719389807,5.0346453,5.0336389,5.033366,5.0331546,5.0329275,5.03283896023153,5.03280693723774,5.03242,5.0321576290849,5.03196287003272,5.03177395197616,5.0315329,5.0313923,5.0313231,5.0313194,5.031318,5.03135776031436,5.0313652,5.0314856,5.0316299,5.0316932,5.0318493,5.0320762,5.0325431,5.0327305,5.032829,5.03285663793434,5.0329255,5.0329697,5.0329642,5.0329008,5.0328538,5.0327288,5.0325415,5.0323384,5.0321528,5.03209717338235,5.0319154,5.0316824,5.0311937,5.03113769207845,5.030456,5.0296507,5.0279786,5.0278245,5.02745177557483,5.0272284,5.0265645,5.0258963,5.0251918,5.0245679,5.02435887496675,5.0236963,5.0228912,5.0226753,5.02204856460489,5.02148845692725,5.0212377,5.0209858,5.0207521,5.02069082910576,5.0205834,5.0204635323785,5.02046019629988,5.02033,5.0203274,5.0203697,5.0203617,5.0203166,5.0202045,5.02009435139863,5.019812,5.0194898,5.0191986,5.0189852,5.0187239,5.018494,5.0183138,5.0181607,5.0175582,5.0170522,5.0164844,5.0162769527153,5.01558169101853,5.0153073,5.0151086,5.0146972,5.0137325,5.0132891,5.0130316,5.0128622,5.012781,5.0123118,5.01219861128718,5.0119716,5.0118333,5.0117448,5.0116315321075,5.0115545,5.0113803,5.0110866,5.0108735,5.0107029,5.0106129,5.0105318,5.0102609,5.01006052334099,5.0096828,5.0096201,5.0094838,5.0089988,5.0088547,5.0085709,5.00797021707393,5.00679678271911,5.00650130213308,5.0052496,5.00425872751955,5.00373981684269,5.0034922,5.00343305980514,5.0032298,5.00274802452142,5.0019306189752,5.001748],"lat":[52.138258,52.1382545263797,52.1382482519849,52.1382381,52.1382394,52.1382339,52.1382328,52.1382294,52.1382241,52.1382133,52.1382036,52.1382013,52.1382756,52.138345,52.1384382,52.1384989215058,52.1389292010714,52.1392526337368,52.139995,52.1406432,52.1408287,52.1410064,52.1412342,52.1413478473801,52.1413889512037,52.1418887,52.1422825546849,52.142580883159,52.1429035090039,52.1433859,52.1437583,52.1440599,52.1443518,52.1444642,52.1448343418585,52.1449036,52.1453411,52.1457111,52.1458499,52.1461923,52.1466143,52.1472502,52.1475444,52.1477594,52.1478156210885,52.1479557,52.1481964,52.1485683,52.1488855,52.1491207,52.1495151,52.1499284,52.1502417,52.1504692,52.1505238426,52.1507024,52.1508737,52.1511471,52.1511649569815,52.1513823,52.151543,52.1517824,52.1518054,52.1518610491969,52.1518944,52.1520505,52.1522411,52.1524714,52.1526724,52.1527319228582,52.1529206,52.1530862,52.1531374,52.1532806039253,52.1534779244447,52.1536153,52.1537807,52.1539849,52.1540487496547,52.1541607,52.1543578287419,52.1543641773019,52.15468,52.155075,52.1554137,52.1556716,52.1559236,52.156185,52.1563255137015,52.1566857,52.1569853,52.1571875,52.1573357,52.1575134,52.1576698,52.1577784,52.1578706,52.1581584,52.1583538,52.1585164,52.1585503254723,52.1586640268512,52.1587089,52.1587414,52.1588576,52.1592443,52.1594401,52.1595906,52.1597141,52.1597733,52.1601257,52.160217495504,52.1604016,52.1605437,52.1606345,52.1607658623172,52.1608552,52.1610968,52.161527,52.161948,52.1623653,52.1625898,52.1627918,52.1635207,52.1642639874454,52.1656651,52.1658376,52.1659971,52.1663449,52.1664337,52.1666245,52.1669984529314,52.1677482372555,52.1679329397756,52.1687775,52.1694031734004,52.1697237377053,52.1699067,52.1699513057625,52.1701202,52.170632864507,52.1715165465609,52.171829]}]],[[{"lng":[5.001748,5.00157040636161,5.00150113470515,5.0016084,5.0016944,5.00177659088922,5.00205323199947,5.002095,5.0022399,5.00231291914677,5.0024764,5.002512,5.0025388,5.0025556,5.0026645,5.0026724,5.00267857634223,5.0026857,5.00269100377996,5.0026986,5.0027683,5.0028331,5.00285154190026,5.002968,5.0030311138355,5.00309,5.00377,5.00393686451943,5.00418069792101,5.0042176,5.0043403,5.0046766,5.0047978,5.0048679,5.0048944,5.0048959,5.0049271,5.0050022,5.0052512,5.0053182,5.00534,5.0053556,5.005369,5.00537295093751,5.0053879979766,5.0055341,5.0055769,5.0055644,5.0055895,5.0056425,5.0057322,5.0057551,5.0057732,5.0057927,5.0057955,5.0058002,5.0058015,5.005802,5.0057486,5.0056967,5.0055795,5.005185,5.004999,5.0048675,5.0047336,5.0046,5.0043856,5.00419124882585,5.0037353,5.0032324,5.0030499,5.0029564,5.0028668,5.0028178,5.0027511,5.00274865985471,5.0027474,5.0027737,5.0027846,5.0028240953265,5.0029043,5.0030513,5.00325,5.0035131,5.0037595,5.003955,5.0042867,5.0045202,5.0048685,5.0052754,5.0058392,5.0064836,5.0069162,5.0072903,5.0074051,5.0077943,5.008005,5.008121,5.008636,5.0089962,5.0095479,5.0103571,5.0107878,5.011194,5.0114953,5.0117566,5.0119716,5.0129837362602,5.01344995838255,5.0135223,5.013836,5.0141542,5.0144096,5.0147643,5.01532024848483,5.0153583,5.0159246,5.0163201,5.0167124,5.016988,5.0173908,5.0177958,5.0179682,5.0182133,5.0183732,5.0185765,5.0187245,5.0188029,5.0188559,5.01888140534938,5.0188933,5.0189098,5.0188762,5.0187855,5.0187551024754,5.0186616,5.0185965,5.0185905,5.01859358354204,5.0186061,5.0186114,5.01862825603903,5.01870580463345,5.0188511,5.0188827,5.01897121602635,5.0191646,5.01936,5.01940147356424,5.0196775,5.0199194,5.0203441,5.02041,5.0204685,5.02057149161302,5.0206864,5.0207378,5.02087,5.0210426,5.0211579,5.0213392,5.02157247877707,5.0216593,5.0216846,5.0219755691839,5.022115],"lat":[52.171829,52.1721623031157,52.1724257011243,52.1728398,52.1731571,52.1734351695479,52.174371093463,52.1745124,52.1747434,52.1748791882896,52.1751832,52.1752915,52.1754564,52.1757173,52.1761752,52.1766653,52.1768421852746,52.1770462,52.1771327051826,52.1772566,52.1776049,52.1777983,52.1778469408508,52.1781541,52.1782709642042,52.17838,52.1795,52.1798357409376,52.1801363337387,52.1801871,52.1803842,52.1809307,52.1812197,52.1814624,52.1815933,52.1817219,52.1818044,52.1819401,52.1828497,52.1829844,52.1830541,52.1831201,52.1832662,52.1832830568255,52.1833472555331,52.1839706,52.1842761,52.1844672,52.1846252,52.1848128,52.1850304,52.185123,52.1851947,52.1856065,52.1856657,52.1857658,52.185982,52.1860604,52.1863258,52.1864985,52.1868483,52.1881975,52.1886924,52.1889348,52.1891237,52.18925,52.1894536,52.1895961884884,52.1899307,52.1903673,52.190552,52.1906762,52.1908228,52.1909979,52.1913613,52.1916259572467,52.1917626,52.1920293,52.19214,52.1922804942689,52.1925658,52.1929688,52.1933819,52.1938487,52.1941978,52.1944474,52.1947712,52.1949178,52.1951104,52.1952905,52.1955135,52.1957754,52.1959441,52.1961274,52.1961762,52.1963448,52.1964421,52.196481,52.1966309,52.1967101,52.1967611,52.1967715,52.1968034,52.1968539,52.196936,52.1970463,52.1971762,52.1979741903931,52.1983417653048,52.1983988,52.1986104,52.1988002,52.1989461,52.1991193,52.1993194976418,52.1993332,52.1994736,52.1995742,52.1997075,52.1998637,52.2001678,52.2005097,52.2006552,52.2008775,52.2010639,52.2014467,52.201811,52.2020198,52.2022676,52.2025492503973,52.2026806,52.2028123,52.2030022,52.2032205,52.20327214399,52.203431,52.2036691,52.204197,52.2043136612952,52.2047872,52.2048449,52.2056696916568,52.2061914199502,52.206559,52.2066569,52.2068763857619,52.2073559,52.20764,52.207678534604,52.207935,52.2081043,52.2083949,52.20844,52.2084768,52.2085444370461,52.2086199,52.208654,52.20874,52.2088525,52.2089276,52.2090457,52.2091854778683,52.2092375,52.2092527,52.2094029617657,52.209477]}]],[[{"lng":[5.022115,5.02224127349769,5.0224324086991,5.0225888,5.0231409,5.0238097,5.0253161,5.0265152,5.0266279,5.0275858,5.0277528,5.0280192,5.0283084,5.0285517,5.0287957,5.0289404,5.0290663,5.0292755,5.0294867,5.0296876,5.0304963,5.0310257,5.0313998,5.0315408,5.0314893,5.0313491,5.03124,5.03108,5.03098,5.03082,5.0303993,5.02919,5.02907,5.02904,5.02909,5.0293501,5.0301443,5.0312252,5.031819,5.0322815,5.0323743,5.0324536,5.03246,5.03243,5.03235,5.03225,5.03224,5.0322492,5.0322866,5.0322832,5.0322744,5.0323072,5.03233386348248,5.0323786,5.0325,5.0327357246027,5.03262488995237,5.033258],"lat":[52.209477,52.2095023476968,52.2095510431754,52.2096086,52.2097888,52.209956,52.2103966,52.2107237,52.2106845,52.2103804,52.2105003,52.2107452,52.2110598,52.2114481,52.2119083,52.2122368,52.2124921,52.2129161,52.2133842,52.2136637,52.214688,52.2153725,52.2159428,52.2162655,52.2164419,52.2166116,52.21686,52.21731,52.21745,52.21756,52.217782,52.21842,52.21857,52.21878,52.21887,52.2190824,52.2194628,52.2198484,52.2200479,52.2202884,52.2203962,52.2205443,52.22071,52.22117,52.22167,52.22256,52.22286,52.2236287,52.2243518,52.2253966,52.2260134,52.2266465,52.2268105145349,52.2270857,52.2274639,52.2282860334503,52.2297880321564,52.229807]}]],[[{"lng":[5.033258,5.0328881742658,5.03295370868573,5.03284448465252,5.03264788139275,5.03223283006657,5.03173039951383,5.03133719299429,5.03120612415444,5.03135903780093,5.03153379625406,5.03218914045329,5.03328138078534,5.03404594901778,5.03465760360372,5.03553139586937,5.0376284973069,5.03941977145147,5.04033725333039,5.04195376902183,5.04381057758631,5.04464068023867,5.04490281791836,5.04490281791836,5.04470621465859,5.04400718084608,5.04280571648083,5.04156056250229,5.03983482277765,5.03773772134011,5.03684208426783,5.03649256736157,5.03631780890844,5.03638334332837,5.03657994658814,5.03712606675416,5.03813092785965,5.03926685780498,5.04027171891047,5.04271833725426,5.04416009449257,5.04543166855391,5.04595136863713,5.04634457515667,5.04643195438324,5.04632273035003,5.04616981670354,5.04549262769767,5.04514311079142,5.04496835233829,5.0444440769789,5.04400718084608,5.04372319835975,5.0435921295199,5.04361397432655,5.043433],"lat":[52.229807,52.2303458083093,52.2311753414814,52.2316569987908,52.2319647215581,52.2321787883121,52.232366095875,52.232620297732,52.2329681505458,52.2334765459089,52.2339447996447,52.2348144006206,52.236433150909,52.2375434992019,52.2382792568651,52.2388410999582,52.2399647648066,52.2409145069564,52.2414361876576,52.2420648716362,52.2425865388142,52.2429476894219,52.2432419581142,52.2434425947401,52.2436833574937,52.24387061651,52.244098001396,52.2443120096361,52.2445527676724,52.2450075292879,52.2453017843201,52.2455024116324,52.2457832883456,52.2460106634301,52.2462647871457,52.2464386604283,52.2466794069255,52.2468265291418,52.2469067774178,52.2470271495597,52.2472545182699,52.2475359786751,52.2476958777343,52.2480302380418,52.2483645958295,52.2487524477063,52.2490600519563,52.2495682630009,52.2499694781343,52.2503706896393,52.2514405692464,52.2525772880574,52.2538610766292,52.2544762121528,52.2550779669106,52.255641]}]],[[{"lng":[5.043433,5.04552327238512,5.04677016220139,5.048368],"lat":[52.255641,52.2651848411477,52.2709840687894,52.276948]}]],[[{"lng":[5.048368,5.04911886560008,5.0494465376997,5.04959945134618,5.04966498576611,5.04975236499267,5.04977420979931,5.04986158902588,5.05012372670557,5.05049508841847,5.05099751897121,5.05167470797708,5.05294170676226,5.05444899842049,5.05529002347617,5.05624027256505,5.05748542654359,5.05964806240105,5.06018326016376,5.06093690599287,5.06141749173898,5.06207283593821,5.06250973207103,5.062793],"lat":[52.276948,52.2772970576011,52.2775910984524,52.2779519641037,52.2784331137346,52.2795023164346,52.2800636375257,52.2805180351062,52.2810659789327,52.2816406444473,52.2820415694626,52.2822687586945,52.2824825827898,52.2827966350592,52.2830906394218,52.2834848694826,52.2840795488706,52.2849281224914,52.2850817994406,52.2851753416704,52.2851753416704,52.285088481035,52.2849748937933,52.284801]}]],[[{"lng":[5.062793,5.06294662820385,5.06288109378392,5.06277186975072,5.06272818013744,5.06299031781713,5.06353643798316,5.06421362698903,5.06465052312185,5.06524033290115,5.06563353942069,5.06615781478008,5.06711898627228,5.06816753699105,5.06912870848326,5.06967482864928,5.06995881113561,5.0700025007489,5.06982774229577,5.06936900135631,5.06910686367662,5.06919424290318,5.06947822538951,5.07004619036218,5.07116027550087,5.07231805025285,5.07279863599895,5.07354135942474,5.07424039323726,5.07498311666305,5.07552923682908,5.076446718708,5.0777792519131,5.07898071627836,5.08016033583697,5.080706456003,5.08092490406941,5.08094674887605,5.08079383522956,5.08048800793659,5.07978897412407,5.07908994031156,5.07836906169241,5.07773556229982,5.07712390771387,5.07649040832128,5.0760753569951,5.07507049588961,5.07386903152436,5.07247096389933,5.07102920666103,5.06941269096959,5.06757772721174,5.06607043555351,5.06502188483474,5.06360197240308,5.06264080091087,5.06187623267844,5.06139564693233,5.0610024404128,5.06041263063349,5.05951699356121,5.05853397726236,5.05744173693031,5.05624027256505,5.05510434261972,5.05440530880721,5.0538154990279,5.05359705096149,5.053444137315,5.05305093079547,5.05265772427593,5.05204606968998,5.05099751897121,5.04973052018603,5.04852540113184,5.0483774806687,5.04767868090409,5.04682792333807,5.045592],"lat":[52.284801,52.2845606319735,52.2840929123606,52.2833311869947,52.2824625368247,52.2819279744099,52.2813399483053,52.280832101299,52.2806449983032,52.280538081951,52.2805247173888,52.2806449983032,52.2809924746666,52.2815805053821,52.2821150719886,52.2827298156138,52.2833980055332,52.283879096036,52.2842800007983,52.2847610817248,52.2852822468341,52.2857499538922,52.2862577445367,52.2868323427209,52.2876073703272,52.2885961789685,52.2895047944781,52.2910681040171,52.2921503629185,52.2927382455598,52.2930455447443,52.2932058739073,52.2933127596936,52.2934864485462,52.2938071031,52.2942747201634,52.2947690528337,52.2952901002165,52.295851221312,52.2964123352986,52.2970669592982,52.2974944636261,52.2978551671929,52.2980154789463,52.2980421975154,52.2980822753389,52.2980154789463,52.2976013390633,52.2969868017773,52.2966795299365,52.2966528105453,52.2966528105453,52.2965459328191,52.2962119382623,52.2960249002104,52.2959581007147,52.2960516199804,52.2963455363873,52.2967997669977,52.2974811041783,52.2980421975155,52.2986433610595,52.2991376449713,52.2994849763115,52.2996853585378,52.2999124572982,52.3002464239478,52.3007941238008,52.301448683032,52.3024505406678,52.3030249288222,52.3033054412594,52.3035725943092,52.3039733208618,52.3045877611984,52.3054021260492,52.3055195498611,52.3058409561481,52.3062150224817,52.30635]}]],[[{"lng":[5.045592,5.045145391755,5.04441130976894,5.04386518960292,5.04336275905017,5.04336275905017,5.04360305192323,5.04412732728261,5.04585306700725,5.04845259899753,5.05195101967111,5.05225398231035,5.05255177920548,5.05281974867613,5.05309724264704,5.05322903937502,5.05335697135416,5.05348160735662,5.05360314232215,5.05372186498771,5.05537740270274,5.05649148784143,5.05686284955432,5.05708129762073,5.05708129762073,5.05684100474768,5.05574876441563,5.05369535259138,5.05227544015971,5.05159825115384,5.05150659489022,5.0515179007101,5.05153183943562,5.05154842830314,5.05156768612363,5.05160740629064,5.05165908430253,5.05172337336534,5.05181399987617,5.05186398507464,5.05191738092464,5.05197420490424,5.05203448473677,5.05216552432754,5.05231086748531,5.05247037515657,5.05264555452322,5.05283774844862,5.05305048256549,5.05321670802464,5.05340853458746,5.05360276817098,5.05404152015419,5.05429322785456,5.05455838144876,5.05483917069004,5.05513957025177,5.05545246084349,5.05580072943999,5.05616601811378,5.05718822257569,5.05750164900546,5.05778141481189,5.05810158881364,5.05839772738314,5.05867571772749,5.05893848762811,5.05933351711073,5.05972365068509,5.06010838202852,5.06048724290375,5.06102226990934,5.06260803370091,5.06391872209937,5.0645740662986,5.06505465204471,5.06514203127127,5.06485804878493,5.06404979093922,5.06359104999976,5.06374396364624,5.06418085977906,5.0647092630853,5.06632165082988,5.06787369782909,5.06898671724009,5.069393],"lat":[52.30635,52.3070726995845,52.3074094021635,52.3080104385441,52.3088518757686,52.3091857749966,52.3094662484018,52.309639873905,52.3098134987273,52.3099337001284,52.3099409677449,52.3099273367365,52.3099175451825,52.3099132301926,52.3099146909518,52.3099179229779,52.3099228390395,52.3099294641214,52.3099378113667,52.3099479004728,52.3100806125087,52.3102408800039,52.3104545690948,52.3107617453559,52.3110822748339,52.3112558939988,52.3118034576736,52.3126047581592,52.3133659801846,52.3141138346887,52.3145292868377,52.314566884047,52.31460305608,52.3146378515184,52.3146713045679,52.3147272786979,52.3147875567988,52.3148532327163,52.3149383401392,52.3149826160828,52.3150265808077,52.3150702514356,52.3151136478131,52.3151996963067,52.3152849717494,52.3153694485292,52.3154539712035,52.3155392013993,52.3156267056656,52.3156883733607,52.3157511367505,52.3158090334664,52.3159338851645,52.3160043627065,52.316072727648,52.3161395477828,52.3162057715747,52.3162701116431,52.3163375240162,52.3164050063489,52.3165895586724,52.3166486567501,52.3167038665642,52.3167709251868,52.3168376608984,52.3169054024342,52.3169748716704,52.3170858688736,52.3171997758068,52.3173164455174,52.3174357356576,52.3176078140716,52.3181867490048,52.3188410511134,52.319295050973,52.3198024570689,52.3204567352888,52.3210308898387,52.3218987837372,52.3233007302295,52.3244089041177,52.3252900828208,52.3259700871315,52.3275597047213,52.3287560552162,52.3298559095598,52.329933]}]],[[{"lng":[4.893505,4.89494098189235,4.89795268865435,4.90016289281033,4.90378179851628,4.91281691880227,4.93181010176905,4.95298920093405,4.97173950432326,4.99913632067432,5.02585307420816,5.0468378697111,5.05762172295567,5.06432519929688,5.06821127253817,5.06918279084849,5.07005715732778,5.07030003690536,5.07015430915881,5.069393],"lat":[52.373362,52.373933260568,52.3759794679312,52.3772842463116,52.3779811007898,52.3778328348005,52.376379801761,52.3736811849861,52.3695884624793,52.3617578496519,52.3552312786701,52.3497126075586,52.3458550627298,52.3417597621973,52.3372485573767,52.3347256435129,52.3335086571363,52.3324103723926,52.3316089041519,52.329933]}]],[[{"lng":[4.89257,4.893505],"lat":[52.373074,52.373362]}]],[[{"lng":[4.896699,4.89669514661371,4.89662898738366,4.89647643502355,4.89645392094347,4.8963949,4.8962318,4.89608109619885,4.8960783,4.89581707917964,4.8956662,4.89564755162875,4.8956169,4.89532907343313,4.89530796943382,4.89514511786695,4.8951449,4.89513874899287,4.89505705072328,4.89492638426702,4.89487695511946,4.8948336,4.89479613157097,4.89475169585165,4.8947434,4.89473372937231,4.894634,4.89448209336504,4.89443821802396,4.894278,4.89412532099127,4.8941189,4.89411560389932,4.8939683,4.89392788344664,4.89366590105302,4.89364595125672,4.8936383,4.89332109261377,4.8933128,4.8929153,4.89290194098739,4.8927035,4.8926997604585,4.8925116,4.8924615925118,4.89257],"lat":[52.377087,52.3770764208542,52.3769267381544,52.3767208009708,52.3766934916934,52.3766219,52.3764672,52.3763540003397,52.3763519,52.3761880429029,52.3760934,52.3760820142828,52.3760633,52.3758798188773,52.3758657561026,52.3757429642742,52.3757428,52.3757375811007,52.3756529495479,52.3755175908599,52.3754676756111,52.3754074,52.3753674392397,52.3753200476909,52.3753112,52.3753022100384,52.3752095,52.3750925453636,52.3750614322531,52.3749666,52.3748762017473,52.3748724,52.3748703229772,52.3747775,52.3747457423788,52.3745398871215,52.3745232556885,52.3745163,52.3742371965041,52.3742299,52.3738635,52.3738514592524,52.3736726,52.3736687571862,52.3734754,52.3732676792854,52.373074]}]],[[{"lng":[4.896699,4.89680326060991,4.89720576906449,4.897774],"lat":[52.377087,52.3771009362406,52.3772757235801,52.377573]}]],[[{"lng":[4.897774,4.90347349671191,4.91051244992805,4.92563767465894,4.95257567554841,4.99529651963534,5.0155609109012,5.07810773394294,5.40001678202874,5.71597036664727,5.78310068394331,5.80918461884208,5.83681760225184,5.85243117317531,5.85960203461171,5.86281705822601,5.86530139465524,5.86651920663035,5.86632435671433,5.86600404035119,5.86778573108447,5.86997779263967,5.87212114171586,5.87445934070808,5.87913573869251,5.88551707344209,5.88868338457738,5.89253167041874,5.89525956924299,5.89711064344516,5.89847459285728,5.89944884243737,5.90183575390859,5.90310227836271,5.90406130529311,5.90855807601121,5.91227193818554,5.9151858556728,5.91716772601591,5.917383],"lat":[52.377573,52.3792348801742,52.37854352046,52.3773466248271,52.374283613738,52.3697035698087,52.3678892652441,52.3862668058788,52.4847828681818,52.5795055011051,52.5996048047764,52.6091001762403,52.615681400442,52.6108535590161,52.6088931112976,52.6047811903974,52.6029765564573,52.6001659129057,52.5973255004279,52.5955263806987,52.5930053534195,52.5906971550926,52.5877081535286,52.5861987782461,52.5848669333571,52.5839198190356,52.5831206754079,52.5798351540846,52.5781478987893,52.5766085917007,52.5744179461234,52.5734706060285,52.5720495575088,52.5703027889433,52.568599437549,52.566779433864,52.5648128892881,52.5630582135134,52.5605850460572,52.55894]}]],[[{"lng":[5.917383,5.918985],"lat":[52.55894,52.559609]}]],[[{"lng":[5.918985,5.920472],"lat":[52.559609,52.560243]}]],[[{"lng":[5.920472,5.92027774204179,5.91955925894178,5.92062528220598,5.92028,5.9203635,5.9203736,5.92038,5.9203622,5.9203148,5.9203024,5.9203722,5.9205028,5.9208754,5.9221608,5.9232963,5.923594,5.923852],"lat":[52.560243,52.5603855758383,52.5610592116296,52.5616870076991,52.5626,52.5647574,52.5650191,52.56518,52.5655391,52.5656966,52.5658453,52.5659386,52.5660128,52.5661767,52.5667653,52.5672765,52.5674376,52.567599]}]],[[{"lng":[5.923852,5.92527,5.92571,5.92838,5.92855,5.9287609,5.9289151,5.9291657,5.9295698836572,5.9302152,5.93201285363767,5.9334513,5.9334784,5.9335074,5.9335811,5.9336128,5.9336393,5.9336552,5.9335792,5.9333154,5.9332904,5.9332872,5.9333185,5.9333585,5.9335162,5.93361794716597,5.93371147855064,5.9337196,5.93383,5.9338936,5.93394295098161,5.9340725,5.9344603,5.934512,5.9344929,5.9344304,5.9343217,5.93402633876218,5.933873],"lat":[52.567599,52.56888,52.56924,52.57158,52.57177,52.5722027,52.5725949,52.5729885,52.5734582076747,52.5739853,52.5743220245107,52.5758198,52.5758605,52.5759059,52.5760382,52.5761136,52.5761953,52.5762947,52.5772099,52.5782561,52.5784561,52.5786211,52.5788063,52.5789963,52.5794108,52.5798035958307,52.5801789471009,52.5802098,52.58063,52.5807749,52.5808878641997,52.5811844,52.5820216,52.5822416,52.5825072,52.5827058,52.5828854,52.5831885794487,52.583394]}]],[[{"lng":[5.933873,5.935207,5.9362672,5.93854883113141,5.94017292422424,5.9416239,5.9421404,5.9425789,5.9428222,5.9448369,5.9453818,5.9458917,5.9467522,5.9471756,5.9483752,5.94892486854793,5.94987984772211,5.95024629321918,5.95070157398827,5.95102360184933,5.95116240696186,5.95126789884738,5.95171762741196,5.95247272722411,5.9534204,5.9550124,5.95850242131227,5.95925752112441,5.9597016974845,5.96014587384458,5.96043458847864,5.96096760011074,5.96745257496799,5.9680026,5.96871847759423,5.96928480245334,5.96978450085844,5.97052849626158,5.97099488143967,5.97174998125181,5.97239403697394,5.972716064835,5.97302698828706,5.97342674701114,5.97418184682328,5.97504243852095,5.97595855226362,5.97652487712273,5.97701624722108,5.97741600594515,5.97793236096375,5.9787553,5.9791114,5.979467,5.979851,5.9807844,5.9811658,5.9816042,5.9820807,5.9826819,5.9829495,5.9831357,5.9832744,5.9834564,5.98407,5.98465,5.98479,5.9850921,5.9853864,5.9856874,5.985984,5.9864358,5.986902,5.9872902,5.9877777,5.98817340216597,5.98845101239102,5.98875083143408,5.98921721661217,5.98980575028928,5.9905258,5.9907054,5.9911181,5.99245,5.99534022755119,5.99660414395266,5.9971177,5.997431,5.9977006,5.99836821260715,5.99882111303988,5.9990383,6.0001429,6.0004059,6.0005721,6.0008126,6.0015911,6.0019347,6.0022034,6.0025602,6.002914,6.003261,6.0039524,6.0043532,6.0056658,6.0131505,6.0137259,6.0142173,6.0144333,6.015172,6.0154484,6.0156788,6.015805],"lat":[52.583394,52.584044,52.5844123,52.5854225206564,52.5860860991527,52.5866538,52.5867953,52.5868694,52.5868978,52.5870745,52.5871448,52.5872672,52.5875319,52.5876396,52.5877869,52.5877494880619,52.5874964959918,52.5874391509196,52.5875099889391,52.587638171731,52.5878574309007,52.5881711382666,52.5883094386502,52.5883802752631,52.5886603,52.5889723,52.5894495568902,52.5893281255896,52.5892471712023,52.5893146332021,52.5895979724674,52.5898408332366,52.5910821005931,52.5913005,52.5915408209356,52.5916352627629,52.5916420085999,52.591999536476,52.5924649929248,52.5926538723923,52.5926336353454,52.592802277117,52.5931935235274,52.5934768377127,52.593598257517,52.5935544115154,52.5933773406777,52.5933419264243,52.5934279324186,52.5936555945299,52.5942120969331,52.5944071,52.5944703,52.5944982,52.5944883,52.5943494,52.5943101,52.5943428,52.5944402,52.5946282,52.594736,52.5948279,52.594921,52.595096,52.59585,52.5965,52.5966297,52.5968038,52.5969126,52.5969774,52.5970191,52.5970245,52.5970032,52.5970022,52.5970262,52.5971530104344,52.5974093192313,52.5977128408672,52.5978477386971,52.5979016977127,52.5980071,52.5980257,52.5980313,52.59798,52.5979015146667,52.5978950378292,52.5979326,52.5979314,52.5979076,52.5977960799301,52.5977386217028,52.5977293,52.5977506,52.5977426,52.5977214,52.5976718,52.5974751,52.5974406,52.5974385,52.5974535,52.5974937,52.5974776,52.5975005,52.5975383,52.597703,52.5983395,52.5984377,52.5985621,52.5985989,52.5986103,52.5986585,52.5987595,52.59888]}]],[[{"lng":[6.015805,6.0159276,6.0160031,6.0160478,6.0161145,6.0162,6.0173687,6.0181783,6.0184623,6.0186486,6.0189873,6.0192287,6.0202587,6.0206115,6.0209469,6.0210776,6.0212563,6.0213276,6.0213548,6.0213577,6.0213301,6.0212102,6.0212008,6.0213005,6.0214376,6.0214591,6.0213719,6.0213483,6.0213984,6.0214289,6.0215335,6.0216672,6.0218373,6.0221172,6.022385,6.0257922,6.0269024,6.027221,6.0275005,6.0276367,6.0298432,6.0307653449711,6.03152016917065,6.03185011369976,6.0328373,6.0330769,6.0332042,6.0347195,6.0365035,6.037644,6.038068,6.03945,6.0395395,6.0396829,6.03982743092725,6.03998229198031,6.03998522520622,6.0400048,6.04031,6.04059,6.0408499,6.0410863,6.0411341,6.0411332,6.0410618,6.040893],"lat":[52.59888,52.599092,52.5994491,52.5995355,52.5996122,52.5996851,52.60025,52.6006757,52.6007992,52.6008642,52.6009386,52.6009681,52.6010273,52.6010899,52.6012139,52.6012965,52.6014799,52.6015888,52.60168,52.6017866,52.6020328,52.6027176,52.6029799,52.6033522,52.6037498,52.6040209,52.6046311,52.60499,52.6067506,52.606889,52.6071111,52.6072822,52.6074521,52.6076441,52.6077705,52.6090629,52.6094075,52.609545,52.609702,52.609797,52.6116496,52.6123445057456,52.6129993426103,52.6134089310367,52.6151377,52.6153991,52.6155041,52.616399,52.6174053,52.6180309,52.6182238,52.6187423,52.6187825,52.6188695,52.619014019133,52.6193577512399,52.6197320623344,52.6200091,52.62064,52.62098,52.621271,52.62159,52.62179,52.6220681,52.6223381,52.622679]}]],[[{"lng":[6.040893,6.0408301120661,6.04074383645437,6.0404828,6.04045323062906,6.04044242672135,6.0402412,6.0398964,6.0395993,6.040333],"lat":[52.622679,52.6227293959715,52.6227924432627,52.6229583,52.6229761444561,52.6229826643716,52.6231041,52.6233091,52.6235356,52.623892]}]],[[{"lng":[6.040333,6.04058836418374,6.0406551783248,6.04035796515644,6.0394393062724,6.03838555049366,6.03846660863049,6.0425737,6.0436608,6.0439887904408,6.04485],"lat":[52.623892,52.6240252216999,52.6243106799232,52.6252783759585,52.6263772590712,52.6264920661627,52.6295753439565,52.6329424,52.6339025,52.6341297525933,52.634715]}]],[[{"lng":[6.070683,6.07049591125947,6.07024852013893,6.06988969748992,6.06929165974155,6.0685062,6.0680351,6.0678034,6.067559,6.067422,6.0673338,6.0672894,6.0672338,6.0671885,6.0671437,6.0671055,6.0670652,6.0670143,6.0669776,6.0669475,6.0669008,6.0668649,6.0668517,6.066839,6.066822,6.0668074,6.0667857,6.0667652,6.0667366,6.0667,6.0666536,6.0666093,6.06641938412815,6.0661957,6.0655534,6.0654556,6.0653677,6.0652902,6.0652172,6.0651343,6.0650759,6.0649525,6.0645063,6.06432865783083,6.0642291,6.0638093,6.063754,6.0637235,6.0636904,6.0635729,6.0635287,6.0634452,6.0634,6.0633639,6.0632528,6.0631901,6.0631329,6.0630637,6.0629892,6.0628472,6.0628081,6.0627687,6.0627274,6.0626792,6.062486,6.062329,6.0622458,6.0622077,6.06211500309593,6.06198039254258,6.0619315,6.0611722,6.0608687,6.0606602,6.0605033,6.0603613,6.0602883,6.0601767,6.0600803,6.0600145,6.0599964,6.05985951494084,6.059731,6.0596907,6.05952131911413,6.0593297,6.0592452,6.0591624,6.0590432,6.05774422678656,6.05642,6.0495242,6.0495242,6.0490039,6.0484274,6.0479715,6.0474256,6.0470088,6.0464561,6.04604360877443,6.0458485,6.0455795,6.04485],"lat":[52.640124,52.640183246607,52.6402286291207,52.6402195566193,52.6400834688723,52.6400169,52.6399923,52.6399379,52.6398758,52.6398363,52.6398032,52.6397811,52.6397512,52.6397204,52.6396864,52.6396495,52.6396104,52.6395533,52.639513,52.6394756,52.6394045,52.6393367,52.6393175,52.6393023,52.639284,52.6392688,52.6392534,52.6392408,52.6392275,52.639214,52.6392039,52.639197,52.6391841430283,52.639169,52.6391016,52.6391016,52.6391032,52.6391124,52.6391308,52.6391711,52.6392149,52.6393077,52.6395556,52.6396291049293,52.6396703,52.6397901,52.6398015,52.6398034,52.639803,52.639788,52.6397782,52.6397493,52.6397322,52.6397155,52.6396602,52.639626,52.6395938,52.639547,52.6394926,52.6393828,52.6393598,52.6393393,52.6393215,52.6393042,52.6392484,52.6392014,52.6391714,52.6391563,52.6391213032696,52.6390594895824,52.6390301,52.6386117,52.6384298,52.6383009,52.6381997,52.6380969,52.6380375,52.637948,52.6378577,52.6377908,52.6377718,52.6376322732234,52.6374539,52.6373691,52.6370251227619,52.6366775,52.6366248,52.636582,52.6365336,52.6361802363721,52.63582,52.6341419,52.6341419,52.63405,52.63401,52.6340142,52.634061,52.6341369,52.6342847,52.634409886691,52.6344691,52.6345508,52.634715]}]],[[{"lng":[6.070683,6.07084595642978,6.07109288542589,6.07159556231082,6.07191304244867,6.07259209718797,6.07320060078553,6.07353571870882,6.0738179232758,6.07415304119909,6.07484973372382,6.07528185946701,6.07546705621409,6.07566107185389,6.07608437870436,6.0764900477694,6.07716028361598,6.07760122825188,6.07800689731692,6.07840374748924,6.07910044001397,6.08016752603287,6.08072311627411,6.08120815537361,6.08160500554593,6.08214295800173,6.08263681599395,6.08308657952258,6.08342169744587,6.08425067336137,6.08501791702785,6.08532657827299,6.08550295612735,6.08585571183608,6.08604090858316,6.08628783757927,6.08662295550256,6.08721382131468,6.08752248255981,6.08788998338645,6.08821925471811,6.08906968012371,6.0897391,6.0899042,6.0938568278233,6.0976344,6.0980149979846,6.098133,6.0983045,6.0985232,6.0986411,6.098771,6.1009535,6.1011201,6.1011785,6.1012221,6.1012606,6.1012856,6.1012745,6.1012546,6.1011927,6.1011581,6.1010485,6.1009682,6.10094776550838,6.100831,6.1008022,6.100638,6.1006003687187,6.10028062775631,6.1002045,6.1002246,6.1002592,6.1002936,6.1003988,6.1004248,6.1005781,6.10063178121055,6.1007049,6.1007882,6.1008622,6.1010043,6.1011139,6.1012704,6.1014148,6.10148292762701,6.10155426947864,6.1016308,6.10210923013575,6.1023,6.10261,6.10641398303735,6.10672,6.1071,6.10816639490153,6.10837577774277,6.10844557202318,6.10849675449549,6.10857178334693,6.1086503019124,6.10879745152027,6.10898996741041,6.10940175366484,6.10965068659832,6.11024277474382,6.11121407847957,6.11276816445677,6.11365690675687,6.11528,6.1168937,6.1171638,6.11896,6.1197172,6.1202457,6.12087,6.12712,6.12791,6.12987,6.1302239,6.1310369,6.13410752385957,6.13452071518551,6.13470360938143,6.13492086905988,6.13522098446565,6.13553133593356,6.1359212537458,6.13674249977866,6.13776149627269,6.13848503031297,6.13890072132545,6.14093756997536,6.14365785776906,6.14482164935531,6.14593,6.14621280155152,6.14666697433575,6.14711167976486,6.14742657789938,6.14783154188012,6.14901349751329,6.14963699308498,6.15008832943165,6.15035820064925,6.15035122122121,6.15021861208842,6.15020930618437,6.15046056559385,6.1529432,6.15421,6.154775,6.15521,6.15652572989483,6.1569571,6.15815920205224,6.15855,6.1591663,6.1595394,6.1598217,6.15983082036962,6.1617113,6.1620624,6.1625296,6.163332,6.1635225,6.1637112,6.1643749,6.1645186,6.1646789,6.1649414,6.1651939,6.1654393,6.1659844,6.1665808,6.1671955,6.1677614,6.1679312,6.169843536991,6.1706926,6.1710548,6.17109063116705,6.17173735197778,6.17227244146095,6.17320303186646,6.17357526802866,6.17385560838832,6.17417084588818,6.1745616938585,6.1742080695044,6.1743755757774,6.17448724662606,6.17476642374771,6.17590174404243,6.1757884,6.1759834,6.1773166,6.1774686,6.1776932,6.1779138,6.1781316,6.1783955,6.1786708,6.1790274,6.1799299,6.1804554,6.1808133,6.1809227,6.1810614,6.1811535,6.1812767,6.1814077,6.1815284,6.1816439,6.18182530672834,6.1819908,6.1823315,6.1824239,6.182969],"lat":[52.640124,52.6402236341312,52.6402396885163,52.6402503914365,52.6403467176,52.6406410462298,52.6409353728791,52.6409407242543,52.6409888866025,52.6411387246799,52.6415668306441,52.6419146636549,52.6422143329537,52.6423748692335,52.6426477795569,52.6427815585053,52.6429313904416,52.6430063062173,52.6430063062173,52.6430277107011,52.6431614884874,52.6434129896179,52.6435949257044,52.6437982651408,52.6440390606192,52.6444885419651,52.64495942223,52.6453607366359,52.6455961727068,52.6460777425405,52.6465218522425,52.6467626327256,52.6469980612495,52.647468914495,52.6476989886689,52.6479130100695,52.6481216799265,52.6484534094445,52.6486834784387,52.649018817694,52.6495806604259,52.6503180961722,52.6507743,52.6508564,52.6528456936282,52.6547468,52.6549658369516,52.6550419,52.6551717,52.6553673,52.655499,52.6556701,52.6589838,52.6592654,52.6593817,52.6595214,52.659681,52.6598722,52.6600734,52.66021,52.660394,52.6605047,52.6607785,52.6609684,52.6610332038263,52.6614035,52.6614896,52.6620324,52.6621344385067,52.6628155914724,52.663068,52.6631458,52.6631627,52.6631793,52.6632328,52.6632392,52.6632808,52.6632954056955,52.6633153,52.6633379,52.6633609,52.6634044,52.6634545,52.6635372,52.6636452,52.6636960434599,52.6637492856234,52.6638064,52.6641306920362,52.66426,52.66442,52.6659193822444,52.66604,52.66615,52.666520762516,52.6666914809149,52.6667701381346,52.6668579659839,52.6670632496661,52.667190934219,52.6672967498642,52.6673969217722,52.6674709924016,52.6674985043177,52.6675196673183,52.6675182564519,52.667444891337,52.667547504478,52.6679,52.6683122,52.6683812,52.66884,52.669033,52.6692083,52.66946,52.67325,52.67363,52.6744847,52.6746595,52.6751126,52.6770007537363,52.6773452467051,52.6776707841657,52.6780053787028,52.6782747886816,52.6785155967687,52.6787303839748,52.6791619961851,52.6795780888863,52.6798756985225,52.6799576647125,52.6808786407907,52.6821653199492,52.6827157745334,52.68324,52.6833998447764,52.6835950996146,52.6837862837503,52.6839216616851,52.6840957592332,52.6846773708531,52.6850299475167,52.6854050859612,52.6858563760604,52.6861377248715,52.6864292962256,52.6866291995236,52.6868435568934,52.6892186,52.69036,52.6908838,52.6912214,52.6918705271124,52.6920243,52.6922868200535,52.6923828,52.6925335,52.6925815,52.6926041,52.6926043562933,52.6926572,52.6926778,52.692722,52.6928173,52.6928321,52.6928369,52.6928164,52.6928173,52.6928161,52.6928321,52.692863,52.692912,52.693038,52.6931956,52.6933775,52.6935399,52.6935627,52.6938231859604,52.6939362,52.6941039,52.6941251059764,52.6942749018918,52.6944708917842,52.6947641699799,52.6949784874153,52.6951913938259,52.695542475639,52.696016220116,52.6968847383046,52.6973359097646,52.6974938186739,52.697629168713,52.6982269596987,52.6986469,52.6987583,52.699596,52.6997126,52.6998829,52.7000846,52.7002885,52.7005427,52.7007791,52.7010221,52.701637,52.701986,52.7022081,52.7022865,52.7023492,52.7023915,52.702428,52.7024608,52.7024802,52.7024951,52.7025115725077,52.7025266,52.7025543,52.7025624,52.702662]}]],[[{"lng":[6.182969,6.18310102406705,6.1836518,6.1836518,6.1845977,6.1846864,6.1847716,6.1850769,6.18539937937201,6.18559566054274,6.1856588,6.18645,6.18666,6.1868824,6.1869857,6.18723,6.1872538,6.1873261,6.1873774,6.1874967,6.18771964454987,6.1881352,6.1885169,6.1887472,6.1888798,6.18974158633647,6.189782,6.18988608938684,6.18990528471556,6.1898611,6.1899268754845,6.19001847174724,6.19038534581459,6.190474],"lat":[52.702662,52.7025952380057,52.7024219,52.7024219,52.7021202,52.7020898,52.702065,52.7019837,52.701859329618,52.7017030449427,52.701657,52.70121,52.70105,52.7007706,52.7006438,52.70031,52.7002841,52.7001809,52.7001486,52.7001009,52.7000170297025,52.6998607,52.6997014,52.6995816,52.6995327,52.6991909343901,52.699177,52.6990720215024,52.6989807983419,52.6987803,52.6986123212421,52.6984428886597,52.6978647975314,52.697755]}]],[[{"lng":[6.190474,6.1906402117825,6.1910196149922,6.1913442,6.1919752,6.192314,6.1926659,6.1931604483298,6.1932180115251,6.1933581051279,6.1937701,6.19427,6.194719,6.1954939,6.1961860190154,6.1965879978111,6.197013714644,6.1971919,6.1974189,6.1975649,6.1976444,6.19812,6.1987803091618,6.1997527761356,6.2003948835154,6.2009765025188,6.2011021322236,6.2010183790871,6.2011021322236,6.2012510266885,6.2016418746588,6.2025686528696,6.2031820017799,6.2044150340672,6.2080396836966,6.2082645,6.2085316,6.2087396,6.2091462,6.2093004,6.2094635,6.2096338,6.2100352,6.2105075,6.2110255,6.2113132,6.2126044338555,6.21458,6.2163693,6.2174759541785,6.2175402,6.2198919,6.2209195,6.22098,6.221182314105,6.22157,6.2254,6.225789,6.2269281,6.23001,6.2305995,6.2312564684679,6.23714,6.23951,6.2414603,6.2446108703154,6.2455374202878,6.24585,6.2462333,6.2464735,6.2466412,6.2467107,6.2476078,6.2477295,6.2484021,6.2494972784509,6.24978,6.2506311,6.2512455,6.2513652199709,6.2516561,6.2520368,6.25346,6.2540407,6.2555801,6.25579,6.2569266,6.2571029,6.25738,6.2580934,6.25883,6.2590198336227,6.25991,6.26113,6.2614686,6.261841,6.2652257,6.2688271,6.2694443441768,6.2704663,6.2718296729599,6.2720913,6.2737904,6.2755409,6.2764274,6.2777733507036,6.2799102974036,6.2802965,6.2816715,6.2821112,6.2824999,6.283385226219,6.2835189,6.2842542224414,6.2846356737245,6.2848579421744,6.2860633,6.289287,6.2906563,6.2925201670437,6.2963178,6.2969811,6.2983946,6.2984479,6.2990071641892,6.2995311,6.2996916253281,6.3000347,6.3005244,6.3041981,6.3045159,6.305882,6.3060407058991,6.3082216,6.3101191,6.31091,6.3122741,6.3166117689327,6.3172843,6.3174116,6.31944,6.3270556,6.3274197,6.3278187,6.3281938,6.3285912,6.3335848,6.3350740483776,6.3373862,6.3383518297463,6.3389375473376,6.3394275175713,6.340918,6.3411533,6.3414139,6.341803,6.3419205,6.3420366,6.3421677,6.3422514,6.3422919,6.3424931,6.3426968,6.3428723,6.3430442,6.3462055,6.3469173573342,6.3475535,6.3482223,6.3485571,6.3487288,6.3488911,6.3490702,6.3492494,6.3496116,6.3504605,6.351174,6.3514574,6.3518107,6.3521066,6.3521933,6.3522796,6.3524601,6.3527479,6.3529328,6.3531043,6.353194,6.3532775,6.3533854,6.3534779,6.3535894,6.353893,6.3541948,6.3544120183621,6.3545351,6.355044,6.35751706323197,6.35833219625528,6.35965327391375,6.36005381341659,6.360496],"lat":[52.697755,52.6977836256406,52.6978458303382,52.6978991,52.6979896,52.698069,52.6981434,52.6983367064322,52.6983471014579,52.6983724001926,52.6984468,52.6985543,52.6986764,52.6988782,52.6990395878391,52.69908835938,52.6991641302094,52.6992247,52.6993263,52.6993972,52.69944,52.69996,52.7006828924135,52.7008633458543,52.7010156028646,52.7012327091644,52.7014836488667,52.7017796994693,52.7022251813536,52.702721408985,52.7029807984559,52.7035658907168,52.7039027463802,52.7045765736751,52.7066628348879,52.70682,52.7069332,52.7070354,52.7072773,52.7074096,52.7075367,52.7076586,52.7079474,52.7082359,52.7084991,52.7086254,52.7091647762647,52.70999,52.7107305,52.7111885107424,52.7112151,52.7121884,52.7126054,52.71263,52.7127122974118,52.71287,52.7144569,52.7146295,52.7150965,52.71636,52.7165882,52.7168020245507,52.7187169,52.7194896,52.720155,52.7212241409825,52.7216119349994,52.7217264,52.7218658,52.7219515,52.7220113,52.7220332,52.7223463,52.7223865,52.7226296,52.7230352751234,52.72314,52.7234439,52.7236746,52.7237195023346,52.7238286,52.7239727,52.72451,52.7247346,52.7252711,52.72535,52.7257717,52.7258375,52.72594,52.7262282,52.72651,52.7265732780895,52.72687,52.72735,52.7274899,52.7276437,52.7289515,52.7304063,52.7306581768939,52.7310752,52.7316421960591,52.731751,52.7324758,52.7331978,52.7336394,52.7342437654965,52.7352032907483,52.7353767,52.7360173,52.7362789,52.7365609,52.7372541312446,52.7373588,52.7379329365546,52.7382190600672,52.738403439415,52.739326,52.7417931,52.742841,52.7442479259737,52.7471144,52.7476371,52.7487153,52.7487246,52.7491544795384,52.7495572,52.7496583414078,52.7498745,52.7501831,52.7523825,52.7525493,52.7531965,52.7532709358128,52.7542938,52.75501,52.75525,52.7555056,52.7563730139181,52.7565075,52.756533,52.75694,52.758294,52.7583381,52.758371,52.7583889,52.7584002,52.7584446,52.7584578024207,52.7584783,52.7584845825088,52.7584999541912,52.7584967179342,52.7585056,52.7585079,52.7585107,52.758523,52.758526,52.7585345,52.7585471,52.7585582,52.7585635,52.7585933,52.7586367,52.7586847,52.7587349,52.7596959,52.7599121509088,52.7601054,52.7603097,52.7604142,52.7604753,52.7605411,52.760618,52.7607047,52.7608965,52.7613359,52.7617189,52.7618683,52.7620526,52.7622287,52.7622875,52.76236,52.7625191,52.7628138,52.7630021,52.7631733,52.7632247,52.7632737,52.7633319,52.763376,52.7634224,52.7635135,52.7635962,52.763637307453,52.7636606,52.763749,52.7641470214051,52.7642320618212,52.7641385173543,52.7641767855696,52.764274]}]],[[{"lng":[6.357712,6.35818462907003,6.35864841165226,6.35964624690493,6.36023651564595,6.36054570403411,6.360496],"lat":[52.770038,52.7692151402006,52.7676505490224,52.7662219601482,52.7654056026044,52.7646997811413,52.764274]}]],[[{"lng":[6.464505,6.46199803179825,6.46055538343448,6.45858103734902,6.45673682706956,6.45291083437286,6.44917779582331,6.44690227830108,6.44287922281243,6.43982289045413,6.43856986855055,6.43503017462707,6.43258362147408,6.42611401283244,6.42593554086992,6.42152951429501,6.42099781657331,6.41614932825796,6.41494371296942,6.41326310198895,6.41060182475604,6.4102541762457,6.40755199918805,6.40665778029247,6.40041869793575,6.39429116055562,6.3804595834597,6.37788661266658,6.37240603615061,6.3622321861513,6.36162786339264,6.36145921518092,6.3613889450927,6.36131867500449,6.36113597277513,6.36048948796353,6.36008192145188,6.35985705716958,6.35980084109901,6.35936516655207,6.3579457107701,6.357712],"lat":[52.817696,52.817460777467,52.8171641596313,52.8167596775021,52.8164181118837,52.8150046308865,52.8136832430109,52.8127393699344,52.8121415730522,52.8117774972389,52.8118157028687,52.8118067133117,52.8127775747189,52.8130652331939,52.8130292759886,52.8106088382235,52.8103324008234,52.8076443487671,52.8069931004436,52.8060693112837,52.8045644594211,52.8043677796387,52.8028313989506,52.8023183233749,52.7987260628252,52.7949626381063,52.7866073009469,52.7851141438306,52.7817498550879,52.7754984481988,52.7752944037295,52.7751498716517,52.774877809968,52.7744527101832,52.7737300310216,52.7730328468133,52.7724886952827,52.771706465541,52.771349355986,52.771068767852,52.7704225579985,52.770038]}]],[[{"lng":[6.516947,6.51551232957792,6.51500638494276,6.51438800816646,6.51360098317843,6.51306693050799,6.51216747337881,6.51101504393206,6.5098788,6.5089026,6.5086395,6.50852320925198,6.5073642,6.5072531,6.5070451,6.5065929,6.5065297,6.5064563,6.50607572797867,6.5058748,6.5056733,6.5054195,6.5052804,6.5051754,6.50462006453518,6.5041928,6.5039756,6.5032003,6.5028878,6.5027635,6.5027145,6.5025132,6.5015935,6.50136486400605,6.5006476,6.4996347,6.4993255,6.4983201,6.4981239,6.4977007,6.4970756,6.4970161,6.4960893,6.4958893,6.4956133,6.49467058219267,6.4943476,6.4933141,6.48993,6.488,6.4874489,6.48450500537303,6.4804581921412,6.47650414433937,6.47503106770732,6.47156158458709,6.464505],"lat":[52.859378,52.8593137079225,52.8592288533058,52.8591609694931,52.8591949114127,52.8591185420562,52.8589573174176,52.8587876066247,52.8583898,52.8575442,52.8573436,52.8572555513681,52.8560873,52.8558992,52.8555671,52.8548817,52.8547712,52.8546922,52.854105276921,52.8537954,52.8535018,52.8531029,52.852897,52.8527733,52.8520692648357,52.8516941,52.8515261,52.8508577,52.8505932,52.8504808,52.8504446,52.8502667,52.8494869,52.8492915492166,52.8486787,52.8478745,52.8475291,52.8463183,52.8461096,52.8457468,52.8452562,52.8452098,52.8444977,52.8443801,52.8442433,52.8438714867432,52.8437441,52.8433526,52.84207,52.84133,52.8411168,52.8399946538251,52.8384335030266,52.8365367111341,52.8348740229488,52.8287730513863,52.817696]}]],[[{"lng":[6.542643,6.542833,6.542842,6.5428559,6.54294261855093,6.54319874433521,6.54286144791177,6.54229928720604,6.5410030513225,6.54083,6.54059838047592,6.540301,6.53973026011733,6.539651,6.5392867,6.53919,6.5384723,6.5382375,6.5380419,6.5372,6.5371,6.5368558,6.5367589,6.5363151,6.5360973,6.5350489,6.5347927,6.5345189,6.5330162,6.53292656520764,6.5327557,6.5327036,6.5326668,6.5326073,6.5325597,6.5323412,6.532173,6.5319896,6.5308449,6.529053,6.5278313,6.5276457,6.5274506,6.5272912,6.5272323,6.5270361,6.5269143,6.5267391,6.5267248,6.526428,6.5251129,6.5249897,6.5246,6.5244749,6.5243498,6.5242028,6.5240594,6.52338980901417,6.5230487,6.5228758,6.5227395,6.5225024,6.52206,6.52187905039812,6.5217977,6.5215881,6.52137,6.5213207,6.5211937,6.521097200849,6.52089683729718,6.5206914,6.52035102061129,6.52022529879039,6.52016397107287,6.52010870053363,6.5201001,6.5198272,6.5197271,6.5196388,6.5194062198192,6.51901407636946,6.5188300203268,6.5183421,6.5181265,6.5179147,6.51775138547507,6.517522,6.51733935187155,6.51733935187155,6.51771881034791,6.51797178266549,6.51808421480664,6.51805610677135,6.51798583668314,6.51759232418912,6.5173252978539,6.51729718981862,6.51719881169511,6.51698800143046,6.516947],"lat":[52.919006,52.9168878,52.9167949,52.9166936,52.9161971810968,52.9144849587841,52.9132815401156,52.912213689918,52.9110452743393,52.91065,52.9101105803924,52.909418,52.907997036541,52.9077997,52.9068635,52.90662,52.904815,52.9040535,52.9033823,52.90037,52.90005,52.8993674,52.89911,52.89825,52.89789,52.8963725,52.8959581,52.8953347,52.8915789,52.8913668398659,52.8909626,52.8908206,52.890728,52.890571,52.8904537,52.8899454,52.8895634,52.8892388,52.88753,52.8848624,52.883027,52.8827146,52.8823782,52.8820869,52.8819761,52.8816103,52.8813854,52.8810715,52.8810479,52.8805117,52.8780667,52.8778612,52.87737,52.8772323,52.8770868,52.8769178,52.8767618,52.876110386822,52.8759268,52.8758269,52.8757476,52.8756096,52.87532,52.8752090020046,52.8751591,52.8750204,52.87485,52.8748073,52.874697,52.8745944485334,52.8743670036555,52.8741674,52.8736581558539,52.8733657296257,52.8730714506083,52.8728308551062,52.8728102,52.8721705,52.8719615,52.8717847,52.8713926414397,52.8707532394816,52.8704690195671,52.8697163,52.8694288,52.8691761,52.8689784634648,52.868668,52.8681376821618,52.8674250431663,52.8662712218777,52.8653718987118,52.8645574012674,52.8631065398196,52.8625974541297,52.8619441187433,52.8613756241045,52.8609089438347,52.8604931699121,52.8599501122586,52.859378]}]],[[{"lng":[6.542643,6.54398576932324,6.54409820146438,6.54449171395839,6.54763981391049,6.55092845403902,6.55239007187393,6.56245274850653,6.57015435017505,6.57181272425696,6.574427,6.5746756,6.5761993,6.57777907155765,6.5788883,6.579427,6.5795285,6.5799928,6.58197,6.5834928,6.5842394,6.5850002,6.5855726,6.5862508,6.586439,6.5866251,6.5868088,6.5870302,6.5873051,6.5875301,6.5886546,6.589031,6.58931,6.5894817,6.5896026,6.5897551,6.5901389,6.5903365,6.5905259,6.5910043,6.591527,6.5918009,6.5928771,6.593302,6.5934982,6.5936683,6.594,6.5943433,6.5946979,6.5958102,6.5962373,6.5965412,6.5986967,6.6008605,6.601576,6.6018741,6.60257,6.60305992362125,6.6056508,6.6059901,6.6069874,6.60743654983707,6.6078644,6.6086922,6.6102047,6.61112206800554,6.61178,6.6119087,6.61203,6.6126359,6.6127061,6.6127525,6.61281,6.61320489526391,6.6133264,6.6133739,6.6134681,6.6143358,6.6146701,6.6148487,6.6151723,6.6158244,6.6161332,6.6168938,6.6171339,6.6174937,6.6180537,6.6184826,6.6190464,6.6193955,6.6196236,6.6197647,6.6198584,6.620211,6.6203358,6.62042996387914,6.6205523,6.6206972,6.6208525,6.6211372,6.6213233,6.6224053,6.6226141,6.6228997,6.623163,6.6235056,6.623857,6.6241493,6.624421,6.6245937,6.6246082,6.6250563,6.6255936,6.6261318,6.62660327817419,6.6268462,6.6271168,6.6273293,6.6274205,6.6276768,6.62832,6.62838475442213,6.6286912,6.63002,6.6304678,6.6309367,6.63116981193951,6.63181956989835,6.6320434,6.6347322734962,6.6349,6.6350778,6.6353322,6.6361366,6.6363647,6.63902699206831,6.6397076,6.6401866,6.6403368,6.6403663,6.6401902,6.6401729,6.6399249,6.6398929,6.6397081,6.6397127,6.6397385,6.6399576,6.6401825,6.64051857997444,6.640624,6.6406937,6.6407159,6.64071480307212,6.6407188749932,6.6408004948051,6.6409238,6.6411123,6.6417985,6.6420624,6.6423613,6.6426391,6.6428877,6.6430159,6.64319742333961,6.64331448127145,6.6433739,6.64439,6.64449,6.6448084,6.6449924,6.6452207130663,6.645564,6.64588130596986,6.645967],"lat":[52.919006,52.9189254538114,52.9184339710461,52.9183322842603,52.9183492320745,52.918213649375,52.918501762104,52.9203151335136,52.921687822116,52.9222978919709,52.9233,52.9234595,52.9244976,52.9255788335689,52.926338,52.926701,52.9267744,52.92711,52.92858,52.9297,52.9302441,52.9307709,52.9311364,52.9315366,52.9316451,52.9317486,52.9318304,52.93191,52.931989,52.932043,52.9323422,52.9324683,52.93258,52.9326571,52.9327173,52.93279,52.9330032,52.9331073,52.9331883,52.9333206,52.9334618,52.9335635,52.934058,52.9343353,52.9344604,52.9345582,52.9346986,52.9348016,52.9348642,52.9350042,52.9350767,52.9351495,52.9359144,52.9366954,52.9369777,52.9371164,52.9374045,52.937585185979,52.9385407,52.9386663,52.9390241,52.9391643762443,52.9393005,52.9395669,52.9400656,52.9403646348748,52.9405791,52.9406423,52.94075,52.9415973,52.9417618,52.94193,52.942246,52.9442285962024,52.9448386,52.9450039,52.9451469,52.9461075,52.9464772,52.9466528,52.9469085,52.9473551,52.9475381,52.9479142,52.9480292,52.948193,52.9484081,52.9486183,52.9488891,52.9490733,52.94924,52.9493867,52.9495618,52.9511456,52.9514618,52.9516101573706,52.9518029,52.9519739,52.9521253,52.95236,52.9524922,52.9531577,52.9532739,52.9534094,52.9535215,52.9536526,52.9537809,52.9538881,52.9539825,52.9540441,52.9540493,52.9542018,52.9544087,52.9546535,52.9548595409668,52.9549657,52.9550847,52.9551579,52.9551789,52.9552767,52.9555791,52.9556143207517,52.955781,52.9566121,52.9569185,52.9573417,52.9575701169958,52.9582067822677,52.9584261,52.9610480531222,52.9612116,52.961391,52.9617076,52.962906,52.9634326,52.9708306039639,52.9726819,52.9739485,52.9744104,52.9745687,52.9758681,52.9759955,52.9763075,52.9763769,52.9769719,52.9770744,52.9771667,52.9775708,52.9780331,52.9786919401211,52.9788986,52.9790349,52.9791532,52.9791828856451,52.9794609370095,52.9797283918628,52.979905,52.9801944,52.9812612,52.9816575,52.982172,52.9826858,52.9831329,52.9833862,52.9838283187426,52.9841332330067,52.9842569,52.98629,52.98644,52.9867279,52.9869068,52.9870872222111,52.9873585,52.9875050230193,52.987559]}]],[[{"lng":[6.645967,6.64594949399472,6.64583072225333,6.6458081,6.6457441,6.6456615,6.6453375,6.6450441,6.6448663,6.6447145,6.644606,6.64279541609277,6.64177883171688,6.6403925802952,6.63984388889552,6.639621],"lat":[52.987559,52.9875810337131,52.9877243589367,52.9877503,52.9878027,52.9878609,52.9880657,52.9882544,52.9883627,52.9884553,52.9885206,52.9889616424345,52.9895596824426,52.9905610332181,52.9913261777127,52.991676]}]],[[{"lng":[6.639621,6.6399225,6.6401825,6.6402108,6.6402202,6.6402183,6.6402489,6.6403202,6.6403977,6.6404994,6.640622,6.640836,6.6410861,6.6412785,6.6414283,6.6416177,6.6419713,6.6427273,6.6434781,6.6441159445736,6.64475879188532,6.6449511,6.6450824124717,6.64516461885613,6.64520414788461,6.6453034,6.6453324,6.6454659,6.6467095,6.6473906,6.6481064,6.648213,6.6483095,6.6486401,6.6491417,6.6501412,6.6503034,6.6503863,6.6504439,6.6505369,6.6506138,6.6506714,6.6506948,6.6507108,6.6509063,6.6510943,6.6511212,6.651328,6.6514095,6.651507,6.6515673,6.6516034,6.6516182,6.6516465,6.6517301,6.6518842,6.6520085,6.6520349,6.6520147,6.6519691,6.651971,6.6520917,6.6521671,6.6522734,6.6523111,6.6524407,6.6526871,6.6527951,6.6529001,6.6529922,6.6530729,6.6531195,6.6532153,6.6536402,6.6539763,6.6541114,6.6548862,6.6551474,6.65539,6.6559198,6.6561854,6.6564112,6.6566864,6.657087,6.6574387,6.6576209,6.6576595,6.6577228,6.657771,6.6578155,6.6578638,6.6578608,6.6578594,6.6578583,6.6578655,6.6578829,6.6578679,6.6578708,6.6578599,6.6578605,6.6578606,6.6578617,6.6579927,6.6580187,6.6580518,6.6582073,6.6582892,6.65829,6.65828024280397,6.65834286386608,6.65847874262698,6.6585743,6.6586669,6.6606749,6.6606885,6.6608806,6.6611324,6.6614691,6.6615466,6.661626,6.6616598,6.6616836,6.6616863,6.6616398,6.6615729,6.6616413,6.661707,6.6617409,6.6617611,6.6617995,6.6618225,6.6618915,6.6619753,6.6620899,6.6622031,6.66247,6.6627378,6.66291279479804,6.6629313,6.6631494,6.6633256,6.6635226,6.6637444,6.6639911,6.6641692,6.66369765357046,6.6636123,6.6631992,6.6630221,6.6628749,6.6624639,6.66203369134708,6.66194594147866,6.6617629,6.6615291,6.6610259,6.6604483,6.6603223,6.66021,6.6600662,6.6599594,6.65981802661359,6.6597658,6.6597432,6.6597471,6.6597809,6.6598224,6.6599406,6.6602383,6.6603473,6.6604916,6.6606039,6.6608565,6.6610449,6.6612299,6.6613016,6.6614372,6.66154162761209,6.6615719,6.6616744,6.6618565,6.6618992,6.6618871,6.6618236,6.6617356,6.6616052,6.6613891,6.661142,6.6609085,6.6608685,6.6608379,6.6608026,6.6607789,6.6607917,6.6608184,6.6610694,6.6613646,6.6613982,6.661407,6.6614645,6.6615642,6.6615925,6.6616381,6.66254291996469,6.6634256,6.6654773,6.6659183,6.6666949,6.6669449,6.66788,6.66801950942033,6.66855679840439,6.6692174,6.6709935,6.6712972,6.67161873962318,6.6722578,6.6723113,6.6723301,6.6722789,6.67203,6.67199,6.67198597124704,6.671983,6.6714715,6.6713619,6.6712317,6.6709578777447,6.67072857641901,6.67058542731759,6.6704233,6.6702229,6.6701225,6.669984,6.6699035,6.66985127129867,6.66949198590669,6.66947,6.6692106921176,6.6691089,6.6687104,6.668636,6.6686096,6.66859,6.66855,6.6682518,6.6682031,6.66817599466996,6.6681718,6.6681609,6.6682082,6.66840878896366,6.668757],"lat":[52.991676,52.99226,52.9928446,52.9929253,52.9930048,52.9932307,52.9933609,52.9935078,52.9936482,52.9938348,52.9940199,52.9942965,52.994599,52.994817,52.9949459,52.99506,52.9952241,52.9955391,52.9958684,52.9961458397401,52.9964254536902,52.9965091,52.9965703507791,52.9966486343798,52.9967555204465,52.9973257,52.9974528,52.9975946,52.998459,52.9989407,52.9994181,52.9994943,52.9995447,52.9997852,53.0001791,53.0009619,53.0010822,53.0011511,53.0012142,53.0013377,53.0014618,53.0015796,53.0017443,53.0018084,53.0023227,53.0027148,53.0027398,53.003126,53.0033345,53.0037877,53.0040482,53.004311,53.0044978,53.0046703,53.0049513,53.0053866,53.0058028,53.0059909,53.0062334,53.0066573,53.0067704,53.0072653,53.0075173,53.0077577,53.0077962,53.0081422,53.0084804,53.0086608,53.0088418,53.0090308,53.009224,53.0094391,53.0096643,53.0104823,53.0109474,53.0111099,53.0118412,53.0120869,53.0123467,53.012997,53.0133238,53.0136334,53.0140323,53.0147824,53.0155168,53.015982,53.0161035,53.0163872,53.0169133,53.0175784,53.0182532,53.0190657,53.019829,53.0201937,53.0215869,53.0222831,53.0229783,53.0238206,53.0246676,53.0247794,53.024788,53.024983,53.0257077,53.0257883,53.025999,53.026961,53.0276584,53.027966,53.0281775259084,53.0287566544504,53.0291002936954,53.0291803,53.0292583,53.0309276,53.0309398,53.0311123,53.0313037,53.0316779,53.0317937,53.0319445,53.0320319,53.0321367,53.0324248,53.0327889,53.0332845,53.0337144,53.034134,53.0344086,53.0349174,53.035066,53.0351113,53.0352125,53.0353338,53.0354477,53.0355395,53.03574,53.0359322,53.0360561885812,53.0360693,53.0362047,53.0363052,53.0363879,53.0364674,53.0365381,53.0365898,53.0371997882973,53.0373102,53.03778,53.037963,53.0381048,53.0385217,53.0389905401235,53.0391132795094,53.0393696,53.0396974,53.0404187,53.0412511,53.0415036,53.04174,53.04211,53.0423548,53.0427310173868,53.04287,53.0430234,53.0431682,53.0432911,53.0434238,53.0436695,53.0443116,53.0445529,53.0447561,53.0448606,53.0450448,53.0452071,53.0454876,53.0458332,53.0466095,53.0471990869076,53.04737,53.048014,53.049062,53.0492966,53.049414,53.0495521,53.0496612,53.0498232,53.049977,53.050142,53.0503056,53.0503427,53.0503783,53.0504359,53.0505236,53.0506149,53.0506886,53.051382,53.0521432,53.0522315,53.0522546,53.0523542,53.0524787,53.0525094,53.0525587,53.0533076188732,53.0540382,53.0557301,53.0560712,53.0567379,53.0569881,53.0580746,53.0582389805485,53.0588720491996,53.0596504,53.0617376,53.0620626,53.0622832202834,53.0627217,53.0628037,53.0629151,53.0631412,53.0637,53.06393,53.0644831503997,53.0648911,53.0662718,53.0665329,53.0667841,53.0672385022611,53.0675588835127,53.0676798466385,53.0677781,53.0679721,53.0680898,53.0682789,53.0683944,53.0684806171436,53.0690737070571,53.06911,53.0695532873974,53.0697273,53.070394,53.0706112,53.0707948,53.07132,53.07146,53.0718613,53.0719682,53.0720650171382,53.07208,53.0722458,53.0723968,53.0731096181933,53.074117]}]],[[{"lng":[6.668757,6.66878074604458,6.66898788703345,6.6690136,6.66905,6.66911058055095,6.6691309,6.6692434,6.66935449597283,6.66936,6.66939707382736,6.6694264,6.6694667,6.66951,6.66995102708854,6.67019530706998,6.67029301906256,6.67035583391493,6.67040880006243,6.6705065,6.670574,6.6705989,6.6705892,6.6705663,6.6704664,6.6704187,6.6703627,6.670355,6.67033463578274,6.67032449459015,6.6702961,6.67027671116011,6.6702659,6.670226,6.6702377,6.6703432,6.67042799073778,6.67130957545454,6.67293565635285,6.6734202,6.67368717282559,6.67385703675541,6.673927,6.674138,6.6750251,6.6755118,6.6758288,6.67642428769855,6.67666635721543,6.6768082,6.6771434,6.67734255859115,6.67808055927211,6.6787547,6.6787547,6.6790345,6.6803001,6.6813777,6.6816834,6.6817828,6.6818985,6.6823743,6.6826908,6.6832306,6.6833418,6.6834122,6.6836006,6.68426,6.6844976,6.6851294,6.685215,6.6853137,6.6861848,6.6867238,6.6868097,6.6868894,6.6869601,6.68720186126386,6.68729],"lat":[53.074117,53.074270179051,53.0751088403616,53.0751948,53.0753626,53.0755875494864,53.075663,53.0760916,53.0768332564506,53.07687,53.0771754132599,53.077417,53.0775738,53.07778,53.0784267726888,53.0790766348841,53.0792569174969,53.0795587843675,53.079891356963,53.0800666,53.0802091,53.0803502,53.080459,53.0805586,53.0808352,53.0810206,53.081539,53.0817491,53.0820617914205,53.08221750802,53.0826535,53.0829277691098,53.0830807,53.0835247,53.0837934,53.083994,53.0841344430328,53.0854395499912,53.0870341211337,53.0875679,53.0878540438076,53.0880769637298,53.0881534,53.0883981,53.0893714,53.0899263,53.0900904,53.0903978727616,53.0905228617564,53.0905961,53.090785,53.0909152377869,53.0913755126427,53.0916903,53.0916903,53.0917635,53.0920092,53.0922157,53.0922711,53.0922882,53.0923068,53.0923675,53.0924016,53.092517,53.0925427,53.0925631,53.0926298,53.09287,53.0929674,53.0932084,53.0932447,53.0932825,53.0936108,53.0938102,53.0938434,53.0938742,53.0939077,53.0940293577422,53.094072]}]],[[{"lng":[6.688743,6.6884877,6.6882822,6.6882822,6.6881036,6.6878234,6.6875128,6.6873762,6.6872917,6.6871883,6.6871019,6.6870236,6.6869256,6.6869034,6.6868707,6.6868649,6.6868684,6.68687411889067,6.6868813,6.6869223,6.6869416,6.68696111773965,6.6869713,6.6870352,6.6871422,6.6873051,6.6873167,6.6873266,6.6873325,6.687339,6.6873417,6.68730973801558,6.68729],"lat":[53.096955,53.0968393,53.0968013,53.0968013,53.096774,53.0967088,53.0966195,53.0965753,53.0965473,53.0964942,53.0964465,53.0963933,53.0962755,53.0962201,53.096136,53.0960708,53.0960435,53.0959675142035,53.0958721,53.0955863,53.0954992,53.0952502013336,53.0951203,53.0949396,53.0947321,53.0944874,53.0944627,53.094436,53.0944135,53.0943867,53.0943554,53.0941240392776,53.094072]}]],[[{"lng":[6.688743,6.68932082973743,6.68995591745417,6.6901085,6.6901575,6.6907821,6.6910211,6.6923662,6.6951338,6.69682,6.69791,6.6983451,6.6990083,6.69938,6.6998087,6.70017,6.7007499,6.7010355,6.7013359,6.7017944,6.7021044,6.7055183,6.7055474,6.7056785,6.7057491,6.7061313,6.7065858,6.7069425,6.7072698,6.7078216,6.708224,6.7085969,6.7089076,6.7091383,6.7093597,6.7095231,6.7096883,6.7099293,6.7099969,6.7101641,6.7103416,6.7104919,6.7107041,6.7109082,6.7112403,6.711953,6.7127547,6.7131929,6.713533,6.7136634,6.7139254,6.7144565,6.71501614918101,6.7154814,6.7178008,6.7179159,6.7179844,6.7180613,6.7181123,6.7185128,6.7197011,6.719887,6.7199406,6.7200583,6.7203609,6.7205231,6.7206757,6.7208249,6.720947,6.72108,6.7211532,6.7212053,6.7212418,6.7212551,6.7212582,6.721259,6.7212593,6.72126561949252,6.72127,6.7213,6.72145,6.72175325354844,6.72218,6.72228,6.7224422,6.7225635,6.7226491,6.7226644,6.7228481,6.72293,6.7233,6.72355,6.72422573941766,6.73468636552553,6.73558961425517,6.73602,6.73603,6.7364395,6.73667,6.73684,6.7368748,6.7369419,6.7368748,6.73695,6.73729,6.7379192,6.7379192,6.7381708,6.738279,6.7385369,6.73937448066354,6.74007292598542,6.74030759883221,6.74038143009358,6.7403392,6.74013394261797,6.73994194579187,6.73963711906951,6.73957,6.739565,6.7395538,6.73955,6.73939320165699,6.7393709,6.7393671,6.73931268461167,6.73934707741282,6.7393961949854,6.7397192235962,6.7399504,6.7404092,6.7410899,6.7414873,6.74243090517027,6.7426423,6.74293581347751,6.74301452283232,6.743031],"lat":[53.096955,53.0969919434127,53.097053771915,53.0970771,53.0970845,53.0972263,53.0972998,53.0977357,53.0986709,53.09924,53.0996181,53.0998017,53.1001396,53.10035,53.1006736,53.10098,53.1016147,53.1019636,53.1023317,53.102868,53.1031898,53.1066419,53.1066674,53.1068016,53.1068753,53.1072625,53.1076685,53.1079612,53.108191,53.108583,53.1088307,53.1090688,53.1092719,53.1094402,53.1096271,53.1097799,53.1099591,53.1102599,53.1103536,53.110571,53.110762,53.1109053,53.1110709,53.111205,53.1113997,53.111701,53.1120005,53.1121853,53.1123514,53.1124224,53.1125805,53.1129628,53.1134232881015,53.1138061,53.115697,53.1158184,53.115883,53.1159647,53.1160156,53.1163211,53.1174031,53.1175781,53.1176246,53.1177365,53.1180242,53.1181578,53.1183097,53.1184828,53.1186542,53.1188844,53.1190602,53.1192473,53.119504,53.1197429,53.1200479,53.1201306,53.1201552,53.1204060902665,53.12058,53.12647,53.12729,53.1285528902043,53.13033,53.13095,53.1314256,53.1317228,53.1319241,53.1319566,53.1324105,53.13259,53.13353,53.13433,53.1355434191393,53.1493998014676,53.151087606957,53.15223,53.1524,53.1535261,53.15416,53.15461,53.1546568,53.1546513,53.1546568,53.15476,53.15501,53.1552116,53.1552116,53.1552407,53.155247,53.1552553,53.1553046449737,53.1553440383029,53.1554487607695,53.1555771369508,53.155858,53.1572379361117,53.158469239475,53.1603014614718,53.16075,53.1607898,53.1608798,53.16091,53.1619146141713,53.1620575,53.1621011,53.1626123889348,53.1628310508722,53.1628801309992,53.1629314469544,53.1629428,53.1629612,53.1630038,53.1630355,53.1631311676996,53.1631526,53.1632195598456,53.1632523201359,53.163259]}]],[[{"lng":[6.795179,6.79411907928396,6.78909220271687,6.78664399870938,6.78095710476096,6.77526253619183,6.7727913083222,6.77069613686753,6.76713511285664,6.76464853574559,6.76041214511195,6.75329009709018,6.74954488218217,6.74512430065142,6.74363542423307,6.74329390361134,6.743031],"lat":[53.164275,53.1642524488004,53.1641374208423,53.1641052129588,53.1639671788986,53.1638337455517,53.163672704753,53.1636681035785,53.1637049129611,53.1637279188092,53.1636957106184,53.1634978597732,53.1632954068014,53.1631803762783,53.1631435664458,53.1631803762783,53.163259]}]],[[{"lng":[6.862903,6.86240003907905,6.84953781069524,6.8483710426777,6.83606441811177,6.81745863526067,6.81102057602105,6.80870787512914,6.80717996462998,6.80374216600688,6.79860977573926,6.795179],"lat":[53.162237,53.1620208409615,53.1611380781271,53.1611713902618,53.1622040536143,53.1637363469974,53.1642568146402,53.1644733273207,53.1644858184036,53.1644441814466,53.1643484162922,53.164275]}]],[[{"lng":[6.971194,6.97081223404194,6.9701177292696,6.96815922581159,6.96633962330805,6.96503395433604,6.96463114156808,6.96195035314683,6.94847696056336,6.92478045773099,6.91444622671851,6.91272385488309,6.91155708686556,6.91000139617551,6.90797344224026,6.9069177949863,6.90411199570603,6.9020007011981,6.9002227689809,6.89897266039068,6.89716694798259,6.88591597067062,6.86452522368242,6.862903],"lat":[53.17594,53.1761092768104,53.1763174046028,53.1766670570211,53.1760010499593,53.1754848873746,53.1754266105636,53.175268430249,53.1745024962657,53.173003890227,53.1722712193345,53.1720547459946,53.1717217079544,53.1710223196559,53.1700897841911,53.1692904519476,53.1670922115005,53.1656599639598,53.1649438222645,53.1646273837552,53.1643275977521,53.163611433824,53.1621790701169,53.162237]}]],[[{"lng":[7.038785,7.03833198800922,7.03812363657752,7.03747080209152,7.03677629731917,7.03602623216504,7.03520671653367,7.03484557405206,7.03452610185678,7.03383159708443,7.03335933383924,7.03288707059404,7.03226201629894,7.03123414923587,7.02808109756942,7.02549753981631,7.02473358456673,7.02431688170332,7.02359459674008,7.02046932526453,7.01967758982406,7.01921921667431,7.01367706859101,7.0097322814841,7.00639865857685,7.00555136275459,7.00514854998663,7.00370398006015,7.00335672767398,7.00314837624228,7.00264833280619,7.00043980763014,6.99967585238056,6.99887022684464,6.99777290930434,6.9952726921239,6.99432816563351,6.985702416361,6.97941020312356,6.97584044859371,6.97493759238967,6.97442365885813,6.97352080265408,6.97300686912255,6.97211790301395,6.97143728833705,6.971194],"lat":[53.143014,53.1429957810531,53.1432207324888,53.1437206203503,53.1441038637707,53.1445037663026,53.1448286843681,53.1449953080635,53.1452702357478,53.1460200295781,53.1463865906865,53.1465865317906,53.1467448178378,53.1469364264829,53.1474529324851,53.1478444732183,53.1479694322753,53.1481610354565,53.14865253536,53.1507767499069,53.1512598899198,53.1514014988934,53.1525843320771,53.1534672708192,53.1543335326936,53.1544917901863,53.1545167781582,53.1545917419864,53.1546583763905,53.1547749863489,53.1552330938333,53.1575901877879,53.1585146663366,53.1601803431984,53.1620374966896,53.1650021133144,53.165751562617,53.1698150154119,53.1725959272964,53.174469194478,53.1749021157022,53.1750602973674,53.1751851772175,53.1752934061269,53.1756180912168,53.1758595221266,53.17594]}]],[[{"lng":[7.039675,7.03938594651784,7.0393019,7.0389161,7.0389145,7.03888877374126,7.038785],"lat":[53.142255,53.1424320561141,53.1425012,53.1424851,53.1425525,53.1427929643076,53.143014]}]],[[{"lng":[7.050126,7.0459306,7.04506,7.0445279,7.0442867,7.04349380824088,7.0427268,7.0427034,7.0426661,7.0426144,7.0425544,7.0424655,7.0424391,7.0423946,7.0423538,7.0423314,7.0423146,7.0422469,7.042223,7.0421562,7.042152,7.0420476,7.0420304,7.0419353,7.0419265,7.0418294,7.0417214,7.0415806,7.0415314,7.0412965,7.0412581,7.04102320018284,7.040963,7.0409119,7.0407922,7.0406871,7.0405733,7.0404568,7.0404204,7.0403157,7.0401464,7.0401048,7.0400812,7.0399939,7.0399161,7.0397886,7.0397119,7.0396665,7.0397465,7.0398192,7.0398549,7.0399437,7.0399557,7.03996092929181,7.0399614,7.0399165,7.0397532,7.0397197,7.0396617,7.03965271305184,7.039675],"lat":[53.128703,53.1316954,53.13247,53.1330136,53.13326,53.134370100918,53.135939,53.13598,53.1360458,53.136137,53.1362431,53.1364,53.1364466,53.136525,53.1365971,53.1366367,53.1366663,53.1367859,53.1368281,53.136946,53.1369535,53.1371376,53.137168,53.1373359,53.1373531,53.1375229,53.1377135,53.1379648,53.1380618,53.1385049,53.1386058,53.1388830014116,53.1389882,53.1390334,53.1391473,53.1392611,53.1393731,53.1394863,53.1395292,53.1396284,53.1398662,53.139932,53.1399748,53.140131,53.1402716,53.1404939,53.1406912,53.1408092,53.1408298,53.1408899,53.1409468,53.1411198,53.1411455,53.1412387098418,53.1412471,53.1413939,53.1418251,53.1418703,53.1421345,53.1421848718806,53.142255]}]],[[{"lng":[7.140995,7.14027007164233,7.13900717372854,7.13838827011516,7.13811227255785,7.1378111843135,7.13713373576372,7.13877299398294,7.13827118024236,7.13669883052189,7.12114260456399,7.11997170583598,7.11870044435985,7.1160241044101,7.11194268598674,7.11006924802192,7.10645618908976,7.10240051232425,7.09847295231186,7.08861235544172,7.08685447744815,7.08506913261094,7.08314645355548,7.07941096281916,7.07850455697873,7.07529093627175,7.07383519355833,7.07207731556477,7.07119837656799,7.06891862854508,7.06702341633327,7.0659247425873,7.06430419881198,7.06232658606922,7.06142018022879,7.05850869480195,7.05760228896152,7.05581694412431,7.05496547197118,7.05419640034899,7.05144971598405,7.05092784595472,7.050126],"lat":[53.138163,53.1383316078349,53.1384369684094,53.1385824659207,53.1387179286779,53.138898545023,53.1394855429001,53.1426762628485,53.1445624889509,53.1456460281866,53.1506018684312,53.1500401015175,53.1488162267237,53.1475722526644,53.146288112612,53.145264786017,53.1439003126272,53.1429349989028,53.1427691272828,53.1435434569571,53.1435434569571,53.143296331982,53.1427526520317,53.1423078178627,53.1420442102553,53.1403471975536,53.1395728102578,53.1389137462439,53.1382711490947,53.137249564609,53.1366234201746,53.1359807887659,53.1356017965059,53.1346954971106,53.1339704438294,53.1330146730823,53.132437904587,53.1317952105722,53.1314161813905,53.1310701083481,53.1300153923615,53.1291089751273,53.128703]}]],[[{"lng":[7.190474,7.1882620642018,7.18828953104545,7.1859823161789,7.18488364243293,7.18334549918856,7.17947267423399,7.17867613576816,7.17820919942612,7.17702812514919,7.17684959066547,7.17662985591628,7.17621785326153,7.17613545273059,7.17417157340965,7.17369090364579,7.17266089700894,7.17117768745187,7.1708068850626,7.17062835057888,7.17005154686224,7.16881553889802,7.16814260122861,7.16795033332306,7.16782673252664,7.16738726302825,7.16709886116993,7.165917786893,7.16546458397279,7.16501138105257,7.16426977627404,7.16328096990266,7.16306123515347,7.15162129477348,7.14786418624972,7.14543038960793,7.14251986991258,7.142018056172,7.140995],"lat":[53.159927,53.1596858744692,53.1549096688319,53.1548602570311,53.154662609259,53.1547449626079,53.153443761225,53.1531472794443,53.1528343242325,53.1525048952294,53.1519036807832,53.1515577727663,53.1511789179314,53.1510883217147,53.147917333708,53.1474396061286,53.1457016213077,53.1433128070245,53.1425467108595,53.1422419161125,53.1414840387181,53.1396304778779,53.1387407402647,53.1384523954177,53.1381475716179,53.1374637698205,53.137093029356,53.1355276454553,53.1350580191628,53.134489517309,53.1335831944508,53.1323802273116,53.1318940872001,53.1365575097033,53.1368866367414,53.1370973647196,53.1376091283634,53.1377646631648,53.138163]}]],[[{"lng":[7.190474,7.20126074795889,7.204543035775,7.20749572146731,7.20911626524263,7.20930853314817,7.21124494562546,7.21140974668735,7.21155394761651,7.21175994894388,7.21254962069881,7.21718465056465,7.21742498544658,7.21763785348486,7.21901806237825,7.21934766450204,7.21958799938397,7.21990386808594,7.22075534023907,7.22133214395571,7.22224541650705,7.22289775404373,7.22346769104945,7.22412689529704,7.22441529715536,7.22458009821725,7.22479983296645,7.2251088349575,7.22768385154964,7.2278280524788,7.22790358629883,7.22802718709526,7.22837738935179,7.22854905712459,7.22864519107737,7.2287756585847,7.23065713737469,7.23103480647487,7.23146054255143,7.23187941191709,7.23898645771137,7.24730891133715,7.24870972036327,7.25013799623304,7.25122293655719,7.25211560897579,7.25309068192535,7.25449149095147,7.25542536363555,7.25883125224808,7.26086379867813,7.26126206791105,7.2614543358166,7.26185260504951,7.26212727348601,7.26220967401696,7.26237447507885,7.26260794324987,7.26291007853002,7.26341821513753,7.2660687655497,7.26642583451714,7.26741464088852,7.27012012498799,7.27058706133003,7.27083426292287,7.27101279740659,7.27331314556223,7.27408221718442,7.2747963551193,7.27548302621054,7.27581262833433,7.27589502886528,7.275917],"lat":[53.159927,53.1592865052194,53.1584959901812,53.1577054605853,53.1570796143308,53.15693138625,53.1562314133977,53.1562314133977,53.1562602360346,53.156342586319,53.1567378654853,53.158990063786,53.1590765260822,53.1590971123176,53.1590682915853,53.1591012295634,53.1591629882043,53.1593235602546,53.1599617252866,53.1603775567506,53.1610898131661,53.1617732388468,53.1622878594363,53.1627654218237,53.1629547985405,53.1630453697183,53.1631112395458,53.163193576688,53.1639057863756,53.163975771661,53.1640581071446,53.1641116251243,53.16418984359,53.1642968791541,53.1644368483352,53.1645356498352,53.165466019472,53.1656306845636,53.1656553842728,53.1656389178016,53.1650461206287,53.1679770934706,53.1683640327919,53.1686768747987,53.1688579928128,53.1689485515333,53.1690308774769,53.1693519471478,53.1694178072962,53.1703233740861,53.1707926148144,53.171278314725,53.1717804732587,53.1730317279723,53.1752871870569,53.1762502484241,53.1770239743731,53.1772626742428,53.1774026011351,53.1774519869881,53.1776906844762,53.1777729936469,53.1782092296149,53.1802175040175,53.1804068037227,53.1804685317068,53.1804891076818,53.1809746978257,53.1812956781233,53.1816660370203,53.1821022333977,53.1822750646975,53.1823491350414,53.182414]}]],[[{"lng":[7.297466,7.2846138,7.2834403,7.2822939,7.2817428,7.2811938,7.2802578,7.2801368,7.2798158,7.2790586,7.278631,7.2782065,7.2780642,7.2777226,7.27749114169081,7.2772368,7.2763801,7.275917],"lat":[53.173999,53.1808937,53.1811635,53.181431,53.1815604,53.1816835,53.1819189,53.1819477,53.1820242,53.1821837,53.1822493,53.1823002,53.1823092,53.1823268,53.1823316121224,53.1823369,53.182348,53.182414]}]],[[{"lng":[7.30937,7.30844056821306,7.30450365395673,7.30400925077105,7.3030021331706,7.30173865836275,7.30150061238446,7.30146398992626,7.30128087763527,7.30036531618031,7.29974273439094,7.29820459114661,7.297466],"lat":[53.160188,53.160452732306,53.1618525289398,53.1621654184155,53.1630766274037,53.1642732459685,53.1658814926789,53.1674622361743,53.1682086781407,53.1696137101492,53.1711065062513,53.1737187744647,53.173999]}]],[[{"lng":[7.30937,7.31005195637379,7.31091258414145,7.31585661599823,7.31724826940977,7.31790747365734,7.31917094846518,7.32015975483654,7.32290643920142,7.32406004663467,7.32468262842404,7.32550663373351,7.32601934814828,7.32918719078244,7.33035910944479,7.33167751793993,7.332428278333,7.33308748258057,7.33788502460456,7.33858085131033,7.33911187695421,7.34015561701286,7.34054015282394,7.34096131109323,7.341474025508,7.34200505115188,7.34275581154495,7.34447706708027,7.34535600607703,7.34583209803361,7.34667441457217,7.34742517496524,7.3483590476493,7.34890838452228,7.34932954279156,7.34971407860264,7.35008945879917,7.35144448975251,7.3517924031054,7.35268965333126,7.35396228375365,7.354946],"lat":[53.160188,53.160008081469,53.1599202486601,53.1595908740262,53.1595798948282,53.1596677283335,53.1600739559577,53.1604143058722,53.1617537212631,53.1623026499222,53.1625112409719,53.1626539605797,53.1626868958065,53.1626539605797,53.1626759174004,53.1627527661841,53.1628076580882,53.1628296148302,53.162599068479,53.1625441763081,53.162423413285,53.1620940578587,53.161995250738,53.1619403577948,53.1619513363891,53.1620281864702,53.1621379720616,53.1626210253277,53.1629833117097,53.1634443990459,53.1641469987951,53.1645861177977,53.1650362101126,53.1652777211161,53.1653655429622,53.1653874983956,53.1653874983956,53.1652557656264,53.1652118546135,53.165222832371,53.1653216320616,53.165115]}]],[[{"lng":[7.354946,7.35539971523794,7.35553704945618,7.35578425104903,7.35600398579821,7.35613216440191,7.35634274353655,7.35661741197304,7.3574602,7.3576318,7.3577257,7.3578276,7.3579462,7.357998,7.3580778,7.358116,7.3581327,7.3581589,7.3581965,7.3583312,7.3586397,7.3590347,7.3593867,7.359911,7.36044698246713,7.36116836860521,7.3618543,7.3622414,7.3624541,7.3632677,7.3643944,7.3648435,7.3654552,7.3657248,7.3664915,7.3668903,7.3670243,7.3675565,7.3678908,7.3686298,7.3691934,7.3696181,7.3699943,7.3707362,7.3712336,7.3718921,7.3727131,7.3731585,7.3734506,7.3741283,7.3744503,7.3746589,7.3749103,7.3752597,7.3754676,7.3755798,7.3760257,7.3765015,7.3767269,7.37737242718821,7.37983223563053,7.3848861348619,7.38630220324556,7.38710789732592,7.38770606414316,7.3884507207932,7.38913434001289,7.39089221800641,7.39302852806798,7.39500614081069,7.39636117176402,7.39732556316324,7.40074365926175,7.40142727848145,7.40212310518722,7.40247712228314,7.40274568697659,7.40311191155858,7.40316074150284,7.40274568697659,7.40214752015935,7.40178129557737,7.40158597580031,7.40156156082818,7.40163480574458,7.40172025814704,7.40173246563311,7.40147610842572,7.40107326138554,7.40073145177569,7.40082911166422,7.40093897903881,7.40161039077245,7.40214752015935,7.40289217680939,7.40338047625203,7.40396643558321,7.4040396804996,7.40401526552747,7.40386877569468,7.40378332329222,7.40381994575041,7.40368566340369,7.4033560612799,7.40328281636351,7.4033560612799,7.40355138105696,7.40375890832008,7.40412513290206,7.40421668904756,7.40408240670084,7.40411902915904,7.40430214145003,7.40441811256765,7.40535198525171,7.4055534087718,7.40731739050835,7.409282795765,7.4091821,7.40908747598794,7.41091859889785,7.41173650046428,7.41213934750446,7.41301828650122,7.41588704572675,7.41578938583823,7.41541095377018,7.41840178785637,7.41864593757769,7.41990636051402,7.41956201924611,7.4196181,7.4197756,7.4199681,7.4201733,7.4203912,7.4209498,7.4214299,7.4218175,7.4224424,7.4230808,7.42508],"lat":[53.165115,53.1657003621019,53.1661065326338,53.1666114960427,53.1669078848416,53.1671713398332,53.1674951677113,53.1678464358668,53.1686172,53.1687957,53.1689066,53.1690545,53.1692469,53.1693152,53.1694255,53.1694941,53.1695315,53.1696052,53.1696651,53.1698096,53.1701505,53.1705966,53.1709745,53.1715825,53.1721500272466,53.172545677841,53.1727579,53.1728735,53.172937,53.173194,53.1735513,53.1736953,53.1738914,53.1739796,53.1742515,53.1744137,53.1744685,53.1747107,53.1748739,53.1752934,53.1756685,53.175954,53.1762277,53.1767738,53.1771509,53.1776272,53.1782341,53.178553,53.178784,53.1793763,53.1796917,53.1799133,53.1802129,53.1806375,53.1809175,53.1810749,53.1816901,53.1823368,53.182619,53.1824446947399,53.1817021582718,53.1798439318578,53.1809340035702,53.1816436326834,53.1821849913286,53.1829311771799,53.1835676195592,53.1847965850128,53.1859011608155,53.1867862639391,53.187137374167,53.1872909839877,53.187568943217,53.1876274607199,53.1877518101485,53.1878907884948,53.1880517102279,53.1883150353975,53.1884978991473,53.1892659183808,53.189712094666,53.1901143807792,53.1903191795324,53.1904874063479,53.1907872872952,53.1910067110497,53.1912919602519,53.1917015455312,53.1925499597344,53.193427611933,53.1943418138687,53.1946855487536,53.1955997238653,53.1962286650225,53.1968649100601,53.1978741069578,53.1993951704323,53.1997023016995,53.2001044940787,53.2004116202634,53.2010112412338,53.2016766644418,53.2019472181766,53.2023055164401,53.202539505526,53.2027003722816,53.2030294160372,53.2031537207986,53.2032780251994,53.2033840492564,53.2058956459017,53.2061990748318,53.2067401234965,53.2069850554431,53.2080342257003,53.2083303283374,53.209214968472,53.209902196963,53.2114349,53.2121831316708,53.2121904421637,53.2122343050952,53.2123439622274,53.2126217590397,53.2136671361635,53.2140180262633,53.2144785651578,53.2145224257467,53.2150999193124,53.214931799014,53.2170306297379,53.2171343,53.2170745,53.2170307,53.216999,53.2169781,53.2169385,53.2168994,53.2168593,53.2167589,53.2166457,53.216324]}]],[[{"lng":[7.42508,7.4269904,7.4274632,7.4276728,7.427704,7.4278307,7.428082],"lat":[53.216324,53.2159643,53.21587,53.2157947,53.2157835,53.2157269,53.215579]}]],[[{"lng":[7.428082,7.42876265939298,7.42910711158323,7.42969630611917,7.43023773809473,7.4306303,7.4319573,7.4328282,7.4340513,7.4343843,7.4396255,7.4404349,7.4406833,7.4409602,7.440988,7.4413555,7.44187,7.4420843,7.4422111,7.4424884,7.442911,7.4433904,7.44383,7.4442173,7.4443483,7.4446071,7.4447668,7.4453762,7.4459033,7.4460925,7.4463576,7.4464004,7.446497,7.4465105,7.4465823,7.4466209,7.4466533,7.4466543,7.4466583,7.4466906,7.4466937,7.4472988,7.4473427,7.4473588,7.4473564,7.4473538,7.4473225,7.447136,7.4472562,7.4475356,7.4479366,7.4482981,7.4483866,7.4484478,7.4484705,7.4486718,7.4495815,7.4498453,7.4500294,7.4503621,7.45062276741825,7.45072562603188,7.45088436793194,7.451264],"lat":[53.215579,53.2152624112875,53.2151647083545,53.2150235815024,53.2150021017042,53.2150158,53.2151666,53.2152656,53.2154046,53.2154423,53.2160391,53.2161704,53.2162222,53.2162816,53.216297,53.2163997,53.2165679,53.2166455,53.2166962,53.2168158,53.2170257,53.2173031,53.2175718,53.2178662,53.217997,53.2183047,53.218506,53.2193231,53.2202046,53.2205395,53.2210145,53.2211095,53.2213239,53.2213721,53.2216576,53.2219821,53.2222328,53.2222408,53.2222714,53.2223816,53.2223924,53.2234291,53.2235168,53.2235707,53.2236023,53.2236362,53.223686,53.2238925,53.2239395,53.2240289,53.2241639,53.2242852,53.2243229,53.2243589,53.2243802,53.2245699,53.2253584,53.2255871,53.2257468,53.226063,53.2262849729148,53.2263725623753,53.2264106954142,53.226324]}]],[[{"lng":[7.643661,7.6430153351482,7.64264911056621,7.64189835017312,7.64140394698744,7.64081798765625,7.64041514061606,7.63992073743037,7.63902348720449,7.638382594186,7.63796143591671,7.63774170116751,7.63766845625111,7.63779663485481,7.6382910380405,7.6384741503315,7.63863895139339,7.6383276604987,7.63785156854211,7.63766845625111,7.63733885412732,7.63651484881784,7.63539786384276,7.6333836286418,7.63173561802284,7.63076512288056,7.62995942880018,7.6287142652214,7.62697469845694,7.62545486644167,7.6241547691756,7.62283636068043,7.62193911045455,7.62052914581388,7.61899100256951,7.61776415021983,7.61668378770296,7.61589640485167,7.61540200166598,7.61521888937499,7.61525551183319,7.61529213429139,7.61525551183319,7.61510902200039,7.6147244861893,7.61419346054541,7.61351594506873,7.61294829696664,7.61238064886455,7.61183131199156,7.61120873020217,7.61060445964188,7.6098720104779,7.60888320410652,7.60776621913145,7.60631963203258,7.6050012235374,7.60371943750043,7.60232778408886,7.6006797734699,7.59976421201491,7.59914163022553,7.59850073720704,7.59815282385415,7.59789646664676,7.59767673189756,7.59765842066846,7.59787815541765,7.59785984418856,7.59785984418856,7.59758517575206,7.59690766027538,7.5956441854675,7.59406941976494,7.59242140914597,7.5909381995889,7.58982121461383,7.58934512265724,7.58930850019904,7.58943667880273,7.59143260277459,7.59364826149564,7.59485680261622,7.5957174303839,7.59606534373679,7.59604703250769,7.5957174303839,7.59483849138712,7.59397786361944,7.59187207227298,7.59020575042492,7.58777035695467,7.58577443298281,7.58432784588394,7.58280801386867,7.58048248777302,7.57942043648525,7.57888941084136,7.57850487503027,7.57841331888477,7.57868798732126,7.57896265575776,7.57923732419425,7.57899927821596,7.57841331888477,7.57772664779353,7.5771132216187,7.57642655052746,7.57561170083253,7.57442147094105,7.57312137367498,7.57149167428511,7.57020988824814,7.56911121450216,7.56847032148368,7.56784773969429,7.5674815151123,7.56704204561391,7.56662088734462,7.56594337186793,7.56493625426745,7.56330655487759,7.56228112604801,7.56160361057132,7.56123738598933,7.56079791649094,7.56046831436715,7.56012040101426,7.55936964062117,7.55830758933339,7.55671451240173,7.55576232848855,7.55459040982617,7.55402276172409,7.55372978205849,7.5535649809966,7.5535100473093,7.55372978205849,7.55383964943309,7.55380302697489,7.55409600664048,7.55391289434949,7.5532536901019,7.55252124093792,7.55149581210834,7.55076336294435,7.54962806674018,7.54801667857941,7.54618555566945,7.54505025946527,7.5438783408029,7.54292615688972,7.54153450347815,7.53984987040098,7.53867795173861,7.53787225765823,7.53732292078524,7.53648060424666,7.53582139999907,7.53479597116949,7.53373391988171,7.53234226647014,7.53080412322578,7.52857015327562,7.52564035661969,7.5250177748303,7.5247980400811,7.5249445299139,7.52542062187049,7.52600658120168,7.52589671382708,7.52534737695409,7.52384585616792,7.52175837605057,7.5202202328062,7.51831586497984,7.51644811961168,7.51490997636731,7.51311547591555,7.51194355725317,7.51102799579819,7.50956309747022,7.50846442372425,7.50648681098149,7.50454582069693,7.50282456516157,7.5013962892918,7.50022437062942,7.49949192146544,7.49886933967605,7.49791715576287,7.49626914514391,7.49520709385613,7.49282663407318,7.48978697004264,7.48553876489154,7.48136380465682,7.47964254912146,7.47803116096069,7.47715222196391,7.47704235458931,7.47726208933851,7.4775916914623,7.4775184465459,7.47682261984012,7.47561407871954,7.47436891514077,7.47253779223081,7.47008408753146,7.46711766841732,7.46422449421959,7.46213701410223,7.45924383990449,7.45737609453633,7.45631404324855,7.45550834916817,7.45543510425177,7.45565483900097,7.45576470637556,7.45627742079035,7.45770569666012,7.45993966661028,7.46202714672763,7.46301595309901,7.46297933064081,7.46217363656043,7.46092847298165,7.45935370727909,7.45726622716173,7.45528861441897,7.45433643050579,7.45268841988683,7.451264],"lat":[53.216849,53.2167798147984,53.2168127084772,53.2169223538908,53.2171087504498,53.2174047903764,53.2175473273892,53.2175582917551,53.2174925055173,53.2174376835753,53.2175363630204,53.2177008282571,53.2179530070603,53.2182161485763,53.2187753189307,53.2190932752884,53.2197401447306,53.2203212225103,53.2210557812518,53.2214175742103,53.2217026210555,53.2219218865685,53.2220863349666,53.2223494510902,53.2225577502083,53.2227770113434,53.2230181972955,53.2235882677828,53.224509134848,53.225320358476,53.2261973397062,53.2266906337556,53.2268769878065,53.2269866074575,53.2271291125844,53.2271839221223,53.2274250832557,53.2276991283494,53.2280499035102,53.2284883684203,53.2289487517436,53.2293981687845,53.22986950354,53.2302202609209,53.2306696646158,53.2310532981842,53.2315026931353,53.2317547906767,53.2317986335758,53.2317767121319,53.2315246147195,53.2311081026991,53.2305490933558,53.2300010378946,53.2297270075332,53.2296283561738,53.2298037362112,53.2301654552696,53.2306806256226,53.2315355755075,53.2320616900279,53.2324124294497,53.2327631659981,53.2328070078646,53.2327631659981,53.2325220349297,53.2319849653963,53.2312725158228,53.2301654552696,53.2290364432439,53.2280718268624,53.2274141214154,53.2269646835498,53.2267344818405,53.2264165822075,53.2258794360855,53.2252984337152,53.2248489736437,53.2242679572925,53.2239500393485,53.2227331592062,53.2210886716471,53.2203870044037,53.2199155652684,53.2194989403181,53.2192358066826,53.2190823113153,53.2190603833608,53.2191590590677,53.2199923115278,53.2203431498193,53.2205076042792,53.2205514586952,53.2203870044037,53.2201129125146,53.2193454458939,53.2191261671907,53.2191590590677,53.2194112292859,53.2197511085353,53.2203760407618,53.2209900003855,53.2215272078368,53.2223055985152,53.2232593818893,53.2238294491662,53.2242350693387,53.2244707657823,53.2246187605606,53.2246735733117,53.2245639477394,53.2241583306818,53.2235444164763,53.2231168638844,53.2230181972955,53.2230730120951,53.2232703447931,53.2238842629275,53.2245200974319,53.2250134108024,53.2254847938248,53.2260109826975,53.2264275443031,53.2268769878065,53.2274031595724,53.228652791604,53.2294639367363,53.2297927749799,53.2300668049203,53.2302202609209,53.230768313576,53.2311519462605,53.2314040458672,53.2313602025639,53.2312177115184,53.230943688944,53.2306148595397,53.2298256586654,53.2287075991916,53.2275018160587,53.2266906337556,53.2260109826976,53.2253751703291,53.2250682230484,53.2248489736437,53.2248708986347,53.2252216969638,53.2260109826976,53.2268221778757,53.2271729602204,53.2273921977266,53.2273921977266,53.2270633410463,53.2266687096963,53.2261644532342,53.2255505677875,53.2249366735402,53.223840411924,53.2233141963803,53.2228976044892,53.2226125655973,53.2224810085458,53.2224810085458,53.222656417858,53.2230730120951,53.2233141963803,53.2236869330584,53.2245200974319,53.2255944170398,53.2268002538837,53.2275018160587,53.228225290013,53.2292775939439,53.2301983386688,53.2308121574853,53.2312505941094,53.2312944375249,53.2308998451693,53.2298037362112,53.2286199070178,53.2279622099894,53.2276333576869,53.2275018160587,53.2280060567723,53.2286199070178,53.2287514452112,53.2283787526137,53.2278087458959,53.2266687096963,53.2253532455963,53.2239938902397,53.2227002700738,53.2221301877661,53.2216258778607,53.2212531232444,53.2211434889163,53.2202664041886,53.2195427957669,53.2184902526033,53.2172841817644,53.216297371274,53.2148500081001,53.2133148725086,53.2121963819385,53.2113410459175,53.2108585411417,53.2107488802076,53.2109462696869,53.2113191140001,53.211648091582,53.2117138867952,53.211648091582,53.2119112718288,53.2125472840843,53.2129859077251,53.2139508639277,53.2156833443847,53.2182051843788,53.2198497826509,53.2209680734077,53.2220863349666,53.2236650074612,53.2248708986347,53.2259232850011,53.2268879497842,53.2278087458959,53.2285322146644,53.2288172141563,53.2288829830005,53.2284664452814,53.2279183631617,53.226624861544,53.226324]}]],[[{"lng":[7.643661,7.6438752976672,7.64418486685515,7.6451423,7.6453451,7.645883,7.646372,7.6466342,7.6467217,7.64755957183642,7.64754126060732,7.64754736435035,7.64821267234097,7.65039170860382,7.65213127536829,7.65221062402772,7.65232049140232,7.65250970743634,7.65277216838677,7.65395629453521,7.65416992554138,7.65437745280451,7.65443849023484,7.65443849023484,7.65426148168688,7.65427979291597,7.65436524531844,7.6545788763246,7.65492678967749,7.65570196504271,7.65677012007352,7.65747815426537,7.65781996387523,7.65818008471419,7.65896136382244,7.6594496632651,7.65971212421553,7.66014548997088,7.66053002578198,7.66095118405127,7.66151272841032,7.66200102785298,7.66277620321819,7.66348423741005,7.66362462349981,7.66372838713137,7.66390539567934,7.664067],"lat":[53.216849,53.2168379951789,53.2168263083315,53.2167212,53.2166989,53.2165888,53.2164689,53.2164147,53.2163979,53.2162553399521,53.2160799044239,53.2159848765462,53.2154878042121,53.2137662811837,53.2122493829677,53.2121945543156,53.2121506913434,53.2121031730729,53.21207758629,53.2120227574181,53.2120264126784,53.2119971705872,53.2119459968795,53.2118875125673,53.2106227697864,53.2105313851912,53.2104326896096,53.2103230275855,53.2101695002803,53.2098734103525,53.2096540831609,53.2094859315537,53.2093762671065,53.2092117699095,53.2089339509872,53.2087986963863,53.2087584854766,53.2087548299375,53.2087877297783,53.2088462183217,53.2088973957318,53.2089083623117,53.2088937402045,53.2087365522373,53.2086853746352,53.2085537748062,53.2083344408597,53.208149]}]],[[{"lng":[7.664067,7.66548626512494,7.66615157311556,7.66660325010001,7.66773244256116,7.6687517676477,7.66930110452069,7.66960018792932,7.66971005530391,7.66976498899121,7.66973447027605,7.66955746172808,7.66972226278998,7.67075379536259,7.6717670167061,7.672318],"lat":[53.208149,53.2083344408597,53.2084441079733,53.2084660413623,53.2084660413623,53.2084148634371,53.2084294857077,53.2084623857982,53.2085099081067,53.2086634413584,53.2088242851273,53.2093287457589,53.2093726116201,53.2094237884014,53.2094822760766,53.209537]}]],[[{"lng":[7.672318,7.67374462944886,7.67398267542716,7.67409254280176,7.67415968397512,7.67425734386365,7.674564,7.6746897,7.6751265,7.6752744,7.6754055,7.6756521,7.6759289,7.6762703,7.6771057,7.6779597,7.6783233,7.6794253,7.6797829,7.6801472,7.6803976,7.68184429645392,7.68236921835478,7.68263778304824,7.6829795926581,7.68373645679421,7.68488],"lat":[53.209537,53.2096760159306,53.2097527805361,53.2098807212396,53.2100342495795,53.2101512231836,53.2102269,53.2102476,53.210328,53.2103551,53.2103778,53.2104124,53.2104329,53.2104474,53.210525,53.2106042,53.210638,53.210745,53.210751,53.2107347,53.2107064,53.2105240744152,53.2098734103525,53.209420132919,53.2086451636192,53.2072706552921,53.206121]}]],[[{"lng":[7.68488,7.6858117294255,7.68637327378456,7.68686157322721,7.68742311758627,7.68777713468219,7.68830205658305,7.68826543412485,7.68839971647158,7.68853399881831,7.68880256351177,7.68946176775936,7.68973033245282,7.68993785971595,7.69070693133813,7.69112198586439,7.69141496552998,7.69178119011198,7.6924648093317,7.69328271089814,7.69353906810554,7.69381984028507,7.69399074509,7.69414944240886,7.69485747660071,7.69597140970427,7.69637425674447,7.69675879255556,7.69699378666233,7.69714027649513,7.69778727325665,7.69798259303372,7.69805583795011,7.69803142297798,7.69808025292225,7.69817791281078,7.69795817806158,7.69798259303372,7.69820232778291,7.6987150421977,7.69974047102728,7.70096121963392,7.7019483,7.7022635,7.7025357,7.70331116070113,7.704183232987,7.70496603803101,7.70555657516947,7.70589304400418,7.7064080473226,7.70737625356125,7.70778825621599,7.70809039149613,7.7085867,7.7087745,7.7090175,7.7093056,7.71080775364943,7.7118497,7.7128017,7.7132864,7.71363,7.7140976,7.7144716,7.715232,7.7157502,7.7167755,7.7172709,7.7177424,7.7182263,7.7186914,7.72308728812871,7.72505116744965,7.7260811740865,7.72664424438131,7.72944586243355,7.7331397,7.7334621,7.7336589,7.7338742,7.7340168,7.7344024,7.7347851,7.7350764,7.7351316,7.7355667,7.7358293,7.7359192,7.7360402,7.7369385,7.7373737,7.7380078,7.73825035351019,7.7386893,7.7390865,7.7392959,7.7398389,7.7403948,7.7408423,7.7414533,7.7417597,7.7422282,7.7428522,7.74293028605114,7.74346569143956,7.74356],"lat":[53.206121,53.2064079094587,53.2064883356519,53.20645177831,53.2062251220946,53.2058668565994,53.205084511331,53.2045141945646,53.2043167754541,53.2042363451853,53.2042363451853,53.2043094636177,53.2043460227871,53.2043021517801,53.2040169891398,53.2039219345047,53.2039219345047,53.2039584940047,53.204112043564,53.2042217214839,53.2043021517801,53.2044337646664,53.2046458067997,53.2047554833536,53.2049309652559,53.2047170965917,53.2047554833536,53.2049529004432,53.2052526802106,53.2053477318941,53.2054208484303,53.2056401972903,53.2058449218799,53.206195876044,53.2064590897809,53.2068685301581,53.207351079866,53.2076873991763,53.2082138067105,53.208740207778,53.2093250902684,53.209763746897,53.2099739,53.2099811,53.2099956,53.2099197887722,53.2097594060472,53.2094715381121,53.2092206801915,53.2091918931231,53.2092659169743,53.2094962125823,53.2106641212527,53.2111575942221,53.2114896,53.2115223,53.211561,53.2115954,53.2117115998806,53.2117922,53.2118704,53.2118965,53.2119011,53.2119023,53.2119119,53.2119339,53.2119433,53.2119691,53.2119838,53.2119956,53.2120331,53.2120875,53.2124405973438,53.2125968579198,53.2128189114423,53.2149078030541,53.2155739242356,53.2154351,53.2154576,53.2154708,53.2154721,53.2154729,53.2154718,53.2154696,53.2154679,53.2154676,53.2154916,53.2155245,53.2155332,53.2155538,53.2156941,53.2157621,53.2158647,53.2159039214512,53.2159749,53.2160417,53.216072,53.2161504,53.2161948,53.2162058,53.2161905,53.2161765,53.2161236,53.2160472,53.2160360818351,53.2159598488664,53.215949]}]],[[{"lng":[7.74356,7.74357566589104,7.74366280855212,7.7442584,7.7447736,7.7457383,7.7468962,7.7469995,7.7476383,7.7490797,7.7498311,7.7502592,7.75095,7.7513058,7.7516725,7.7518072,7.7521308,7.7525792,7.7535649,7.7538018,7.7554169,7.7558543,7.7572012,7.7581483,7.7586035,7.7587994,7.7590972,7.7593973,7.7600538,7.7604435,7.7605946,7.7608382,7.7610666,7.7611519,7.7618747,7.7620484,7.7620908,7.7623551,7.7624474,7.7638715,7.7641563,7.7646492,7.7663167,7.7670282,7.7677326,7.7677785,7.7679915,7.7683425,7.7683961,7.7685457,7.7686471,7.76889186263934,7.76904762243537,7.7691970113554,7.7705949480566,7.77128480601631,7.7715518,7.7718013,7.7723259,7.7730705,7.7734345,7.7736187,7.7738039,7.7739974,7.7741305,7.7743237,7.7746591,7.7754033,7.7757358,7.7760936,7.7763043,7.7765177,7.7768909,7.7770363,7.7773089,7.7775814,7.7779145,7.7782764,7.7790016,7.7792744,7.7795895,7.7808609,7.7828008,7.7838448,7.7851721,7.7853374,7.7858732,7.7864992,7.7867037,7.7874579,7.788065,7.7887639,7.7894301,7.7896858,7.789981,7.7902127,7.7903976,7.7907831,7.7911221,7.7913783,7.7917343,7.7921202,7.7924774,7.7928,7.7931914,7.7935246,7.7935674,7.7939779,7.7948866,7.7951287,7.795923,7.7966387,7.797488],"lat":[53.215949,53.2159467438311,53.2159336241161,53.2158615,53.2158359,53.2158642,53.2159934,53.2160062,53.2160925,53.2162872,53.2163967,53.2164609,53.2165353,53.2165589,53.2165728,53.2165747,53.2165845,53.2165855,53.2166015,53.2166078,53.2166384,53.2166479,53.2166688,53.2166954,53.2166996,53.2167014,53.2167032,53.2167104,53.2167205,53.216719,53.2167173,53.2166993,53.2166788,53.2166698,53.2165933,53.2165749,53.21657,53.2165391,53.2165295,53.216354,53.2163231,53.2162653,53.2160657,53.2159916,53.2159132,53.2159081,53.2158844,53.2158438,53.2158374,53.2158182,53.2158052,53.2157748377193,53.2157674662276,53.2157556546777,53.2157126905711,53.2157212841526,53.215728,53.2157342,53.2157473,53.2157654,53.2157948,53.2158164,53.2158491,53.2158955,53.2159344,53.2160088,53.2161411,53.2164389,53.2165764,53.2167072,53.2167738,53.2168324,53.2169225,53.2169505,53.2169931,53.2170305,53.2170643,53.2170804,53.2171234,53.2171459,53.2171911,53.2173641,53.2176327,53.2177725,53.2179359,53.2179587,53.2180326,53.218122,53.2181497,53.2182505,53.2183316,53.2184294,53.2184965,53.2185217,53.2185446,53.2185607,53.218567,53.2185686,53.2185554,53.21854,53.2185096,53.2184647,53.2184099,53.2183499,53.218264,53.2181834,53.218173,53.2180727,53.2178454,53.2177784,53.2175763,53.2173944,53.217324]}]],[[{"lng":[7.797488,7.7983626,7.799278,7.80079880620014,7.8026369,7.8026369,7.8027458,7.8028526,7.8029795,7.8031618,7.8038258,7.8039267,7.8040367,7.8042188,7.8044713,7.8047491,7.8052583,7.8055038,7.8056557,7.80588653067517,7.805967],"lat":[53.217324,53.217554,53.2177716,53.2179746812274,53.2183726,53.2183726,53.2183992,53.2184129,53.2184161,53.218376,53.2181766,53.2181507,53.2181307,53.218108,53.2180947,53.2180872,53.2180886,53.2180947,53.2181004,53.2181742765558,53.218204]}]],[[{"lng":[7.805967,7.80611134251392,7.80631026420164,7.8063467,7.8067567,7.8070452,7.8072011,7.8073902,7.8082223,7.8090473,7.809184,7.8097353,7.8100918,7.8102748,7.8104827,7.8106714,7.8109017,7.8113702,7.8123082,7.8124404,7.81263682810519,7.81281536258891,7.81298016365081,7.8132548320873,7.81340589972737,7.81359816763292,7.815864182234,7.81611825053776,7.81635171870877,7.81831559802971,7.81964773994671,7.82177642032954,7.82437203705441,7.82690585338106,7.82921993495853,7.83036667568089,7.83099841308483,7.83172628444154,7.83191168563617,7.8317606179961,7.83239235540004,7.83267389054744,7.83417083352634,7.83419830036998,7.83418456694816,7.8341125,7.8348461,7.8351893,7.8360941,7.8366048,7.8371999,7.8377857,7.8383961,7.8396122,7.8405115,7.8410038,7.841464,7.8435858,7.8440235,7.8444503,7.844941,7.8453324,7.8456809,7.8460475,7.8464072,7.8467802,7.8472833,7.8477666,7.8480236,7.8482714,7.8487936,7.8492439,7.8496027,7.8511441,7.8535592,7.8542512,7.8549243,7.8556469,7.8571154,7.8578738,7.8582583,7.8586403,7.8594343,7.8597465,7.8607339,7.8617324,7.8619876,7.8622465,7.8627665,7.8633236,7.8638194,7.8642335,7.8648825,7.8658626,7.86843572097899,7.86987773027058,7.87032406647988,7.8723360127772,7.87251454726093,7.87289908307202,7.87374368851424,7.87392908970887,7.87412822432533,7.87436855920726,7.8747256281747,7.87383905949854,7.8768239,7.8770417,7.8771183,7.8774236,7.8777201,7.8778776,7.8779883,7.8781374,7.8782522,7.8783164,7.8786179,7.8788378,7.8789736,7.8790859,7.8791229,7.8791917,7.8793878,7.8795658,7.879778,7.8799329,7.880102,7.8806892,7.8813492,7.8824188,7.8831072,7.883888,7.8845432,7.8861147,7.8880454,7.8883188,7.8886106,7.8886771,7.8891755,7.8896534,7.8905796,7.8919303,7.8932756,7.8937113208789,7.89380058812076,7.8939310556281,7.8959361352145,7.8961558699637,7.89631380431468,7.896441],"lat":[53.218204,53.21813389339,53.2180332838187,53.2180253,53.2179289,53.2178627,53.2178307,53.2178079,53.2177961,53.2177858,53.2177865,53.2177838,53.2177762,53.2177653,53.2177462,53.217711,53.2176599,53.2175406,53.2173065,53.21727,53.2171964661968,53.2173033695207,53.2173321511393,53.2173527094264,53.2173485977698,53.2173074811817,53.2166619455752,53.2165879345019,53.2165509289173,53.2164358002274,53.2164070180066,53.2152803844553,53.213869994853,53.2130763732922,53.2126404919836,53.2123855404834,53.2117563794833,53.2109915046477,53.2105514898188,53.2089887838118,53.2069283922068,53.2062292348188,53.2048144343852,53.2047116127656,53.2046375810468,53.204557,53.2042788,53.2041517,53.2037997,53.2036145,53.203423,53.2032443,53.2030785,53.2027744,53.2025317,53.2023962,53.2022764,53.2016964,53.2015851,53.2014632,53.2013254,53.201237,53.2011618,53.2011012,53.2010594,53.2010295,53.2010204,53.201023,53.201036,53.2010542,53.2011157,53.2011984,53.2012659,53.2016305,53.2021845,53.2023388,53.2024581,53.2025531,53.2027237,53.2028072,53.2028516,53.2028931,53.2029786,53.2030163,53.2031191,53.2032077,53.20323,53.2032519,53.2032913,53.2033361,53.2033596,53.2033744,53.2033573,53.2033098,53.2031486937747,53.2030828854088,53.2030253030057,53.2026839200273,53.2026427893644,53.202511170978,53.2021368789825,53.2020957477946,53.2020834084305,53.2020834084305,53.2021163133935,53.2011040172056,53.2000994,53.2000014,53.1999666,53.1998029,53.1996064,53.199488,53.1993945,53.1992591,53.1991366,53.1990646,53.1987265,53.198479,53.198346,53.198222,53.1981869,53.1981016,53.1979214,53.1977769,53.1976789,53.197616,53.1975604,53.1974419,53.1973173,53.1971261,53.1969983,53.1968979,53.1968594,53.1968537,53.1968541,53.1968412,53.1968234,53.1968184,53.1967549,53.1966735,53.1965103,53.1962796,53.1960464,53.195834663721,53.1957400480199,53.1956865694877,53.1952422837183,53.1951805769973,53.1951641218567,53.195142]}]],[[{"lng":[7.896441,7.8975068953357,7.89781418064903,7.89809228244098,7.89853175193937,7.89935575724885,7.90068789916585,7.90133336999161,7.90160803842811,7.90292644692328,7.90339338326532,7.90368178512364,7.90388778645101,7.9041723,7.9043085,7.9046912,7.9050903,7.9053731,7.9056248,7.9059604,7.9063087,7.9070234,7.9074724,7.9075568,7.9077657,7.9080803,7.908604,7.9098076,7.9103882,7.9107016,7.910966,7.91122250533108,7.911292],"lat":[53.195142,53.1950602486359,53.1949831148932,53.194773310411,53.1945017972039,53.1945429356792,53.1946910338632,53.1947979933445,53.1950365943029,53.1968466272894,53.1973402593857,53.1975294835156,53.1975953004084,53.1976796,53.1977013,53.1977393,53.1977677,53.1977807,53.1977763,53.1977655,53.1977391,53.1976594,53.1976176,53.1976098,53.1975938,53.1975758,53.1975615,53.197523,53.197492,53.197463,53.1974291,53.1974066851011,53.197398]}]],[[{"lng":[7.911292,7.91142922886097,7.9116441,7.9118317,7.9119629,7.9120912,7.9125707,7.9129914,7.9133863,7.9138115,7.9142133,7.9144168,7.9146141,7.9148473,7.9151157,7.9154754,7.9158611,7.91637734934008,7.9174343,7.9227715,7.924548,7.9249977,7.9253926,7.9258179,7.9264802,7.9270423,7.9275487,7.927924,7.9282793,7.930298,7.9306842,7.9313004593816,7.93198713047283,7.93211073126925,7.93216566495655,7.93214506482381,7.93194593020736,7.93176052901272,7.93010565168285,7.92998891759733,7.92884904358588,7.92864304225851,7.9291374454442,7.92953571467712,7.92994771733186,7.93069618882131,7.93153392755261,7.93226179890932,7.93289353631326,7.93366260793545,7.94228719684136,7.94363230921171,7.94497818455053,7.94607685829651,7.94735482949468,7.94868620844321,7.95412464348579,7.95527825091906,7.95604808550972,7.96283239589113,7.96500227653943,7.96593614922351,7.97127845031332,7.97199258824821,7.97295392777594,7.97658641784857,7.97666881837952,7.97690915326146,7.97698468708149,7.97720442183069,7.97743789000171,7.98216218710941,7.98231325474948,7.98424966722676,7.98457926935055,7.98549940861281,7.98558867585467,7.9868575,7.9871753,7.9882112,7.9911718,7.9925769,7.9927878,7.9931549,7.9938731,7.9941885,7.9946081,7.9948304,7.9949928,7.9954448,7.9959974,7.9963206,7.9967293,7.9970782,7.9973681,7.9977109,7.9980396,7.99875751682815,7.9989145,7.9996626,8.000888,8.001153,8.0018684,8.0019161,8.0028246,8.003458,8.004598],"lat":[53.197398,53.1973720099529,53.1973066,53.1972541,53.1972184,53.1971801,53.1970247,53.196845,53.1966477,53.1963557,53.1960574,53.1958828,53.1957143,53.195523,53.1953202,53.1950355,53.1947243,53.1942886472628,53.1939561,53.1929642,53.1929878,53.1930299,53.1931201,53.1932469,53.1934279,53.1935261,53.1935855,53.1935983,53.1935634,53.1931027,53.1930443,53.1929554189981,53.1919845045043,53.1917253159849,53.1914743541794,53.1913632722556,53.1889893416103,53.1885696729555,53.1872653823886,53.1870925711984,53.1841917079791,53.1838460601298,53.1834633753329,53.1832205950356,53.1830436525972,53.1827967549515,53.182607465794,53.1825498558846,53.1825416258912,53.1826156957747,53.1837102691411,53.1836316290465,53.183582250308,53.1837797649208,53.1843604160367,53.1848825381248,53.1859029895603,53.1858865308589,53.185364420994,53.1867963218236,53.1872653823886,53.1875533994914,53.1897669520851,53.1899808953505,53.1901372378309,53.1905116346116,53.1903840932183,53.1901372378309,53.1900179238842,53.1898533523784,53.1897957522022,53.1898903810223,53.1896599800521,53.1894131204958,53.1891210015184,53.1868333531074,53.1865288727135,53.1861993,53.1861456,53.1859893,53.1855458,53.1853237,53.1852893,53.1852122,53.1850656,53.1850042,53.1849195,53.1848725,53.1848406,53.184752,53.1846538,53.1846045,53.184561,53.184543,53.1845475,53.1845728,53.1846133,53.1847153589455,53.1847768,53.1851147,53.1856224,53.1857322,53.1860215,53.1860394,53.1863809,53.186616,53.187145]}]],[[{"lng":[8.212924,8.212002,8.2116998,8.2115339,8.2110547,8.2107956,8.2105956,8.2103624,8.2100181,8.2096144,8.2092756,8.2092307,8.2089584,8.2088636,8.2087341,8.2085733,8.2083535,8.2081699,8.2080892,8.2077654,8.2071914,8.2063345,8.2058406,8.2036674,8.2035715,8.2034891,8.2029471,8.2024564,8.2020894,8.2019769,8.2018164,8.2016825,8.2003427,8.1996314,8.1983374,8.1978905,8.1977726,8.1976496,8.1975194,8.1973186,8.1965389,8.1961656,8.1957159,8.1951243,8.1950744,8.18850379815355,8.18544582289391,8.17922000500004,8.17601553990761,8.17455979719419,8.17317729939717,8.17138279894541,8.17043977064678,8.16958829849365,8.16798606594743,8.16618240988112,8.16479075646955,8.16363714903627,8.16213562825011,8.16033197218379,8.15943472195791,8.15875720648123,8.1576859995789,8.15573585367979,8.15539709594145,8.15526891733775,8.15521398365045,8.15523229487955,8.15582740982529,8.15595558842899,8.15601967773084,8.15600136650174,8.15593727719989,8.15578163175254,8.15567176437794,8.15567176437794,8.15579078736709,8.15563514191975,8.15556189700335,8.1553879403269,8.15471042485022,8.15470126923567,8.15475620292296,8.15493015959941,8.15518651680681,8.15690777234217,8.15769515519345,8.15782333379715,8.15793320117175,8.1579240455572,8.15784164502625,8.1571284,8.1569495,8.15507512349704,8.15456240908225,8.15411378396931,8.15184319156096,8.15105733464484,8.15033404109541,8.14962905877507,8.1485120738,8.14725775460667,8.1456829889041,8.14410822320154,8.14321097297566,8.14247852381167,8.1411051816292,8.13986001805043,8.13918250257374,8.1381029,8.1375147,8.1364358182088,8.13481527443349,8.13377153437481,8.1323974,8.13236,8.1318399,8.12945923992185,8.12907470411076,8.12734429296084,8.12706962452435,8.12643788712041,8.126233,8.1253976,8.1250675,8.1249072,8.1247569,8.1246067,8.1243916,8.1242375,8.1240866,8.124049,8.1236913,8.1233032,8.1229111,8.122535,8.1191199,8.1179109,8.11766265413637,8.1174555,8.1172234,8.1165714,8.116466,8.11619408185004,8.1028972761777,8.09527980487226,8.0943842,8.0939506,8.0937295,8.0932748,8.0929381,8.0926037,8.0921277,8.0917601,8.091375,8.0900196,8.0895799,8.0891511,8.0886471,8.0880358,8.0874332,8.0868182,8.0860001,8.0852157,8.0844272,8.0836215,8.0815913,8.0801022,8.0796001,8.078314,8.0757627,8.0753044,8.0748771,8.074287,8.0711083,8.0698361,8.06845862984154,8.0679366,8.0673545,8.0670777,8.066815,8.0666751,8.0664868,8.0661517,8.0659541,8.0658352,8.0656745,8.0650437,8.0627314,8.0618846,8.0613008,8.0570248,8.055948,8.0542441,8.0538673,8.0535254,8.0532883,8.0503919,8.0501422,8.049907,8.0496864,8.0493761,8.0490941,8.0453551,8.0451137,8.04478659063008,8.04267164366908,8.04211315118154,8.04039189564617,8.03987002561684,8.03961366840944,8.0391467320674,8.03859739519441,8.03779170111403,8.03769098935398,8.03754449952118,8.03729729792834,8.03483443761444,8.0308119,8.03037,8.0299778,8.0293671,8.028941,8.0282689,8.0275292,8.0271506,8.0267491,8.0259907,8.0256527,8.0250239,8.0241682,8.0235992,8.0235157,8.0232317,8.0229742,8.0229184,8.0226366,8.0223939,8.0222947,8.0219822,8.02104292586463,8.0205753,8.0202632,8.0201359,8.0200128,8.0197167,8.0193893,8.0190678,8.0184867,8.0181138,8.0178698,8.0174515,8.0170696,8.0168751,8.0166123191029,8.0164774,8.0162595,8.0160671,8.0158974,8.0154512,8.0153568,8.0150645,8.0147804,8.0146193,8.0142616,8.01298608942649,8.01131824164326,8.01104357320677,8.01068650423933,8.01026076816276,8.01003416670265,8.00953289680605,8.00929256192412,8.00892175953485,8.00801535369442,8.00790777522346,8.00724857097588,8.00699679157576,8.00638794320819,8.00544491490956,8.004598],"lat":[53.140346,53.1405058,53.1405251,53.1405299,53.1405161,53.1405269,53.1405484,53.1405877,53.140661,53.1407741,53.1408909,53.1409081,53.1411018,53.1411618,53.1412126,53.1412426,53.1412632,53.1412757,53.1412776,53.1412886,53.1413161,53.1413708,53.1413998,53.1415177,53.1415228,53.1415268,53.1415545,53.1415818,53.1415997,53.1416058,53.1416165,53.1416255,53.1417029,53.1417401,53.1418109,53.1418557,53.1418725,53.1418872,53.1419066,53.1419402,53.142118,53.1422342,53.1423791,53.1426042,53.142623,53.1474217598803,53.1486847068272,53.1496291472268,53.1495358022984,53.1494808934222,53.1494479480628,53.1494424571671,53.1494699116386,53.149585220227,53.1499640891232,53.150414334769,53.1507163261546,53.1509194828007,53.1509304642137,53.1510018433294,53.1513752090782,53.1518528928757,53.1531431616272,53.1549769236915,53.1553008435813,53.1555369192494,53.1558004440434,53.1559376958996,53.1566019886822,53.1567886476142,53.1569423661252,53.1571015740028,53.1572717610811,53.1575462549471,53.1578811350868,53.1582269921609,53.1593194430299,53.1598519341287,53.1607028082561,53.1610870684626,53.1618336213153,53.1620202574997,53.1622453177014,53.1626076072546,53.1628985345306,53.164117114144,53.1647099241488,53.1648636142949,53.1650776817955,53.165511302435,53.1656485232437,53.1659492,53.1660166,53.1660853398957,53.1661731600892,53.1663103787817,53.1676825415761,53.1680360950817,53.1682611237383,53.1683105201148,53.1683324740416,53.1680360950817,53.1677232484034,53.1675256598525,53.1675585913408,53.1676628742205,53.1679537672296,53.1685684780456,53.1687989923314,53.1690248,53.1692775,53.1702479109167,53.171416889115,53.1718614217618,53.1724962,53.1725527,53.1733957,53.1738535300189,53.1740510894328,53.1754175171391,53.1760595585023,53.1787263966976,53.1803911,53.1810842,53.181354,53.1814817,53.181596,53.1816972,53.1818148,53.1818877,53.1819584,53.1819752,53.1821352,53.1823172,53.1825027,53.1826731,53.184233,53.1847867,53.1849004113404,53.1849953,53.1851097,53.1854325,53.1854797,53.1856089007164,53.1875245978681,53.1887644216655,53.1885226,53.1883914,53.1883214,53.1881565,53.1880129,53.1878672,53.187643,53.1874467,53.1872209,53.186337,53.1860752,53.1858514,53.1856208,53.1853814,53.1851861,53.1850261,53.1848546,53.1847416,53.1846723,53.1846386,53.1845599,53.1845035,53.1844845,53.184453,53.1843591,53.1843289,53.184274,53.184173,53.1835854,53.18335,53.183101482756,53.1830073,53.1829017,53.1828517,53.1828041,53.1827787,53.1827446,53.1826877,53.182649,53.1826244,53.1825922,53.1824657,53.1820299,53.1818745,53.1817873,53.1814343,53.1813454,53.1811822,53.1811726,53.1811849,53.1812125,53.1816428,53.1816713,53.1816851,53.1816895,53.1816706,53.1816439,53.1812623,53.1812288,53.1813468884315,53.182246716753,53.1825320242302,53.1835635046597,53.1838378303851,53.1839091547862,53.1840847220376,53.1843480715671,53.1848253884677,53.1850119246632,53.1851381104531,53.1852368642905,53.1860214005862,53.1812754,53.1813306,53.1813581,53.18137,53.181347,53.1812919,53.1812314,53.1812094,53.1811876,53.1811251,53.1811038,53.1810502,53.1809772,53.1809313,53.1809246,53.1809017,53.1808809,53.1808748,53.1808537,53.180833,53.1808261,53.1808009,53.1807226817625,53.1807124,53.1807045,53.1807026,53.1807075,53.1807107,53.1807359,53.180782,53.1808579,53.1809349,53.1810171,53.181218,53.1815344,53.1817159,53.1819757739944,53.1821092,53.1823135,53.1824573,53.1825745,53.1828101,53.1828512,53.1829515,53.1830377,53.1830783,53.18315,53.1836069396843,53.184541467848,53.1846854857885,53.1847225188951,53.184813044132,53.1848994544071,53.1851751431688,53.1853068147878,53.1854453438672,53.1858664146584,53.1859733961552,53.1863176553144,53.1864479518848,53.1866715124464,53.1869156447345,53.187145]}]],[[{"lng":[8.212924,8.21296003836327,8.21312729556395,8.21322834678936,8.21335378968988,8.21364648979107,8.21383116961682,8.21404372564269,8.214479],"lat":[53.140346,53.1401305194075,53.139764728112,53.1395807861693,53.1394846343857,53.1393215939131,53.139064490372,53.1388596425468,53.138916]}]],[[{"lng":[8.214479,8.21493227952131,8.21516922722227,8.21518664984734,8.21508370363246,8.21473804702469,8.21581717009287,8.21668552693679,8.217053],"lat":[53.138916,53.1390310458959,53.1390958445448,53.139332045244,53.1396218318034,53.1404107584816,53.1402995004687,53.1402034137711,53.140241]}]],[[{"lng":[8.217053,8.22079199323073,8.22261540162156,8.22436776812704,8.22607277337561,8.22775409799573,8.22960118701501,8.23173244357573,8.23360321322347,8.23537926035739,8.23729739126204,8.24030483107549,8.24440157979775,8.24636707195929,8.24776422903799,8.24885353794679,8.24961131805727,8.25025069502548,8.25084271073679,8.25136368456275,8.25171889398953,8.25216882593013,8.25261875787072,8.25313973169667,8.25446584689001,8.25609981025322,8.25858627624072,8.2613805903981,8.26396177889941,8.26542997786346,8.26746651191036,8.26940832344346,8.2713264543481,8.27246312451381,8.27341034965191,8.2738602815925,8.27445229730381,8.2748548679875,8.27537584181345,8.27601521878167,8.27672563763524,8.27741237586036,8.27800439157167,8.27843064288381,8.27857272665452,8.27895161670976,8.27937786802191,8.28013564813238,8.28160384709643,8.28307204606048,8.28513226073583,8.28686094661286,8.28892116128821,8.29098137596357,8.29278110372595,8.293254716295,8.29330207755191,8.29297054875357,8.29278110372595,8.29263901995524,8.2928047843544,8.29313631315274,8.29450978960297,8.29597798856702,8.29751722941643,8.29941167969262,8.30064307237214,8.30213495196464,8.30339002527262,8.30464509858059,8.30587649126011,8.30682371639821,8.30848136038988,8.30954698867023,8.31044685255142,8.31137039706107,8.31217553842845,8.31298067979583,8.31414103059,8.31627228715071,8.31880611439511,8.31972965890476,8.32089000969892,8.32193195735083,8.3231870306588,8.32503411967809,8.3271653762388,8.32901246525809,8.33026753856606,8.33109636056189,8.33223303072761,8.33329865900797,8.33490894174273,8.33689811453273,8.33838999412523,8.33997659623154,8.34206049153534,8.34449959626594,8.34693870099653,8.3491409994426,8.35044343400748,8.3516274654301,8.35337983193558,8.35527428221177,8.35700296808879,8.35880269585117,8.36034193670058,8.36136020372403,8.36235479011903,8.36325465400022,8.36368090531236,8.36382298908308,8.36382298908308,8.36346777965629,8.36311257022951,8.36273368017427,8.36278104143117,8.3630178477157,8.36346777965629,8.36415451788141,8.36538591056093,8.36645153884129,8.3675408477501,8.36841703100284,8.36938793676939,8.37040620379284,8.37104558076105,8.37175599961462,8.37241905721129,8.37296371166569,8.37353204674855,8.37402933994605,8.37445559125819,8.37476343942807,8.37502392634105,8.3754975389101,8.37618427713522,8.37670525096117,8.37734462792938,8.37791296301224,8.37855233998045,8.37904963317795,8.37957060700391,8.37976005203152,8.37994949705914,8.38032838711438,8.38103880596795,8.38234124053283,8.38357263321236,8.38456721960736,8.38610646045676,8.38738521439319,8.38856924581581,8.38958751283926,8.39070050237652,8.39188453379914,8.39285543956569,8.39375530344688,8.39458412544271,8.3959812825214,8.39763892651307,8.39948601553235,8.40040956004199,8.40116734015247,8.40246977471735,8.40355908362616,8.40479047630568,8.40609291087057,8.40708749726556,8.40744270669235,8.40777423549068,8.4079163192614,8.40786895800449,8.40786895800449,8.40801104177521,8.40860305748652,8.40992917267985,8.41191834546985,8.4138838376314,8.41660710990342,8.41774378006913,8.41883308897794,8.41975663348759,8.42051441359806,8.42129587433699,8.42202997381901,8.4228824764433,8.42385338220985,8.42432699477889,8.42454012043497,8.42425595289354,8.42364025655378,8.42295351832866,8.42221941884663,8.4218878900483,8.42174580627759,8.42153268062152,8.4208459423964,8.42011184291437,8.41956718845997,8.41923565966164,8.41914093714783,8.4194014240608,8.42018288479973,8.42122483245163,8.4228351151864,8.42468220420568,8.42723971207854,8.43041291629116,8.43289938227866,8.43512536135318,8.43711453414318,8.43910370693318,8.44021669647044,8.44140072789306,8.44192170171901,8.44249003680187,8.44324781691235,8.44438448707806,8.44632629861115,8.44769977506139,8.44912061276854,8.45039936670496,8.45167812064139,8.45286215206401,8.45423562851425,8.45622480130425,8.4577877227821,8.46020314688425,8.46238176470187,8.4637552411521,8.46527080137305,8.46626538776805,8.46697580662162,8.4676862254752,8.46801775427353,8.4686808118702,8.46953331449448,8.469770120779,8.46962803700829,8.46920178569615,8.46853872809948,8.46749678044758,8.46626538776805,8.46484455006091,8.46380260240901,8.46304482229853,8.46228704218805,8.46157662333448,8.46039259191186,8.45954008928758,8.45845078037877,8.45646160758877,8.45466187982639,8.45343048714686,8.45290951332091,8.45234117823806,8.45148867561377,8.45011519916353,8.44859963894258,8.44760505254758,8.44727352374925,8.44717880123544,8.44732088500615,8.44826811014425,8.45011519916353,8.45210437195353,8.45414090600044,8.45797716780972,8.46148190082067,8.46313954481234,8.46408676995043,8.46489191131782,8.46503399508853,8.46455505165174,8.46459882142443,8.46473013074251,8.464983],"lat":[53.140241,53.1404373058309,53.1404231008689,53.1404088959022,53.1402668459768,53.1400253600257,53.1399543344876,53.1399117191084,53.1400821803716,53.1403378709982,53.1408066331941,53.1411901620949,53.1416163013017,53.1417015286358,53.1415310737986,53.1412469809,53.1411617526641,53.1412469809,53.1413606182849,53.1416163013017,53.1421418671678,53.1429231018412,53.1435196714779,53.1440878253332,53.1448264141093,53.1456928193692,53.146985293091,53.148263525547,53.1496979411089,53.1504364334067,53.151174913004,53.1518565752058,53.1524530207553,53.1527228386392,53.1530068556326,53.1532340678746,53.1536600875876,53.1541003035169,53.1546257166208,53.1550801227687,53.1553215240786,53.1553215240786,53.155165323386,53.154753518836,53.1543701110491,53.1539724993573,53.1537168898966,53.1536458869986,53.1537310904622,53.1539298978862,53.1540719026254,53.1540719026254,53.1538162937566,53.1534186769359,53.152836445662,53.1526092313158,53.1522116033167,53.1518423740203,53.1514305375978,53.151118107332,53.1508056747929,53.1504790383441,53.1502234080858,53.1500671888454,53.1499961799119,53.1499677763056,53.1499677763056,53.150010381708,53.1501097941491,53.1501950046298,53.1503228200337,53.1504364334067,53.1510328986832,53.1513595309188,53.1517713680224,53.1522968096265,53.1528648473707,53.1536032852035,53.1545689155142,53.1565427099033,53.1589849773441,53.1599788836158,53.1609443705295,53.1616400755786,53.1621937919463,53.1625487345788,53.1625487345788,53.1623499670662,53.162080209684,53.1617820548154,53.1615832837523,53.1614838978756,53.1614838978756,53.161441303858,53.1614838978756,53.1618388463786,53.1624209555692,53.1632586110381,53.1640252644268,53.1648344948138,53.165317186376,53.1656721031789,53.1658566587564,53.1657572827748,53.1655727267699,53.1654023666764,53.165175218833,53.1650900380818,53.1651610220529,53.1652745961624,53.1655017434798,53.1657998725094,53.1661689817719,53.1667936209839,53.1676737789876,53.1685255276098,53.168979786629,53.1692495006465,53.1694766269252,53.1696185802389,53.1695617989698,53.1695050176255,53.1694198454682,53.1694198454682,53.1695050176255,53.1698173140892,53.1701721936754,53.1707257999693,53.1711090616809,53.1712368148241,53.1712793991206,53.1711942304853,53.1709955030121,53.170441900198,53.1697321425518,53.1689513955812,53.1683551792379,53.1679860887787,53.1679151094803,53.1679293053494,53.168099655412,53.1686674840699,53.1693062823289,53.1697321425518,53.1702573643394,53.1708251644455,53.1713503728542,53.1717762127905,53.1717620181941,53.1715065146549,53.1708535542535,53.1705838503184,53.1702573643394,53.1699734614687,53.1700444373625,53.1702431692404,53.1706974100767,53.1713787623148,53.1720033256951,53.1728833768458,53.1737208281144,53.1746718122763,53.1752253605151,53.1755092286363,53.1756937419079,53.1756511619926,53.1755234219931,53.1753247145712,53.1753814882142,53.1755234219931,53.1760059933288,53.1765879103669,53.1771698195116,53.1777233355191,53.1782200745732,53.1786316540034,53.1792135354251,53.1800366712725,53.180760450087,53.1816403216094,53.1818815735513,53.1818815735513,53.1815125994438,53.180916557642,53.1800366712725,53.1793980327643,53.1790716138549,53.1791283825393,53.1792986881418,53.1797244491898,53.1802353568691,53.1807320668342,53.1812571539715,53.1820944417848,53.1829459041544,53.1839534461225,53.1850602959297,53.1866211894164,53.187529319484,53.1883239175173,53.1890901231049,53.1898988808168,53.1906224932077,53.1912893408731,53.1919278023234,53.1926088173913,53.1927790694681,53.1926513804739,53.1926371927844,53.1926088173913,53.1927081311849,53.193020258753,53.1935735702208,53.1940985001406,53.1950064718551,53.1961130363825,53.197673527557,53.1984679376067,53.1990921066034,53.1997730078704,53.2001985656696,53.2006524893318,53.2007943394904,53.2008227094658,53.2005957491369,53.2001418248738,53.1994325585889,53.1989502508113,53.1985246806184,53.1981558497001,53.1980707344222,53.1981558497001,53.1984395660728,53.1988651371107,53.1993758167791,53.2001134544478,53.2008510794224,53.201872385354,53.2026383488321,53.2035461396726,53.2044539112849,53.2051914615617,53.2056736991104,53.2059289991452,53.2058438993026,53.205645332346,53.2051630944781,53.2042837055722,53.2036596121756,53.2031206151102,53.2029504041023,53.2030355096908,53.2036028759617,53.2040283957357,53.2046241163215,53.2051630944781,53.2061842976589,53.2070069158573,53.2081699008481,53.2097583171552,53.2106943206203,53.2112899485428,53.212112468728,53.2130767826293,53.2147500993587,53.2163666314157,53.2180681782068,53.2193726516405,53.2210173658679,53.2224635279194,53.2233708987847,53.2242498959779,53.2256108882954,53.2271136003993,53.2287599591702,53.2306988285629,53.2330829935282,53.235771]}]],[[{"lng":[8.62331,8.61943169259801,8.61573751455944,8.61071722132754,8.60493914798516,8.6003451060654,8.59499328403516,8.58774701172874,8.58272671849683,8.57704336766826,8.57173890689493,8.56757111628731,8.56136679163279,8.55393107429874,8.5474899433597,8.5415697862466,8.53484448776613,8.52972947202042,8.52575112644042,8.52253056097089,8.51916791173066,8.51542637243518,8.51286886456233,8.50855899018399,8.5046280058609,8.49922882257376,8.493640194259,8.48947240365138,8.48483100047472,8.48123154494995,8.47791625696662,8.47526402657996,8.47242235116567,8.46962803700829,8.46654955530948,8.46556175642366,8.4647739005152,8.46459882142443,8.46473013074251,8.464983],"lat":[53.16807,53.1684687448943,53.1690649596598,53.1702573643394,53.1719323530416,53.172784017136,53.1733801719435,53.1737776038799,53.1742034197264,53.1751401997125,53.1763040493832,53.1774394820485,53.1794831851148,53.1821512064687,53.1847339201102,53.1869191717147,53.189075934237,53.1910907064828,53.1931337591234,53.1946659847157,53.1965386305183,53.198893508363,53.2005390088669,53.2036596121757,53.2070636475647,53.2107510474458,53.2135022083327,53.2152322293382,53.21687710255,53.2178696678973,53.2190323580531,53.2207621557309,53.2233425437362,53.2251288751298,53.2266599570192,53.2275284695053,53.229100577718,53.2306988285629,53.2330829935282,53.235771]}]],[[{"lng":[8.801675,8.79992840274055,8.79770242366602,8.79377143934291,8.79055087387337,8.78624099949503,8.78027348112501,8.7754426329207,8.7715590098545,8.76843316689878,8.76511787891543,8.76208675847352,8.76076064328018,8.75848730294874,8.75564562753445,8.75261450709253,8.74929921910919,8.74636282118108,8.74437364839108,8.74247919811488,8.74200558554583,8.74162669549059,8.74181614051821,8.74143725046297,8.74039530281106,8.73840613002105,8.73461722946866,8.72959693623674,8.72372414038053,8.71936690474528,8.71614633927574,8.71356515077442,8.70958680519441,8.7056558208713,8.69786489411044,8.69218154328185,8.68337234949754,8.67683649604466,8.6656592394151,8.65588034165615,8.65185058839185,8.64528784736142,8.63872510633098,8.63308345176096,8.62686611815318,8.62329690811908,8.62168500681336,8.62030337712274,8.61961256227744,8.61926715485478,8.61987161784442,8.62148351915015,8.62269244512944,8.62295150069643,8.62331],"lat":[53.075981,53.0769439875924,53.0779397923975,53.0792485293998,53.0800451324447,53.0807848220736,53.0818089867317,53.0830607104466,53.0844830795775,53.0864743173994,53.0893187830599,53.0931300723082,53.0955190665809,53.097737299638,53.0993866808494,53.1009791268449,53.1020596816271,53.1033108164125,53.1047893833331,53.106609088232,53.1092247791435,53.1113285889989,53.1151947819601,53.1182078971798,53.1205386527687,53.1219597831823,53.1230966536854,53.1234945512577,53.1230966536854,53.1218460944782,53.1201975744783,53.1187479615521,53.1177246759378,53.1178668003963,53.1184352935319,53.1194017146096,53.121618716168,53.1235513934674,53.1260523762723,53.1273355658868,53.1276809941603,53.1287863459776,53.1301679957689,53.1320331525227,53.1352106405346,53.1376972063736,53.140183628296,53.1434986336619,53.1474348702468,53.150680267638,53.1544259799009,53.1573774400065,53.1617956209071,53.1660925559192,53.16807]}]],[[{"lng":[8.8031,8.801675],"lat":[53.07518,53.075981]}]],[[{"lng":[8.8031,8.80470729606328,8.80578594413185,8.80606017669166,8.806253],"lat":[53.07518,53.0755167799051,53.0756870196541,53.0756156288735,53.075689]}]],[[{"lng":[8.806253,8.80656293638463,8.807392],"lat":[53.075689,53.075846275584,53.075887]}]],[[{"lng":[8.807392,8.80752002683316,8.80772864586203,8.80772864586204,8.8079813,8.8081377,8.8082585,8.8083564,8.8083812,8.8083961,8.8084916,8.8086069,8.8086543,8.8086698,8.8086893,8.8087361,8.8088627,8.8088858,8.8090116,8.80908142363214,8.8092313021287,8.809272],"lat":[53.075887,53.0758416159592,53.0757521165576,53.0757521165576,53.0755993,53.0754877,53.075381,53.0752733,53.0752428,53.0752246,53.0751073,53.0749628,53.0749223,53.0749125,53.0749001,53.0748717,53.0748173,53.0748101,53.0747708,53.0747519465304,53.0746792616687,53.074656]}]],[[{"lng":[8.860087,8.85985951271213,8.85945745965384,8.859274,8.8587726,8.8586525,8.8586341,8.8585024,8.8581464,8.8579259,8.8576697,8.8573763335119,8.85718933292488,8.85683574239385,8.85620595764141,8.8554471,8.8554471,8.8549543,8.8547291,8.8545062,8.8535213,8.8524333,8.8522435,8.8520849,8.8520206,8.8519875,8.8517234,8.8514575,8.8507674,8.8506956,8.8505921,8.8505283,8.8503893,8.8499292,8.84959203574631,8.8492709,8.848682,8.8484703,8.8482394,8.8480002,8.8477449,8.847552,8.8469902,8.8469214,8.8468389,8.8467339,8.846054,8.845785,8.8454438,8.8453307,8.8452335,8.8450633,8.8449452,8.844889,8.8448481,8.8443321,8.8441483,8.8439901,8.8436012,8.8433159,8.8431169,8.8429511,8.8427867,8.8426221,8.8424211,8.8418814,8.8417759,8.8416946,8.8415423,8.8413186,8.8411419,8.8410463,8.8409538,8.8407171,8.8405052,8.8404713,8.8399618,8.8396483,8.8389963,8.8380303,8.8378858,8.8378297,8.8372441,8.83338049605247,8.8329509310293,8.8324857435408,8.83209816001473,8.8320207,8.8314025,8.8309417,8.8307311,8.8303264,8.8301347,8.8298901,8.829652,8.8295338,8.82939670522694,8.8289142,8.8286695,8.8284745,8.8283504,8.8281491,8.8273673,8.827219,8.8271665,8.82707483226633,8.8270167,8.8267112,8.8265384,8.8264201,8.8263323,8.826223,8.8261191,8.8260151,8.8258657,8.8257277,8.8256031,8.8252172,8.82497094550647,8.8246599,8.824252,8.8240934,8.8239596,8.8235109,8.8234623,8.8232417,8.8230696,8.8225491,8.8221396,8.8219055,8.8216757,8.821497,8.8212967,8.8205033,8.8200953,8.8197468,8.8194257,8.8191971,8.8189904,8.8186251,8.8184177,8.8181588,8.8180094,8.8179098,8.8177171,8.8175103,8.8172769,8.8171966,8.817133,8.8167859,8.8166113,8.8164552,8.8162913,8.8161392,8.8159664,8.815839,8.8156614,8.8155683,8.8153578,8.8152974,8.815196,8.8151047,8.8145437,8.8144555,8.8141588,8.8140754,8.813797,8.8133909,8.8131191,8.8128696,8.8127178,8.8125468,8.8124679,8.8119125,8.8116341,8.8113132,8.8111389505097,8.8109921,8.8106641,8.8104992,8.8104035,8.8101563,8.8101481,8.8100996,8.8099814,8.80965249510668,8.8095812,8.80946515490872,8.80930390036383,8.809272],"lat":[53.068596,53.0686965222489,53.068865130349,53.0689353,53.0691255,53.0691802,53.0691885,53.0692676,53.069498,53.0696726,53.0699192,53.0701402569276,53.0702313989095,53.0703627500748,53.0705004340255,53.0705897,53.0705897,53.0706243,53.0706433,53.0706494,53.0707447,53.0708508,53.0708687,53.0708865,53.0708981,53.0709046,53.0709373,53.0709845,53.0711329,53.0711478,53.0711693,53.0711811,53.0712068,53.071281,53.0713343173622,53.0713851,53.0714789,53.0715117,53.0715453,53.0715707,53.071587,53.0715897,53.0715967,53.0715975,53.0715985,53.071599,53.0716077,53.071609,53.0716087,53.0716086,53.0716109,53.0716265,53.0716515,53.0716689,53.0716816,53.0718843,53.071952,53.0720055,53.07212,53.0722068,53.0722615,53.0723049,53.0723428,53.0723796,53.0724247,53.0725322,53.0725504,53.0725599,53.0725737,53.0725759,53.0725626,53.0725497,53.0725349,53.0724885,53.0724432,53.0724375,53.072333,53.0722706,53.0721396,53.0719425,53.0719144,53.0719031,53.0717836,53.0709715824245,53.0709130392171,53.0709105212279,53.0710115612013,53.0710373,53.0712484,53.0714075,53.0714796,53.0716176,53.071684,53.0717707,53.0718641,53.07191,53.0719652717102,53.0721598,53.0722534,53.0723202,53.0723587,53.072417,53.0726536,53.0727007,53.0727162,53.0727431863008,53.0727603,53.0728539,53.0728992,53.072928,53.0729452,53.0729648,53.0729793,53.0729932,53.0730065,53.0730128,53.073015,53.073025,53.0730293303323,53.0730348,53.0730411,53.0730398,53.0730325,53.0729885,53.0729842,53.0729645,53.0729439,53.0728836,53.0728423,53.0728211,53.0728011,53.072786,53.0727703,53.0726978,53.072654,53.072613,53.0725737,53.072543,53.072515,53.0724638,53.0724383,53.072413,53.0724019,53.0723949,53.0723858,53.0723825,53.0723802,53.0723803,53.0723804,53.0723933,53.0724022,53.0724133,53.0724321,53.0724648,53.0725143,53.0725509,53.0726036,53.0726313,53.0726909,53.072708,53.0727364,53.0727625,53.0729297,53.0729562,53.0730332,53.0730515,53.0731126,53.0732001,53.0732587,53.0733134,53.0733515,53.0734008,53.0734244,53.0735824,53.0736651,53.0737766,53.073840363089,53.0738941,53.0740196,53.0740803,53.0741229,53.0742454,53.0742495,53.0742735,53.0743322,53.0744936116624,53.0745286,53.0745890811461,53.0746432785395,53.074656]}]],[[{"lng":[8.886329,8.88631639629589,8.88626254401257,8.8862364,8.8860785,8.8857527,8.8855075,8.8854345,8.8853224,8.8851625,8.8849988,8.8848883,8.8847572,8.8846497,8.8845523,8.8845221,8.8845035,8.8844807,8.8844764,8.8845242,8.8845597,8.88458856739937,8.8846072,8.8846249,8.8847746,8.8847886,8.8848245,8.88477406244045,8.88466436941652,8.88399707018767,8.88332977095881,8.88231511048753,8.8815198360641,8.88041376473955,8.87894205000193,8.87686702363275,8.87418868563199,8.87339341120856,8.87186684995898,8.87055967475724,8.86948102668868,8.86924335847018,8.8689505,8.8686405,8.8681818,8.867815,8.8677464,8.8671632,8.8668786,8.86639025703059,8.8661362,8.8654386,8.86484089540149,8.8637805990093,8.8636641,8.863547,8.863446,8.8634122,8.8629547,8.8624975,8.8624283,8.8613638,8.8609115,8.86070082405261,8.86011987102495,8.860087],"lat":[53.051802,53.0518089115867,53.0518346436407,53.0518461,53.0518974,53.0519867,53.0520643,53.0521004,53.0521559,53.052284,53.0524321,53.0525505,53.052694,53.0528348,53.0529999,53.0530932,53.0531675,53.0532816,53.0534308,53.053779,53.054007,53.054184641096,53.0542993,53.0544109,53.0552065,53.0552861,53.055477,53.0556586388658,53.0558124747375,53.056433306782,53.0570156720729,53.0577958094671,53.05840562534,53.0588066707138,53.0591527753641,53.0597625720306,53.0607514131429,53.0610755283463,53.0625092968569,53.0638496349884,53.0650745796743,53.0652338748541,53.0654058,53.0655234,53.0656985,53.0658407,53.0658658,53.0660792,53.0661689,53.0663261118541,53.0664079,53.0665945,53.0668011240546,53.0671409138673,53.0671771,53.0672159,53.0672494,53.0672622,53.0674389,53.0676056,53.0676332,53.0680667,53.0682409,53.0683225886589,53.0685776983576,53.068596]}]],[[{"lng":[8.918209,8.91819277305493,8.9181513,8.9181513,8.9179145,8.917715,8.9174825,8.9171676,8.9170919,8.9167923,8.9165149,8.9161775,8.916068,8.9157835,8.9153492,8.9148378,8.9142468,8.9135037,8.9130778,8.9126969,8.9123243,8.9111047,8.9109102,8.9107267,8.9092356,8.9090114,8.9089189,8.9087475,8.9083941,8.90767689484994,8.907447,8.9072754,8.9060319,8.9060163,8.905896,8.9054206,8.9045152,8.9031198,8.9030727,8.9028473,8.9023486,8.9022941,8.901831,8.9013009,8.9008726,8.9005574,8.9004183,8.9002778,8.8999167,8.8996221,8.8993824,8.8986532,8.89842525572772,8.8981711,8.8977132,8.8971718,8.8968851,8.8961306,8.895993,8.8952553,8.8945493,8.8933747,8.8924891,8.8920492,8.88917208694532,8.8875609,8.8875609,8.8874922,8.8874087,8.8873398,8.8871113,8.8870381,8.8868014,8.8865795,8.8864361,8.886361,8.886329],"lat":[53.038871,53.0388755539018,53.038887,53.038887,53.0389339,53.0389622,53.0389814,53.0390015,53.0390064,53.0390223,53.039037,53.0390507,53.0390496,53.0390455,53.039034,53.0390204,53.0390062,53.0389941,53.0390183,53.0390753,53.0391384,53.0393476,53.0393814,53.0394179,53.0396997,53.0397412,53.0397579,53.0397892,53.0398538,53.0399848068117,53.0400268,53.0400581,53.040287,53.0402899,53.040312,53.0404286,53.0406959,53.0411163,53.0411305,53.0411982,53.0413562,53.0413735,53.0414993,53.0416519,53.0418099,53.0419331,53.0420053,53.0420765,53.0422952,53.0424925,53.0427218,53.0434694,53.0436371079625,53.0438241,53.0441279,53.044487,53.0446772,53.0451979,53.0452889,53.0457758,53.0462621,53.0470279,53.0476157,53.0479168,53.0494976485665,53.0504845,53.0504845,53.050541,53.0506186,53.0506845,53.0509032,53.0510067,53.0513948,53.0516625,53.0517494,53.0517915,53.051802]}]],[[{"lng":[8.941825,8.94133113586961,8.94099431083797,8.9406316,8.9405792,8.9401806,8.939918,8.9394206,8.9392944,8.9389499,8.9388534,8.9385688,8.9382278,8.9380497,8.9377525,8.9374342,8.9364599,8.936215,8.9361231,8.9353106,8.9351371,8.9348746,8.93453862777655,8.93425745720162,8.933976,8.933976,8.933737,8.9335918,8.9332957,8.9329737,8.93128088590698,8.92998518173193,8.9296846,8.9288929,8.9279976,8.926982,8.9265087,8.9261111,8.9256645,8.9255135,8.9255056,8.9253543,8.9252807,8.9251643,8.9249425,8.9247234,8.924593,8.9244302,8.9239958,8.9238463438277,8.9237512,8.9235079,8.9230401,8.9224515,8.9222383,8.9216686,8.9210137,8.9206918,8.9203773,8.9201417,8.9199381,8.9197437,8.9196189,8.9194871,8.9191935,8.9190496,8.9188787,8.9187241,8.9185418,8.9183094,8.918209],"lat":[53.038019,53.0381029336738,53.038166887166,53.0382094,53.0382126,53.0382563,53.0382888,53.0383514,53.0383677,53.0384084,53.0384198,53.0384534,53.0384969,53.0385098,53.0385037,53.0384642,53.0383294,53.0383391,53.038341,53.0383681,53.0383738,53.0383752,53.0383868274545,53.0383935094858,53.0383888,53.0383888,53.0384052,53.0384145,53.0384273,53.038437,53.0385017071333,53.0385293199054,53.0385134,53.0384814,53.0384444,53.0384041,53.0383848,53.0383691,53.0383495,53.0383442,53.0383437,53.0383331,53.038328,53.0383153,53.0382845,53.0382322,53.0381901,53.0381338,53.0379835,53.037931440879,53.0378983,53.0378244,53.0377237,53.0376834,53.0376698,53.0376333,53.0375899,53.0375775,53.0375812,53.0376145,53.0376747,53.0377856,53.0378569,53.0379817,53.0383194,53.0384562,53.0385813,53.038672,53.0387557,53.0388408,53.038871]}]],[[{"lng":[8.970547,8.9704001153028,8.97021505707998,8.9689267,8.9688475,8.9685199,8.9676109,8.9674941,8.9672354,8.9668384,8.9666049,8.9663422,8.9661234,8.965911,8.9656019,8.9648773,8.9646292,8.9643658,8.9635775,8.9634169,8.9632477,8.962695,8.96223876842883,8.96200250301107,8.9615588,8.9604265,8.9600595,8.9598834,8.9597228,8.9595894,8.9587102,8.9582664,8.957592,8.9573981,8.9570098,8.9563313,8.9563091,8.9556271,8.9554993,8.9552425,8.95465737053915,8.9545528,8.9544774,8.9539559,8.9538737,8.9536666,8.9529745,8.9527484,8.9524252,8.9522179,8.9520515,8.9515975,8.9515344,8.9509421,8.9503382,8.9500669,8.9499298,8.9496466,8.9492636,8.9488637,8.9484484,8.9480567,8.9473081,8.9470464,8.9469862,8.9464493,8.9458613,8.9456843,8.9454453,8.945265,8.9451119,8.9446787,8.9443307,8.9442826,8.9441752,8.9441167,8.9439552,8.9437773,8.9432822,8.9431045,8.9429884,8.9428478,8.9423904,8.94202786938279,8.941825],"lat":[53.028939,53.0289865209996,53.0290393179281,53.0292113,53.0292241,53.0292643,53.0293757,53.0293958,53.0294403,53.0295274,53.0295926,53.0296774,53.0297567,53.0298482,53.0299984,53.0304079,53.0305336,53.030648,53.030914,53.030966,53.0310239,53.0312128,53.0313592024394,53.0314350186402,53.0315774,53.0319435,53.0320655,53.0321241,53.032175,53.0322172,53.0325001,53.0326429,53.0328676,53.0329298,53.0330544,53.0332754,53.0332824,53.0335052,53.0335469,53.0336308,53.0338218558311,53.033856,53.0338806,53.0340472,53.0340777,53.0341482,53.0343701,53.0344426,53.0345462,53.0346127,53.034666,53.0348116,53.0348318,53.0350217,53.0352233,53.0353207,53.0353754,53.0355137,53.0357051,53.0359152,53.0361317,53.0363409,53.0367484,53.0368793,53.0369123,53.0371815,53.0374846,53.0375611,53.0376526,53.0377151,53.0377597,53.0378614,53.0379282,53.0379379,53.0379592,53.0379676,53.0379909,53.038012,53.0380528,53.0380601,53.0380649,53.0380659,53.0380647,53.0380355261054,53.038019]}]],[[{"lng":[9.032515,9.0308066,9.0307533,9.0303882,9.0301268,9.0299843,9.029894,9.029625,9.0294181,9.0290243,9.028613,9.0284716,9.0280836,9.027403,9.026898,9.026094,9.0257179,9.0255197,9.0252883,9.0250378,9.0239077,9.0237889,9.0219542,9.0211185,9.0206761,9.0204229,9.0197311,9.0192808,9.0192171,9.0189685,9.0183838,9.0181415,9.0180325,9.0173646,9.0172696,9.0163765,9.0160982,9.0158085,9.0153933,9.0142137,9.0134133,9.0129198,9.0123102,9.0103988,9.0103791,9.0099804,9.0098604,9.0086986,9.0078321,9.0066891,9.0051725,9.0041787,9.0032715,9.0017619,8.9999282,8.99975547186457,8.9986584,8.9982251,8.9979973,8.9970871,8.9969136,8.9958517,8.9952035,8.9948669,8.9938279,8.9936124,8.9924807,8.992177,8.991745,8.9909776,8.988935,8.987684,8.9871944,8.9868982,8.9865694,8.9863134,8.9859202,8.9852761,8.985065,8.9848896115037,8.9847979,8.984503,8.9838155,8.9835434,8.9834071,8.98305,8.982613,8.9822854,8.9819386,8.9815433,8.9811907,8.9793153,8.9785651,8.9785122,8.9783918,8.9781729,8.9775108,8.97729,8.9770461,8.9769376,8.9764334,8.9757702,8.9756508,8.9755544,8.9750947,8.9745771,8.974098,8.9737232,8.9733422,8.9730178,8.9713948,8.97129528803482,8.971042,8.970547],"lat":[53.01297,53.0126223,53.0126107,53.0125315,53.0124699,53.0124376,53.0124199,53.0123766,53.0123504,53.0123435,53.0123841,53.0124028,53.0124516,53.012549,53.0126179,53.0127212,53.0127802,53.012824,53.0128953,53.012991,53.0134348,53.0134828,53.0142105,53.014542,53.0147171,53.0148173,53.0150911,53.0152693,53.0152945,53.0153912,53.0156227,53.0157182,53.0157621,53.0160308,53.0160682,53.0164102,53.0165167,53.0166186,53.0167706,53.0170432,53.0172381,53.0173583,53.0175041,53.0179493,53.0179539,53.018043,53.018071,53.0183468,53.0185537,53.0188219,53.0191796,53.0194124,53.0196285,53.019981,53.0204129,53.020453790088,53.0207135,53.0208161,53.0208682,53.0210757,53.0211163,53.0213624,53.0215189,53.0215995,53.0218436,53.0218931,53.0221487,53.0222169,53.0223249,53.0224987,53.022976,53.0232833,53.0234161,53.0235117,53.0236313,53.0237301,53.0238983,53.0241793,53.0242726,53.0243501491993,53.0243907,53.0245211,53.0248148,53.0249282,53.0249834,53.0251166,53.0252608,53.0253565,53.0254486,53.0255414,53.0256214,53.0259939,53.0261459,53.026156,53.0261804,53.0262244,53.0263585,53.0264005,53.026448,53.0264671,53.0265793,53.0267371,53.0267736,53.0268031,53.0269582,53.0271512,53.0273512,53.0275283,53.0277084,53.0278617,53.0286074,53.0286532353634,53.0287699,53.028939]}]],[[{"lng":[9.054039,9.0539232464372,9.05373652471005,9.05353107878106,9.0512047,9.0499818,9.0489883,9.04568944229566,9.0445361,9.0438315,9.04354529487415,9.0435074,9.0430235,9.04290297914774,9.04280899032416,9.0426033,9.0420357,9.0413645,9.0410954,9.0395732,9.0389438,9.0388885,9.0387887,9.0386987,9.0385158,9.0381135,9.0375041,9.0367557,9.0365687,9.0365258,9.0362627724292,9.03380823118458,9.032515],"lat":[53.029817,53.0297816613471,53.0297013615593,53.0295795341023,53.0279793,53.027138,53.0265228,53.0244797213154,53.0237654,53.0233093,53.0230250377667,53.0229874,53.0225087,53.0223745558194,53.0222699424817,53.022041,53.0214165,53.020678,53.0203663,53.018639,53.0180105,53.017954,53.0178504,53.0177444,53.0175484,53.0171032,53.0164337,53.0155946,53.0152977,53.0152127,53.0146114245583,53.0136579007547,53.01297]}]],[[{"lng":[9.085822,9.08579227697187,9.08572653163695,9.0856129,9.0854135,9.0852623,9.085035,9.084982,9.0848785,9.0847548,9.084784,9.08528476573549,9.0857914,9.0858371,9.0860027,9.0860275,9.0860988,9.0864844,9.0865604,9.0866856,9.0867675,9.0866592,9.08662766546173,9.08660483359212,9.0865686,9.0864682,9.0863373,9.0862453,9.085982,9.0858108,9.0855928,9.0853617,9.0851273,9.0837842,9.082788,9.0815999,9.0803908,9.0796573,9.0794994,9.0770513,9.0757635,9.0752457,9.0745649,9.0738078,9.0734664,9.06979523498886,9.0692805,9.068985,9.0682378,9.067887,9.0668393,9.0644248,9.0623267,9.0612391,9.0604702,9.05877089496952,9.0575562,9.0561637,9.0549434,9.05438276562692,9.05418994986646,9.054039],"lat":[53.06366,53.0636354483843,53.0635753755123,53.0634817,53.0633112,53.0631698,53.0629478,53.0628919,53.0627878,53.0626634,53.0623643,53.0616162098576,53.0607895,53.0607104,53.0604188,53.0603789,53.0602645,53.0596384,53.0595149,53.0591778,53.0587328,53.0580518,53.0578583124051,53.0577182215434,53.0574959,53.0568792,53.0565246,53.0562755,53.0556998,53.055418,53.0551334,53.0549148,53.0547389,53.0540371,53.0535645,53.0530246,53.0524392,53.0520024,53.0519068,53.0503782,53.0495926,53.0491873,53.048417,53.0475331,53.0471609,53.0428944357972,53.0422962,53.0419409,53.0410975,53.040726,53.0399152,53.0381268,53.0365923,53.0357771,53.0351898,53.033852014581,53.0330673,53.0317414,53.030528,53.0301552983868,53.0300000935982,53.029817]}]],[[{"lng":[9.143259,9.14322466689481,9.14311245929506,9.14280388839573,9.1426355769961,9.1409299,9.1406622,9.1403606,9.1400092,9.1399129,9.1395597,9.1378861,9.1377283,9.1368583,9.1364714,9.1361632,9.131835,9.129994,9.129919,9.1296268,9.1291713,9.1287649,9.1276096,9.1269482,9.1267646,9.1265502,9.126079,9.1257112,9.1253614,9.1242542,9.12405821816028,9.1240021,9.1236234,9.1221336,9.1217192,9.1216164,9.1212234,9.1210377,9.1208572,9.12052642191024,9.1195334,9.1194476,9.1183561,9.1169238,9.1164746,9.1160169,9.114777,9.1140314,9.1107682,9.1064727,9.1050263,9.1047501,9.104517,9.1040379,9.1037844,9.1034404,9.1030226,9.10291703299516,9.1006495,9.0970947,9.0958896,9.0926606,9.0914011,9.09127559463064,9.09078957369922,9.09048352003454,9.089999,9.0898787,9.089642,9.0891561,9.088609,9.0883085,9.0880313,9.087741,9.0876033,9.0866133,9.0865873,9.08618530568662,9.085822,9.085822],"lat":[53.110276,53.1097395657521,53.1093606840054,53.1086534291468,53.108249278291,53.1071113,53.1069665,53.1068262,53.1067117,53.1066827,53.1065999,53.1062986,53.1062617,53.105996,53.1058019,53.105607,53.1026744,53.1014771,53.1014252,53.1012416,53.1009406,53.1006881,53.0999648,53.0995316,53.0993937,53.0992261,53.0988088,53.0984238,53.0979699,53.0965112,53.0962531043439,53.0961792,53.0956803,53.0937175,53.0931715,53.0930361,53.0925182,53.0922736,53.0920358,53.0915999581431,53.0902915,53.0901785,53.0887404,53.086853,53.0863341,53.0858932,53.0849796,53.08443,53.0820248,53.0788585,53.0777922,53.0775886,53.0774168,53.0770635,53.0768767,53.0766231,53.0763151,53.0762372752603,53.0745656,53.0719448,53.0710563,53.0686755,53.0677468,53.067654257835,53.0672869428197,53.0670702782869,53.066713,53.0666242,53.0664497,53.0660914,53.065688,53.0654664,53.065262,53.0650479,53.0649464,53.0642164,53.0641972,53.0639114283474,53.06366,53.06366]}]],[[{"lng":[9.146686,9.14904543613205,9.15044803112898,9.15120543242732,9.15140179572689,9.1482132,9.1480033,9.1473363,9.1468655,9.1458365,9.1444917,9.1440738,9.1433882,9.1433041,9.1431838,9.1430846,9.1430349,9.1429899,9.1429049,9.1428083,9.1422157,9.1421576,9.1420451,9.1419435,9.1418534,9.1417161,9.1414762,9.1413528,9.1413129,9.1412872,9.1412184,9.1412158,9.141159,9.1410963,9.1410554,9.1410539,9.1409967,9.1409895,9.140986,9.14100168731322,9.1410851,9.1412157,9.1414062,9.1415908,9.1417176,9.1420058,9.1422902,9.1424234,9.143259],"lat":[53.140233,53.1398373637291,53.1393830235048,53.1392484032562,53.1380536300634,53.1342638,53.1336533,53.1319445,53.1307384,53.1284671,53.125895,53.1251426,53.123292,53.1229713,53.1225188,53.1220407,53.1218702,53.1217331,53.1215755,53.1214097,53.1206048,53.1205259,53.1203564,53.1201759,53.1198612,53.118978,53.1168812,53.1154243,53.1150812,53.1145635,53.1140661,53.1139018,53.1134016,53.1128166,53.1124358,53.1124117,53.1114874,53.1113445,53.1112754,53.1111906610556,53.1111738,53.1111498,53.1111148,53.1110738,53.1110334,53.1109195,53.1107559,53.110675,53.110276]}]],[[{"lng":[9.203015,9.20103612618065,9.20034885463216,9.19978781663339,9.19658990004039,9.19591665444187,9.19191925870063,9.18412083051772,9.1832799,9.1828754,9.1817071,9.1813706,9.1795441,9.1782245,9.17786525683142,9.17789330873136,9.17881902142933,9.17884707332927,9.17859460622982,9.169842413449,9.16355878786277,9.16038892316972,9.1584583,9.156605,9.1561521,9.1557417,9.15519826257128,9.1551191,9.1549283,9.1547917,9.1543433,9.1538617,9.1528783,9.1526933,9.1521434,9.1514815,9.1512031,9.15104348542341,9.1506949,9.1505533,9.1503126,9.1498667,9.148605,9.1479531,9.1478381,9.1475683,9.147135,9.1469835,9.1469179,9.1468433,9.146865,9.1468783,9.14689150428439,9.1469117,9.1469504,9.1469733,9.1470035,9.1470141,9.14701,9.1469421,9.146686],"lat":[53.218914,53.2177906303544,53.2173791073985,53.2171355511337,53.2165728468814,53.2161445148093,53.2108026021674,53.1948737021786,53.1930447,53.1927634,53.1919072,53.1916606,53.1901567,53.1891024,53.1885710790696,53.1881004460395,53.1871759732635,53.1866717069853,53.1862682896933,53.1809898971529,53.1778628996455,53.1761984370481,53.175025,53.1714733,53.1698776,53.1684317,53.1657057096671,53.1653086,53.1645583,53.1640633,53.161945,53.1599033,53.1573567,53.15684,53.1556099,53.153761,53.1530482,53.1527002671718,53.1519404,53.1516317,53.1512742,53.1504867,53.148365,53.1472552,53.1470593,53.1466,53.1459367,53.1456515,53.1455022,53.1449817,53.1447283,53.1441583,53.1438150302378,53.14329,53.1428303,53.1425117,53.1419021,53.1412709,53.1410317,53.1407676,53.140233]}]],[[{"lng":[9.257674,9.25766739821405,9.25764065117808,9.2572673,9.2567278,9.2566213,9.2565714,9.2565604,9.2556789,9.2548007,9.2542385,9.253705,9.2535507,9.2527485,9.2527279,9.2523339,9.251316,9.2504878,9.2498176,9.2495337,9.2482609,9.2477533,9.2473199,9.2473038,9.2468786,9.2465095,9.2462554,9.2461318,9.2460901,9.2460152,9.2459291,9.2458432,9.2457022,9.2450316,9.2441131,9.2440209,9.2438045,9.2435408,9.2433477,9.2431311,9.2297277,9.2278768,9.2241971,9.2211088,9.216424,9.215836,9.2153542,9.2148856,9.2145042,9.2141769,9.2139783,9.2137844,9.2134734,9.2131978,9.2129158,9.212523,9.2121556,9.2119067,9.2116908,9.2116286,9.2116165,9.2116481,9.2117148,9.2117813,9.2120409,9.2126858,9.2127239,9.21279728953294,9.2128639,9.2130669,9.2131083,9.2130188,9.2129761,9.2128967,9.2122167,9.2122017,9.2114097,9.211215,9.2108384,9.2106565,9.2087187,9.2075937,9.2072525,9.2069425,9.20593,9.2056964,9.2053136,9.2047168,9.204404,9.2039144,9.2036372,9.203329,9.20315986687131,9.203015],"lat":[53.273983,53.2739509844833,53.2738760007209,53.2732267,53.2722886,53.2721034,53.2720167,53.2719975,53.270739,53.2696464,53.268853,53.2681751,53.267979,53.2669231,53.266896,53.2663772,53.2650366,53.263991,53.2631598,53.2628489,53.2617542,53.2613233,53.2609361,53.2609187,53.2604572,53.2598411,53.2593732,53.2589066,53.2582055,53.2572527,53.2564386,53.2558224,53.2551065,53.254591,53.2539028,53.2538404,53.2537083,53.2535902,53.253521,53.2534657,53.2510313,53.2506951,53.2500079,53.2494408,53.2486131,53.2485063,53.2484171,53.248308,53.2481817,53.2480298,53.2479121,53.2477702,53.2474554,53.2470467,53.2466042,53.2459049,53.2452216,53.2447467,53.2442231,53.243831,53.2433786,53.2428639,53.2421752,53.2415779,53.2400095,53.2359921,53.2357219,53.2351509328897,53.2346327,53.2333296,53.232945,53.2325305,53.2323326,53.231965,53.23056,53.2305283,53.2288517,53.2285612,53.2280523,53.2278302,53.2256921,53.2243679,53.2239663,53.2236014,53.2224095,53.2221345,53.2216839,53.2208971,53.2204864,53.2198841,53.219543,53.2192447,53.2191013058134,53.218914]}]],[[{"lng":[9.274875,9.27455402332585,9.27284565475866,9.27192327732532,9.26346562949386,9.26228744969644,9.25880901410406,9.25023915867284,9.25099655997118,9.25265162206755,9.25303032271672,9.25426460631402,9.25575135701076,9.25616185371333,9.2565818,9.2573555,9.2575383,9.25761775024945,9.25771672977242,9.257674],"lat":[53.296491,53.2963937313464,53.2959384554424,53.2957154634123,53.2946340296509,53.2930747048652,53.2885976174035,53.2773525257535,53.2764383651475,53.2762706271337,53.2762119186733,53.2758596662178,53.2751048297523,53.2749413132214,53.2747879,53.2745271,53.2743903,53.2742498335895,53.2740748391786,53.273983]}]],[[{"lng":[9.278606,9.27784924118734,9.27746352756318,9.27712690476392,9.2767271651898,9.27632742561567,9.27607010798505,9.27584084640244,9.27547106965629,9.274875],"lat":[53.295483,53.2953885212091,53.2953298390103,53.2953088810626,53.295342413774,53.2954178622783,53.29551411625,53.2956599833229,53.2960047580605,53.296491]}]],[[{"lng":[9.334932,9.33491678305797,9.33489362606441,9.3348252,9.3347427,9.3346439,9.3344352,9.3342187,9.3339173,9.3337801,9.3336191,9.3334408,9.3327883,9.332488,9.3322723,9.3313915,9.33076,9.3295092,9.3293703,9.3287867,9.3287308,9.3281502,9.328104,9.3272086,9.3262182,9.3251674,9.3228075,9.3221911,9.3215421,9.320981,9.3207986,9.3200656,9.319766,9.317483,9.3153864,9.3143897,9.3110682,9.3064315,9.3018844,9.301007,9.3001457,9.2991934,9.2981018,9.2980743,9.2971314,9.2954791,9.2951577,9.2944595,9.2935668,9.291556,9.2914171,9.2897413,9.2890891,9.2889121,9.2887491,9.2871126,9.2870454,9.2853883,9.2848244,9.2841586,9.2840538,9.2839256,9.2837082,9.2831841,9.28274429772661,9.28247780467719,9.28228144137763,9.28187468882851,9.28135572867965,9.28075261283097,9.28024767863208,9.27971469253325,9.27908352478463,9.2787328760354,9.27856456463577,9.27866274628555,9.278606],"lat":[53.314912,53.3148484847608,53.3147876127377,53.3146678,53.3145364,53.3144107,53.3141746,53.3139762,53.3137581,53.3136774,53.3135864,53.3134945,53.3131959,53.3130741,53.3129802,53.312505,53.3121389,53.3114142,53.3113337,53.31099,53.3109568,53.3106095,53.3105846,53.3100683,53.3094846,53.3088761,53.3075034,53.3071633,53.3068723,53.3066745,53.3066049,53.3063995,53.3063208,53.3057249,53.3051954,53.3049359,53.3040711,53.3028654,53.3016878,53.3014679,53.3012355,53.3009688,53.3006341,53.3006257,53.3003394,53.2998399,53.2997432,53.2995291,53.2992516,53.2986289,53.2985859,53.2980874,53.2978936,53.2978377,53.2977885,53.2972826,53.2972624,53.296769,53.296613,53.2964373,53.2964117,53.2963873,53.2963506,53.2962722,53.296034020075,53.2941645615108,53.2940471936696,53.294022043376,53.2940723439484,53.2942567788741,53.2945082797621,53.2946843295027,53.2947933123118,53.2948519952477,53.2948436119761,53.2952711567309,53.295483]}]],[[{"lng":[9.373245,9.37322826918041,9.37319727199731,9.3729354,9.3725954,9.3724351,9.3723068,9.372184,9.3720749,9.3718297,9.3715628,9.3710243,9.37086688183151,9.370502,9.3698665,9.3675747,9.3653553,9.3642284,9.3631121,9.3620725,9.3616969,9.3615141,9.3612095,9.3598971,9.3596616,9.3593958,9.358705,9.3574664,9.3558501,9.3554684,9.354449,9.3534995,9.3531908,9.3515208,9.3498593,9.3487649,9.34750364589164,9.3454495,9.345363,9.3444528,9.3426351,9.3423123,9.3420357,9.3411708,9.3409788,9.3404502,9.3404073,9.3400456,9.3394634,9.3378621,9.3369104,9.3364063,9.3362003,9.3360472,9.3358268,9.3355853,9.33516928082808,9.334932],"lat":[53.336027,53.3359962344443,53.335943985219,53.3356263,53.3352608,53.3350885,53.3349623,53.3348748,53.3348142,53.334707,53.3346153,53.3344545,53.3344082963336,53.3343012,53.3341208,53.3334642,53.3328111,53.3324919,53.332162,53.3318475,53.331715,53.3316236,53.3314308,53.3303992,53.3302449,53.3300834,53.3296804,53.3289706,53.3281019,53.3278968,53.3273264,53.3267833,53.3266125,53.3256756,53.3247409,53.3241178,53.3234087773785,53.322254,53.3222068,53.3216986,53.3206642,53.3204805,53.3203269,53.3198467,53.3197286,53.3194331,53.3194091,53.3192068,53.3188729,53.3179624,53.3174212,53.3171481,53.3170275,53.3169167,53.3164573,53.3159947,53.315268906321,53.314912]}]],[[{"lng":[9.398899,9.39870058614849,9.39843318464667,9.3980717,9.3979082,9.3975088,9.3971127,9.395699,9.3949427,9.3938958,9.3937236,9.3918868,9.3911034,9.38823846147598,9.3878882,9.3868096,9.3849192,9.3847324,9.3838668,9.3837167,9.3824417,9.3817927,9.3811456,9.379195,9.3785526,9.37838495756261,9.3770303,9.376501,9.376299,9.3758858,9.3757791,9.3753939,9.3748113,9.3737265,9.3735143,9.3733696,9.373245],"lat":[53.346258,53.3462746210852,53.3462225458669,53.3461192,53.3460693,53.3459513,53.3458112,53.345223,53.3449022,53.3444429,53.344376,53.3436106,53.3432852,53.3420857451867,53.3419391,53.3414778,53.3406991,53.3406314,53.3403178,53.340265,53.3398551,53.3396565,53.3394698,53.3388598,53.3386577,53.3386061399883,53.3381895,53.338019,53.3379535,53.3378117,53.3377725,53.337603,53.3373191,53.3367198,53.3365058,53.3362497,53.336027]}]],[[{"lng":[9.426945,9.42705729696038,9.42655236276149,9.42507963801472,9.4245465,9.42380327656751,9.4228074341197,9.42213418852117,9.4209197,9.4206999,9.4204769,9.4203125,9.4200607,9.4194554,9.4189052,9.4180696,9.4172376,9.4155924,9.4150742,9.4145373,9.4139164,9.41346,9.40841680945123,9.40698616255436,9.4051908409583,9.4020305,9.4010131,9.4007863,9.4005183,9.4000756,9.4000574,9.3999801,9.3998689,9.3996694,9.3992747,9.39898659224488,9.398899],"lat":[53.365284,53.3651297430717,53.3639245220963,53.3641839822693,53.3646648,53.3648786581799,53.3648702886581,53.364451810468,53.3636966,53.3635452,53.3633283,53.363162,53.3629419,53.3623954,53.3618389,53.3610611,53.3602779,53.3586899,53.3582245,53.3577906,53.3574008,53.3571465,53.3525820577186,53.3519457858035,53.3515271806797,53.3515995,53.3502821,53.3498553,53.3492824,53.3482549,53.3474392,53.3471369,53.346996,53.3468349,53.3465395,53.346352755891,53.346258]}]],[[{"lng":[9.465251,9.4649287718509,9.46368875279296,9.4613974,9.4581267,9.4536552,9.4530482,9.44971,9.446154,9.4430594,9.4373442,9.4345286,9.4335824,9.43185305552393,9.43098511456417,9.430928,9.4303497,9.4300497,9.4296651,9.4289529,9.4282746,9.4280944,9.4278769,9.4275959,9.4273228,9.427004,9.426945],"lat":[53.378304,53.3781783625478,53.3777878207096,53.3771011,53.3762574,53.3750404,53.3748332,53.3737895,53.3726444,53.3717229,53.3700465,53.3692305,53.3689379,53.368454331369,53.3681054100662,53.3680543,53.3675942,53.3673705,53.3670802,53.3666194,53.3661619,53.3660458,53.3659123,53.3657165,53.3655448,53.3653463,53.365284]}]],[[{"lng":[9.549767,9.54959425168456,9.54920392265072,9.54745565263786,9.546555,9.54578553292418,9.545354,9.5419381,9.5410015,9.53217002622787,9.5153469,9.514266,9.513562,9.5124814,9.51152088360561,9.5104086,9.5100507,9.508676,9.5079752,9.5072868,9.506594,9.5060823,9.5055535,9.5045982,9.5035551,9.5030715,9.5023784,9.4990633,9.4974484,9.4965969,9.49624703536197,9.4957249,9.4951085,9.4948495,9.494651,9.4944288074056,9.4942476,9.492428,9.4916717,9.4914152,9.49141465007653,9.4911547,9.4904453,9.4893981,9.4888463,9.4882057,9.4880992,9.4876197,9.4869085,9.4868039,9.4861071,9.4853447,9.485282,9.4850463,9.4835387,9.4834292,9.4818014,9.47995,9.4796854,9.4794271,9.4789343,9.4784862,9.4762307,9.4747173,9.4713858,9.4712686,9.4711506,9.4706279,9.4685746,9.4682217,9.468015,9.4676683,9.4669908,9.4664115,9.465833,9.465251],"lat":[53.414627,53.4145529238833,53.4144144872684,53.4138703333587,53.41359,53.4133397474239,53.4131994,53.4121268,53.4119428,53.4104553314664,53.4076217,53.4073425,53.4070661,53.4066418,53.4062254457497,53.4057433,53.4055881,53.4050491,53.4047948,53.4046134,53.4044911,53.4044222,53.4043699,53.4042917,53.4042131,53.4041752,53.4041148,53.4037278,53.403531,53.4033665,53.4032662751916,53.4031167,53.4028707,53.4027348,53.4026199,53.4024747644839,53.4023564,53.401072,53.4004338,53.4002174,53.4002167153954,53.3998931,53.3990097,53.3977814,53.3971292,53.3963421,53.3962114,53.3956231,53.3947569,53.3946295,53.3937808,53.3928825,53.3928196,53.3926558,53.3916086,53.3915325,53.3904015,53.389143,53.3889631,53.3887915,53.3884197,53.3880789,53.3859056,53.3844381,53.3812013,53.3810927,53.3810051,53.380675,53.379512,53.3792924,53.3791848,53.379045,53.3788159,53.3786292,53.3784428,53.378304]}]],[[{"lng":[9.68523,9.68511670539378,9.68492982966922,9.6843515,9.6827387,9.6824895,9.6822365,9.6819348,9.68132585412475,9.6812631,9.6808414,9.6806143,9.6803739,9.6799359,9.6786342,9.6779266,9.67772295111862,9.67759762003358,9.6774722,9.6774058,9.6772254,9.677125,9.6755867,9.6755332,9.6752836,9.6748912,9.6728757,9.6723241,9.6720585,9.671833,9.6700363,9.6690796,9.6686664,9.6683156,9.6680193,9.6675549,9.6673662,9.6667675,9.6665064,9.6656595,9.6652676,9.6643312,9.6640269,9.6636733,9.6628186,9.6627531,9.6626455,9.6622896,9.6617098,9.6610951,9.6609348,9.6608892,9.660842,9.6595948,9.659325,9.6591327,9.65738519074302,9.6569378,9.6566172,9.655453,9.6543621,9.65368980318985,9.6501615852236,9.64961412855881,9.64863193498964,9.64824495343225,9.64807409339908,9.64737203637629,9.64658057388449,9.64611137503289,9.64587777714997,9.64581126024423,9.64561247556666,9.64106586248814,9.63890443162603,9.63770676336864,9.63562854953036,9.63421903531841,9.63120578689681,9.63085510837634,9.62946064125195,9.62773098666798,9.62748828919914,9.62699145291042,9.62621170481019,9.62571612296165,9.62488219263891,9.62411263382919,9.62367879785179,9.62320631659805,9.62269482669901,9.62232218391653,9.62147520325342,9.6212005233097,9.61960907058225,9.61765906685863,9.61443596031484,9.61346053666825,9.61181378207602,9.61176038099225,9.60902897719129,9.60758782516566,9.60509424268922,9.60413137093594,9.60315252890264,9.60201044893192,9.60004433280091,9.59641296366956,9.59525423625229,9.59453039192489,9.59275295936387,9.59022928343444,9.58826490366199,9.58817579037504,9.58718942548974,9.58684565568782,9.58561465025777,9.58520694431524,9.58492538903809,9.58428501427974,9.5841083,9.58353571982899,9.5820447,9.57935632941583,9.5781585,9.5744532,9.57413251271752,9.5728596,9.5710198,9.5704413,9.56877645465016,9.5677668,9.5634251,9.5626259,9.5594904230188,9.5573861,9.5569862,9.5565959,9.55634050156367,9.5562121,9.55600392263607,9.5558754,9.5554043,9.55522965529618,9.5550882,9.55459644854342,9.5541753,9.5541304,9.5537934,9.5535351609305,9.5534984,9.5529525,9.5526079,9.5521492,9.5514083,9.5511991,9.55009633631431,9.54996448996556,9.549767],"lat":[53.468829,53.4687759870364,53.468706444658,53.4685539,53.468138,53.4680657,53.467972,53.4678398,53.4674784396912,53.4674412,53.4672253,53.467109,53.4670099,53.4668819,53.466612,53.4664798,53.4664391957355,53.4664142067426,53.4663892,53.4663752,53.4663371,53.4663207,53.466059,53.4660513,53.4660019,53.4659186,53.4655246,53.4654067,53.4653502,53.4653025,53.4648941,53.4646366,53.4644928,53.4643707,53.4642634,53.4641031,53.4640353,53.4638364,53.46376,53.4634973,53.4633496,53.4629967,53.4628417,53.4626617,53.4621439,53.4621042,53.462036,53.4618095,53.4614406,53.4610653,53.4609748,53.4609467,53.4609176,53.4601494,53.4599846,53.4598672,53.4588359267288,53.4585719,53.4583827,53.4576613,53.4570103,53.4566032260308,53.4544668352531,53.4541353321073,53.4537219903941,53.4535591337943,53.4534872291766,53.4531917747087,53.4528586922778,53.4526612314103,53.4525536965075,53.4525230759362,53.4524315667315,53.4503385098008,53.4495253266908,53.449074727921,53.4479401624794,53.4471706449601,53.4457195269446,53.4455506442138,53.4447911697912,53.4438491219645,53.4437169363155,53.4435571631795,53.4433064097543,53.4431470384833,53.4428788583539,53.4426313776856,53.4424918607244,53.4423399153388,53.4421754245389,53.4420403571883,53.4417333606902,53.441633799758,53.4410569546631,53.4403501368884,53.4390547823776,53.4386627554644,53.4380921236763,53.4380736190567,53.4371271183786,53.4366277141616,53.4354657456383,53.4350170546956,53.4347646679281,53.4346102296357,53.4344583772802,53.4342598782332,53.4340985263067,53.4339079620283,53.4332130415947,53.431801353442,53.4299718231625,53.4298888254005,53.4288903728804,53.4285423847611,53.4279549842182,53.4277604368272,53.4276260849554,53.4274073885684,53.4273491,53.4271592584246,53.4266649,53.42579430689,53.4254064,53.4241208,53.4240155951256,53.423598,53.4229784,53.4227892,53.4222352497201,53.4218993,53.4204464,53.4201808,53.4191278107649,53.4184211,53.4182757,53.418112,53.4179900237112,53.4179287,53.4178234057381,53.4177584,53.4174565,53.4173376577562,53.4172414,53.4168581919418,53.41653,53.416497,53.4162493,53.4160821887303,53.4160584,53.4157543,53.4155902,53.4154024,53.4151284,53.415053,53.4146953627955,53.4146636001092,53.414627]}]],[[{"lng":[9.68523,9.68561363088715,9.68642852370572,9.687005,9.6874282,9.6879931,9.6887152,9.6891267,9.6892017,9.6893753,9.68969648910201,9.689763],"lat":[53.468829,53.4688521838624,53.4689741506244,53.4690814,53.4691747,53.4693294,53.4696393,53.4697861,53.46981,53.4698653,53.4700893486894,53.470173]}]],[[{"lng":[9.689763,9.6901408,9.6902229,9.690245,9.6903745,9.6904701,9.6906035,9.6908626,9.6910414,9.6910609,9.6910904,9.6914334,9.691759,9.6921269,9.6922955,9.6924616,9.6925634,9.6933301,9.6940884,9.6941578,9.6942262,9.6943739,9.6945307,9.6946436,9.6947661,9.6948713,9.6949382,9.6950852,9.6954417,9.6954822,9.6956423,9.695729,9.6957799,9.6960265,9.6960524,9.6964603,9.6965168,9.6965884,9.6967965,9.696934,9.6970539,9.6977576,9.6978379,9.6979164,9.698046,9.6984258,9.6990435,9.7000891,9.7001062,9.70031984201682,9.700347],"lat":[53.470173,53.4705519,53.470647,53.4706726,53.4707966,53.4708783,53.4709945,53.4712202,53.471404,53.4714219,53.4714408,53.4717848,53.4719808,53.4722069,53.4723105,53.4724066,53.4724655,53.4729291,53.4734011,53.4734382,53.4734739,53.4735607,53.4736528,53.4737218,53.4737975,53.4738627,53.4739052,53.4739981,53.4742256,53.4742512,53.4743525,53.4744069,53.4744387,53.4745912,53.474607,53.4748558,53.4748838,53.4749254,53.4750325,53.4750984,53.475161,53.4754094,53.4754343,53.4754571,53.4754934,53.4755999,53.4758356,53.4762378,53.4762481,53.4763578587173,53.476376]}]],[[{"lng":[9.700347,9.70036581927904,9.70041371264451,9.7010497,9.7011359,9.7014693,9.7015047,9.70198580165638,9.70208818146746,9.702158],"lat":[53.476376,53.4763853456869,53.4764110309629,53.4767932,53.4768411,53.4770874,53.4771067,53.47730202438,53.4773564806571,53.477427]}]],[[{"lng":[9.702158,9.70208818146746,9.70206794872633,9.70218934517314,9.70240178895506,9.70284690926004,9.70325156408275,9.70337296052957,9.7033931932707,9.70345389149411,9.70363598616433,9.70448576129202,9.70527483819631,9.70594251865378,9.70642810444103,9.70701485393396,9.70788486180279,9.70826928388436,9.70847161129572,9.70871440418935,9.70966534302272,9.71047465266814,9.71156722068945,9.71185047906535,9.71225513388806,9.71274071967532,9.71332746916825,9.71359049480301,9.71367142576755,9.71379282221436,9.71379282221436,9.71387375317891,9.7141570115548,9.71510795038817,9.71555307069315,9.7163219148563,9.71686819886696,9.71745494835989,9.71794053414714,9.71838565445212,9.71889147298051,9.71972101536707,9.72036846308341,9.72055055775362,9.72135986739904,9.7218454531863,9.72241196993809,9.72312011587784,9.72441501131051,9.72524455369707,9.72585153593113,9.72619549253043,9.72649898364747,9.72660014735315,9.72651921638861,9.72613479430703,9.7261752597893,9.72635735445952,9.72666084557655,9.72710596588153,9.72757131892765,9.72795574100923,9.72908877451281,9.72963505852348,9.72993854964051,9.73024204075754,9.73066692832139,9.7311322813675,9.73123344507318,9.73117274684977,9.73127391055545,9.73139530700226,9.73171903086043,9.7320427547186,9.73242717680018,9.73307462451651,9.73347927933922,9.73364114126831,9.73374230497398,9.73360067578603,9.73339834837468,9.73309485725765,9.7323260130945,9.73109181588523,9.73072762654479,9.7305050663923,9.73044436816889,9.73054553187457,9.7308490229916,9.73103111766182,9.73103111766182,9.73076809202706,9.73070739380366,9.73074785928593,9.73082879025047,9.73082879025047,9.73068716106252,9.73044436816889,9.73010041156959,9.72943273111212,9.72856272324329,9.72821876664399,9.72779387908014,9.72744992248084,9.72724759506948,9.7270857331404,9.72706550039926,9.72706550039926,9.72700480217586,9.7268024747645,9.72668107831769,9.72653944912974,9.72655968187087,9.72666084557655,9.72678224202336,9.72694410395245,9.72747015522197,9.72795574100923,9.7286841196901,9.72912923999508,9.72961482578234,9.72999924786391,9.73038366994549,9.73068716106252,9.73143577248453,9.73254857324699,9.73289252984629,9.73311508999878,9.73319602096332,9.73303415903424,9.73303415903424,9.73309485725765,9.733297184669,9.73366137400944,9.73426835624351,9.73633209583933,9.73801141335358,9.73867909381105,9.73922537782171,9.73993352376145,9.74047980777211,9.74080353163028,9.74122841919413,9.74149144482889,9.74187586691046,9.74246261640339,9.74284703848497,9.74353495168357,9.74428356310559,9.74517380371555,9.74592241513757,9.74687335397094,9.74816824940361,9.74956430854196,9.75057594559873,9.75124362605621,9.7520934011839,9.75330736565203,9.75504738138968,9.75640297504576,9.7576776377373,9.75860834382954,9.75966044636858,9.7607530143899,9.76172418596441,9.76295838317367,9.76356536540774,9.76425327860635,9.76484002809928,9.76556840678015,9.76593259612059,9.76653957835466,9.76698469865964,9.76734888800008,9.76771307734052,9.76807726668096,9.7687854126207,9.76925076566682,9.77003984257111,9.77093008318107,9.77173939282649,9.77279149536554,9.77345917582301,9.77390429612799,9.77447081287978,9.7749361659259,9.7752801225252,9.7755633809011,9.77566454460678,9.77532058800748,9.77536105348975,9.77568477734791,9.77742479308557,9.77811270628418,9.77902317963527,9.78027760958568,9.78151180679495,9.78317089156806,9.78428369233051,9.78565951872773,9.78626650096179,9.78707581060721,9.78778395654696,9.78863373167465,9.78984769614278,9.79069747127047,9.79496657965006,9.79541169995504,9.79460239030962,9.7940763390401,9.79431913193372,9.79484518320325,9.795553],"lat":[53.477427,53.4776063373574,53.4777207290906,53.4779434910545,53.4781843134837,53.4785937084765,53.4790030995191,53.4794606495393,53.4799422758089,53.4803757347771,53.4807128664685,53.4815075234274,53.4823864448491,53.4834700215388,53.4840599572114,53.4845294919854,53.4852518430283,53.4856852477497,53.4860704926735,53.4864918503028,53.4874669761928,53.4880929711219,53.4886346900459,53.488935642014,53.4895014259331,53.4903079559186,53.4909218714477,53.4913311435302,53.4916802254206,53.4922218985275,53.493088561108,53.4934376285327,53.4937987297416,53.4943163027788,53.4944968500265,53.494629250853,53.4947977603978,53.4949783055957,53.4952190313304,53.495628261943,53.496013416559,53.4964226395052,53.4966392853485,53.4968800016538,53.4979872790573,53.4982761292765,53.4984927656496,53.4986371892834,53.4986732951149,53.4987695771822,53.498913999873,53.499070457233,53.4994315104755,53.4997564557649,53.5001175031655,53.5008756926989,53.5012006269197,53.5014894552473,53.5016459031032,53.5017060752018,53.5016338686733,53.5014653862952,53.5009479005188,53.5008516233983,53.5008516233983,53.5009479005188,53.5012487651109,53.5020550716891,53.50253644144,53.503053907826,53.503787977212,53.5041850914175,53.5047266047933,53.505063542958,53.5052801446497,53.5055328452244,53.5056892781586,53.5058938434322,53.5061585734972,53.5066398966522,53.5068805561801,53.5070610499293,53.5072896742416,53.5074942317916,53.5076145593009,53.5077950499245,53.5080116376582,53.5082522893976,53.5086252968934,53.5089742364175,53.5093472375596,53.5101654221152,53.5105504446758,53.5109595273139,53.5114889225134,53.5118017438409,53.5120664370069,53.5122950343197,53.5124995677123,53.512692068827,53.5127401939691,53.5127401939691,53.5127883190565,53.5129086315359,53.5131131619669,53.5133658158423,53.5137628403174,53.5141959536842,53.5145568781083,53.5150140446323,53.5153990231427,53.5161208484223,53.5166381489753,53.5170110826655,53.5172998032681,53.5175043125045,53.5177930297462,53.5179614472285,53.5181659532714,53.5183824880056,53.5186591696666,53.5190080265768,53.5194290569534,53.5198621124042,53.5211131366191,53.5225084662001,53.5230497454753,53.5236752151353,53.5241683673941,53.5253831813936,53.5261770014227,53.526465659563,53.5266821518771,53.5268024249069,53.5268505340232,53.5268745885608,53.5268745885608,53.5269467520918,53.5270670243701,53.5274278391561,53.5279450016556,53.5285824258155,53.529869271772,53.5302901942055,53.5305547718796,53.5309516352914,53.5313605209774,53.5317212991865,53.5319257388073,53.531997893732,53.5319858679198,53.5318776354559,53.5316250919646,53.5313605209774,53.5312402608914,53.5312402608914,53.5313003909771,53.5314687547629,53.5318054803264,53.5321301774412,53.5322143577683,53.5321422032124,53.5319497904625,53.5317092732957,53.5314206508924,53.5311681046758,53.5310839222689,53.5310839222689,53.5311560786279,53.5312643129359,53.5313725469672,53.5314447028345,53.5313965989366,53.5313003909771,53.5311681046758,53.5309275830693,53.5305186932032,53.5301097993891,53.5293401062056,53.5287868805895,53.5283539163708,53.5280532441703,53.5279690555717,53.5279690555717,53.5280652710993,53.5282817552373,53.5285824258156,53.529027414354,53.5295686103295,53.5313965989366,53.5318535837598,53.532298537928,53.5336814737382,53.5341624843029,53.534451088019,53.5345352637323,53.5345232386406,53.5339941312267,53.5335732456081,53.5332245086406,53.5332124831767,53.5332846359088,53.5335131187496,53.5339340049659,53.5348118398978,53.5356054832127,53.5407998721194,53.5417617260551,53.5437334583012,53.5455127475876,53.5483017532506,53.5512829009002,53.555098]}]],[[{"lng":[9.936447,9.92633270965771,9.88910446596836,9.87235175630815,9.85139063649175,9.795553],"lat":[53.541838,53.5414851952869,53.5420142090918,53.5433126694939,53.5459094708117,53.555098]}]],[[{"lng":[9.98846,9.98824489753239,9.98797175552707,9.98743558788697,9.98646441631247,9.98492672798617,9.98402637100564,9.98336880691874,9.98277699924052,9.98223577341515,9.98185135133357,9.97968644803207,9.97669200234402,9.97367732391482,9.96558422746061,9.96153767923351,9.95765299293549,9.9524734112048,9.94349007414063,9.936447],"lat":[53.54469,53.5445329455108,53.5444367673974,53.5443225556039,53.5442684551731,53.5441903099843,53.5441211814278,53.5440099743826,53.5438356492373,53.5436012108411,53.5434930080664,53.5434569404133,53.5436132333545,53.5439618847591,53.5443946894041,53.5444668230813,53.5441782876347,53.543360759855,53.5424470336513,53.541838]}]],[[{"lng":[9.98846,9.9882402313431,9.98830173642022,9.9883939940359,9.98853238045942,9.9887784007679,9.98907054988422,9.98939345153909,9.98978554640573,9.99011998026257,9.99144233942064,9.99266859689571,9.99297612228131,9.9932644273303,9.99341434595578,9.993701],"lat":[53.54469,53.5450227295616,53.5453653446691,53.5455709124024,53.5457262295832,53.5458952505736,53.5460048854496,53.546045998455,53.5460505665642,53.5460437144001,53.5459500680471,53.5458701258744,53.5458175923645,53.5460322941243,53.5462332905294,53.547316]}]],[[{"lng":[9.993701,9.99392176284202,9.9931],"lat":[53.547316,53.5479120300428,53.54798]}]]],null,null,{"interactive":true,"className":"","stroke":true,"color":"#03F","weight":5,"opacity":0.5,"fill":false,"fillColor":"#03F","fillOpacity":0.2,"smoothFactor":1,"noClip":false},null,null,null,{"interactive":false,"permanent":false,"direction":"auto","opacity":1,"offset":[0,0],"textsize":"10px","textOnly":false,"className":"","sticky":true},null]}],"limits":{"lat":[52.0917582981661,53.555098],"lng":[4.8924615925118,9.99392176284202]}},"evals":[],"jsHooks":[]}</script>

Plot this in 3d to see the difference in elevation:

``` r
line_to_draw$slope = slope_raster(line_to_draw, e = all_elevation)


line_to_draw_3d = slope_3d(line_to_draw, all_elevation)

plot_slope(line_to_draw_3d, title = "Elevation Data, Utrecht to Hamburg")
```

<img src="{{< blogdown/postref >}}index.en_files/figure-html/unnamed-chunk-47-1.png" width="672" />

Perhaps more useful would be to incorporate the slope data in figuring out the total journey lengths, by devising a formula and using it in the calculations above, but this is still an interesting application!

# Conclusions

Viabundus is an amazing resource, and sfnetworks makes it super easy to import and work with the data. The project team should be immensely proud of the result, and I’m very grateful to them for releasing it as open access data.

Converting the files into a ‘tidy’ data structure where necessary helped me to more easily sort and filter results, which might be worth keeping in mind.

I can imagine this having lots of really interesting applications to understanding postal routes, itineraries, and so forth. Another way to use the data would be to link to geo-temporal events on Wikidata (such as battles and sieges), and estimate the routes travelled by individuals in EMLO by their letter dates and locations. This could be a good way of figuring out intermediate stops travelled, and through that the likelihood of them being caught up in certain events.

I’m also hoping to figure out how to correctly exclude certain nodes (or node types) when running the pathing algorithm (rather than just removing those nodes from the itinerary afterwards), perhaps by setting the weight of certain edges to Inf. This would make it possible to estimate a route which avoided tolls and staples, for example.

Hopefully, some day, we’ll have similar data for most of Europe. I look forward to further updates from the project!

------------------------------------------------------------------------
