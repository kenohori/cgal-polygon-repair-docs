The following is in response to the requirement of the Computational Geometry Algorithms Library (CGAL) Project in the program Google Summer of Code (GSoC) 2023 that I generate a final report on the achievements and obstacles I have encountered during the last fifteen weeks. This report also presents a summary of the work done during the program and how it can be developed in the future.

## 1. Background

Polygons are usually represented by one outer ring and possibly multiple inner rings representing holes, where each ring is usually represented as a sequence of points. This allows for several undesirable configurations, such as self-intersecting rings, badly nested rings (inner outside outer), and inner rings that split the interior of the polygon into multiple disjoint parts. Such undesirable configurations are considered as invalid polygons in many applications, such as in Geographic Information Systems through ISO 19107. Polygon repair methods aim to always return a valid set of polygons given a possibly invalid set of input polygons.

The aim of this project is to re-implement the polygon repair method originally described in [this paper](https://doi.org/10.1016/j.cageo.2014.01.009) and implemented as [prepair](https://github.com/tudelft3d/prepair) in CGAL. This method consists of three steps: (1) a constrained triangulation of the polygon, (2) labelling of its faces as interior/exterior, (3) extraction of the resulting polygon(s) from the sets of labelled triangular faces.

The previous implementation of the method already is partly based on CGAL, but this project is a reimplementation of the method with a new approach to label and reconstruct the multipolygons, as well as the elimination of external dependencies. It also incorporates improvements later added to prepair, such as the application of the odd-even counting heuristic to edges.

The implementation is described in this report and in the additions to the CGAL documentation, mostly in the newly created pages for the [Polygon repair](https://kenohori.github.io/cgal-polygon-repair-docs/Polygon_repair/index.html) package. However, multipolygons and their related documentation have been added to the [Polygon](https://kenohori.github.io/cgal-polygon-repair-docs/Polygon/index.html) package instead.

## 2. Work completed

### 2.1 Multipolygon support

CGAL already contains concepts and classes that are meant to deal with polygons and polygons with holes in the Polygon package. However, polygon repair methods will often have to split polygons to generate valid output, resulting in multipolygons.

A `MultiPolygonWithHoles_2` concept and its implementation into a `Multipolygon_with_holes_2` have thus been added to the Polygon package. These follow the logic of existing concepts and classes in the Polygon package for polygons and polygons with holes, such as their available iterators and output to `ostream`.

### 2.2 Special triangulation class

### 2.3 Polygon repair method using the odd-even heuristic

### 2.4 Tests

### 2.5 Documentation

## 3. Prospective development

The 

Only one repair heuristic (odd-even) has been implemented so far.

Functions in polygon package

## 4. Challenges and accomplishments

