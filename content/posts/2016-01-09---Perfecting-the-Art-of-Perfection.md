---
title: Querying Openstreetmaps Using Overpass
date: "2019-03-14T11:46:37.121Z"
template: "post"
draft: false
slug: "/posts/querying-penstreetmaps-using-overpass/"
category: "Tutorials"
tags:
  - "openstreetmaps"
  - "gis"
  - "overpass"
description: "The Overpass API is a powerful API that lets you query data from OpenStreetMap.This is a simple turorial to query and get data from openstreet maps"
---

The Overpass API is a powerful API that lets you query data from OpenStreetMap. You can find different places, routes to locations, and everything under the sun, quite literally! You can ‘talk’ to it and request data for your own specific use, using its own language, the “Overpass Query Language” (QL).

The Overpass API serves up custom selected parts of the OSM map data.

 It acts as a database over the web: the client sends a query to the API and gets back the data set that corresponds to the query. 

https://overpass-turbo.eu/


![Overpass Turbo. Caption: “[Overpass Turbo](https://overpass-turbo.eu/)”](./overpass-turbo.png)


## OSM Data Model

All features in OSM are either nodes, polylines, polygons or relations 
  - Nodes – single point -geo coordinate + tags 
  - Polylines – an ordered list of nodes + tags (start and end node are different) 
  - Polygons – an ordered list of nodes + tags (start and end node are the same) 
  - Relations – a logical grouping of nodes, polylines and polygons + tags (example – country boundary)
    In OSM terminology the term WAY is used to describe either polylines or polygons



Tagging is one of the most important aspects of OpenStreetMap 
Tagging in OpenStreetMap is very flexible. An object can have at minimum 1 attribute (tag) – and there is no upper limit.  There are no strict rules on how many attributes any object should have.

This flexibility means that you must use your own judgement in regards to how many attributes you provide

One tag is mandatory. In this case the most appropriate is amenity = toilets All the others are not mandatory

All the tags can be found in taginfo, with statistical information: https://taginfo.openstreetmap.org/

---

I hope now you know everything about how openstreetmaps models its data. Lets now Dive in

---

## Overpass Query Language

Overpass QL has a c like syntax and is imperative. A query is divided into statements and each statement is terminated by a semicolon

# Execution state
The Overpass QL execution state consists of:

  - the default set _ (a single underscore)
  - other named sets, if created by the user
  - a stack, during the execution of Overpass QL block statements



# statements

Statements are evaluated sequentially and change the execution state according to their semantics.

There are several different types of statement. You almost always need the print statement, which is called an action, because it has an effect outside the execution state (the output). The other statements are grouped into

- Standalone queries: These are complete statements on their own. 
- Filters: They are always part of a query statement and contain the interesting selectors and filters. Block statements: They group statements and enable disjunctions as well as loops. 
- Settings: Things like output format that can be set once at the beginning.


# Sets

Overpass QL manipulates sets. Statements write their results into sets, and sets are then read by subsequent statements as input. An Overpass QL set can contain any combination of – and any number of – OpenStreetMap nodes, ways, relations, and area elements.

Unless you specify a named set as an input or result, all input is implicitly read from (and all results are written to) the default set named _ (a single underscore). Once a new result is (implicitly or explicitly) assigned to an existing set, its previous contents will be replaced and are no longer available. Overpass QL sets always have global scope (visibility).




# 1. Block Queries 

## 1.1 Difference 

The difference block statement is written as a pair of parentheses. Inside the difference statement, exactly two statements must be placed, and between them a minus sign.

    (statement_1; - statement_2;)[->.result_set];

It takes no input set. It produces a result set. Its result set contains all elements that are result of the first sub-statement and not contained in the result of the second sub-statement.

Example:

    (node[name="Foo"]; - node(50.0,7.0,51.0,8.0););

This collects all nodes that have a name tag "Foo" but are not inside the given bounding box.

The result set of the difference statement can be redirected with the usual postfix notation:

Example:

    (node[name="Foo"]; - node(50.0,7.0,51.0,8.0);)->.a;

Same as the preceding example, but the result is written into the variable a.


  (
  node[amenity]( 47.06, 15.435,  47.07,  15.44);
  -node[amenity](47.064, 15.435, 47.066, 15.44);
  );
  out;



## 1.2 For-each loop (foreach)

The foreach statement loops over the content of the input set, once for every element in the input set.

Example:

    foreach.a(...);

This loops over the content of set a instead of the default set "\_".

The name of the variable to put the loop element into can also be chosen by adding a postfix immediately before the opening parenthese.

Example:

    foreach->.b(...);

This puts the element to loop over into the variable b. Without it, the foreach statement does not puts the elements into any set. Example for both input and loop set changed:

    foreach.a->.b(...);


    // get all bank nodes in coordinates
    (
      node[amenity=bank]
      (47.0678,15.4401658,47.069,15.4501658);
      >;
    );
    // foreach bank node, print out adjacent nodes
    foreach->.bank_set(
      node(around.bank_set:15)->.adjacent_set;
      (.adjacent_set);
      out meta;
    );



## 1.3 Union

The union block statement is written as a pair of parentheses. Inside the union, any sequence of statements can be placed, including nested union and foreach statements.

    (statement_1; statement_2; …)[->.result_set];

It takes no input set. It produces a result set. Its result set is the union of the result sets of all sub-statements, regardless of whether a sub-statement has a redirected result set or not.

Example:

    (node[name="Foo"];way[name="Foo"];);

This collects in the first statement all nodes that have a name tag "Foo" and in the second statement all ways that have a name tag "Foo". After the union statement, the result set is the union of the result sets of both statements.

The result set of the union statement can be redirected with the usual postfix notation:

Example:

    (node[name="Foo"];way[name="Foo"];)->.a;

Same as the preceding example, but the result is written into the variable a.

Note: foreach and print statements cannot be subelement of element union.


    (
    node
      [amenity=drinking_water]
      (47.06,15.42,47.09,15.48);
    node
      [tourism=hotel]
      (47.06,15.42,47.09,15.48);
    );
    out;



# 2 Standalone Queries

## 2.1 Item 
The item standalone query consists only of an input set prefix.

It takes the input set specified by its prefix. This is in particular useful for union statements: it reproduces its input set as (part of the) result of the union statement.

The most common usage is the usage with the default input set:

      ._;

In the context of a union statement, the following will return all items in the default inputset along with the recurse down result.

      (._; >;);

But of course other sets are possible too:

      .a;

In the context of a union statement:

      (.a; .a >;);

Note: Subsequent statements in a union statement are not impacted by the item statement. In particular `.a;` does not add the contents of the input set to the default item set ._

The item statement can also be used as filter.


    node[amenity=bank]
      (47.06,15.42,47.09,15.48);
    )->.bank_set;

    //here we print the bank_set by passing the .bank_set query
    .bank_set out;



# 3 Actions 

### Print (out)

The out action can be configured with an arbitrary number of parameters that are appended, separated by whitespace, between the word out and the semicolon.

The out action takes an input set. It doesn't return a result set. The input set can be changed by prepending the variable name.

Allowed values, in any order, are:

- one of the following the degree of verbosity; default is body:
    - _ids_: Print only the ids of the elements.
    - _skel_: Print also the information necessary for geometry. These are also coordinates for nodes and way and relation member ids for ways and relations.
    - _body_: Print all information necessary to use the data. These are also tags for all elements and the roles for relation members.
    - _tags_: Print only ids and tags for each element and not coordinates or members.
    - [_meta_](#docs-tab-meta): Print everything known about the elements. This includes additionally to body for all elements the version, changeset id, timestamp and the user data of the user that last touched the object.

- one of the following modificators for derived information:
    - [_bb_](#docs-tab-bounding_box): Adds the bounding box of each element to the element. For nodes this is equivalent to "geom". For ways it is the enclosing bounding box of all nodes. For relations it is the enclosing bounding box of all node and way members, relations as members have no effect.
    - [_center_](#docs-tab-center): This adds the center of the above mentioned bounding box to ways and relations. Note: The center point is not guaranteed to lie inside the polygon (example).
    - _geom_: Add the full geometry to each object. This adds coordinates to each node, to each node member of a way or relation, and it adds a sequence of "nd" members with coordinates to all relations.

The attribute "geom" can be followed by a bounding box in the format "(south,west,north,east)". In this case only coordinates that are inside the bounding box are produced. For way segments also the first coordinate outside the bounding box is produced to allow for properly formed segments.

- One of the following for the sort order can be added. Default is asc.
    - asc: Sort by object id.
    - qt: Sort by quadtile index; this is roughly geographical and significantly faster than order by ids.
- a non-negative integer for the maximum number of elements to print. Default is no limit.


    (
      // Rio de Janeiro's Christ the Redeemer peak
      node[name=Corcovado][natural=peak];
      >;
    );
    out;
    ===meta===
    (
      // Rio de Janeiro's Christ the Redeemer peak
      node[name=Corcovado][natural=peak];
      >;
    );
    out meta;
    ===bounding box===
    (
      // Rio de Janeiro's Christ the Redeemer peak
      way(141708228);
      >;
    );
    out bb;
    ===limit===
    (
      // Rio de Janeiro's Christ the Redeemer peak
      way(141708228);
      >;
    );
    // only print 20 nodes from the way 
    out 20;
    ===center===
    (
      // Rio de Janeiro's Christ the Redeemer peak
      way(141708228);
      >;
    );
    // print center (not centroid) of the polygon
    out center;
    ===combination===
    (
      // Rio de Janeiro's Christ the Redeemer peak
      way(141708228);
      >;
    );
    // print multiple times using different modifiers
    out geom;
    out center;
    out bb;



# Filters 

## Bounding Box

The bbox-query filter selects all elements within a certain bounding box.

It has no input set. As for all filters, the result set is specified by the whole statement, not the individual filter.

    (south,west,north,east)


    {{< docs_repl >}}
node
(47.065,15.425,47.07,15.43); // a bbox-filter
out;
{{< /docs_repl >}}

## By Element Id

The id-query filter selects the element of given type with given id.


    // get node with id 1170494282
    // put it (implicitly) in the default set
    node(1170494282);
    // print the default set
    out;


## By area (area)

The area filter selects all elements of the chosen type that are inside the given area.

    // search the area of the Dolmites
    area
      [place=region]
      ["region:type"="mountain_area"]
      ["name:en"="Dolomites"];
    out body;

    // get all peaks in the area
    node
      [natural=peak]
      (area);
    out body qt;



## By Tag

The has-kv filter selects all elements that have or have not a tag with a certain value. It supports the basic OSM types node, way, and relation as well as the extended type area.


    node["name"~"^Foo$"];    /* finds exactly "Foo" */
    node["name"~"^Foo"];     /* finds anything that starts with "Foo" */
    node["name"~"Foo$"];     /* finds anything that ends with "Foo" */
    node["name"~"Foo"];      /* finds anything that contains the substring "Foo" */
    node["name"~"."];        /* finds anything, equal to the previous variant */




## By date of change (changed)

The changed filter selects all elements that have been changed between the two given dates.

Example: All changes since the given date and now

    node._(changed:"2012-09-14T07:00:00Z");


Another Example:

    (node(poly:"50.73 7.13 50.73 7.17 50.75 7.15");>;);
    // This finds all nodes that have changed
    // between the two given dates 
    node._(changed:"2012-09-14T07:00:00Z","2012-09-14T07:01:00Z");
    out;



##  Relative to other elements (around)


The around filter selects all elements within a certain radius around the elements in the input set



Example: Find all cinemas in Bonn which are at most 100m away from bus stops

    area[name="Bonn"];
    node(area)[highway=bus_stop];
    node(around:100)[amenity=cinema];
    out;





## By Input Set

The "item" filter selects all elements from its input set.

    // get area Alpe and Cividale
    (area[name="Cividale del Friuli"])->.Cividale;
    (area[name="Julijske Alpe"])->.Alpe;

    (node[power=pole](area.Cividale))->.Cividale_nodes;
    (node[power=pole](area.Alpe))->.Alpe_nodes;

    // print out nodes that are present in both areas (intersection)
    node.Alpe_nodes.Cividale_nodes;
    out body qt;

    (relation[name="Cividale del Friuli"];>;);
    out body;
    out skel qt;

    (relation[name="Julijske Alpe"];>;);
    out body;
    out skel qt;




## newer 

The newer filter selects all elements that have been changed since the given date


    (node(poly:"50.73 7.13 50.73 7.17 50.75 7.15");>;);
    // This finds all nodes that have changed
    // since 14 Sep 2012, 7 h UTC, in the given input set.
    (node._(newer:"2012-09-14T07:00:00Z");>;);
    out;



## Area pivot (pivot)

The _pivot_ filter selects the element of the chosen type that defines the outline of the given area.

    // determine area for Greater London and store it to .London_area
    area[name="London"][admin_level=6][boundary=administrative]->.London_area;
    // convert back to relations using the pivot filter
    rel(pivot.London_area);
    // output the geom
    out geom;



## By polygon (poly)

The _polygon_ filter selects all elements of the chosen type inside the given bounding box.



  (node(poly:"50.73 7.13 50.73 7.17 50.75 7.15");>;);
  out;



## Recurse (n, w, r, bn, bw, br)

The recurse filter selects all elements that are members of an element from the input set or have an element of the input set as member, depending on the given parameter.


      node(w);        // select child nodes from all ways of the input set
      node(r);        // select node members of relations of the input set
      way(bn);        // select parent ways for all nodes from the input set
      way(r);         // select way members of relations from the input set
      rel(bn);        // select relations that have node members from the input set
      rel(bw);        // select relations that have way members from the input set
      rel(r);         // select all members of type relation from all relations of the input set
      rel(br);        // select all parent relations of all relations from the input set


You can also restrict the recurse to a specific role. Just add a colon and then the name of the role before the closing parenthesis.

Examples with default input set:

    node(r:"role");        // select node members of relations of the input set
    way(r:"role");         // select way members of relations from the input set
    rel(bn:"role");        // select relations that have node members from the input set
    rel(bw:"role");        // select relations that have way members from the input set
    rel(r:"role");         // select all members of type relation from all relations of the input set
    rel(br:"role");        // select all parent relations of all relations from the input set
    Example with modified input set:

    node(r.foo:"role");

And you can also search explicitly for empty roles:

    node(r:"");
    node(r.foo:"");









