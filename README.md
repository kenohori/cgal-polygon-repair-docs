This repository contains the final report and documentation produced for the Computational Geometry Algorithms Library (CGAL) Project in the Google Summer of Code (GSoC) 2023. It documents the achievements and obstacles that I have encountered during the last fifteen weeks. This report also presents a summary of the work done during the program and how it can be developed in the future.

## 1. Background

Polygons are usually represented by one outer ring and possibly multiple inner rings representing holes, where each ring is usually represented as a sequence of points. This allows for several undesirable configurations, such as self-intersecting rings, badly nested rings (inner outside outer), and inner rings that split the interior of the polygon into multiple disjoint parts. Such undesirable configurations are considered as invalid polygons in many applications, such as in Geographic Information Systems through ISO 19107. Polygon repair methods aim to apply a method to always return a valid set of polygons given a possibly invalid set of input polygons.

The aim of this project is to re-implement the polygon repair method originally described in [this paper](https://doi.org/10.1016/j.cageo.2014.01.009) and implemented as [prepair](https://github.com/tudelft3d/prepair) in CGAL. This method consists of three steps: (1) a constrained triangulation of the polygon, (2) labelling of its faces as interior/exterior, (3) extraction of the resulting polygon(s) from the sets of labelled triangular faces.

The previous implementation of the method already is partly based on CGAL, but this project is a reimplementation of the method with a new approach to label and reconstruct the multipolygons, as well as the elimination of external dependencies. It also incorporates improvements later added to prepair, such as the application of the odd-even counting heuristic to edges, which creates more robust results even when these are partially overlaping.

The implementation is described in this report and in the additions to the CGAL documentation, mostly in the newly created pages for the [Polygon repair](https://kenohori.github.io/cgal-polygon-repair-docs/Polygon_repair/index.html) package. However, multipolygons and their related documentation have been added to the [Polygon](https://kenohori.github.io/cgal-polygon-repair-docs/Polygon/index.html) package instead.

## 2. Work completed

### 2.1 Multipolygons

CGAL already contains concepts and classes that are meant to deal with polygons and polygons with holes in the Polygon package. However, polygon repair methods will often have to split polygons to generate valid output, resulting in multipolygons. The well-known text (WKT) reader/writer in CGAL supports multipolygons, but reads and writes these as ranges of polygons with holes, which is not ideal from a user perspective since a multipolygon is understood to be a single entity (with the same semantics and so on).

A `MultiPolygonWithHoles_2` concept and its implementation into a `Multipolygon_with_holes_2` have thus been added to the Polygon package. These follow the logic of existing concepts and classes in the Polygon package for polygons and polygons with holes, such as their available iterators and support for being sent as output to `ostream`. Additionally, support `Multipolygon_with_holes_2` for has been added to CGAL's WKT reader/writer and a function to draw multipolygons using `CGAL::draw` was created.

### 2.2 Triangulation classes

A custom class `Triangulation_with_odd_even_constraints_2` has been created to implement the odd-even counting on overlapping constraints. Using the method `odd_even_insert_constraint` implemented in this class, it is possible to add constraints to a constrained triangulation in a way that only the parts of the input constraints that overlap an odd number of times are kept, whereas the ones that overlap an even number of times are erased.

In addition, a simple class to store the label and repair states (part of the repair algorithm) has been written. These are stored in each face and could easily be extended or modified to support other repair methods in the future.

### 2.3 Polygon repair method using the odd-even heuristic

The main part of the repair method is in the `Polygon_repair` class. Mainly, this involves:
* adding the edges of the input polygons to the triangulation (using the odd-even constraint adder in the new triangulation class), including some pre-processing to remove identical edges that are present an even number of times in the input, which serves to optimise the code;
* labelling each triangle by applying the odd-even rule every time that a constrained edge is passed, resulting in the polygon exterior labelled as -1, polygon interiors with positive labels, and holes with negative labels;
* reconstructing multipolygons from sets of adjacent triangles by creating rings for outer boundaries and holes, assembling them into polygons with holes, then into a single multipolygon.

### 2.4 Tests

The method was tested in four different ways:
1. using the WKT reader to compare the results of repairing unit tests that represent a range of problematic configurations, which are tested against WKT files that are known to be valid;
2. robustness tests using a large library of clipart files, which were processed correctly without any crashes;
3. performance tests using large invalid polygons identified from the previous development of prepair;
4. building a small validator for this project, which is not comprehensive but at least tests the simplicity of boundaries, correct nesting of boundaries/holes in each polygon, connectivity of interior of polygons, and no overlaps or connectivity between polygons of a multipolygon. This validator is used to test the unit test files, the clipart files and the large invalid polygons.

### 2.5 Documentation and examples

Together with the implementation, documentation for the method was provided in the form of the User and Reference manuals for the [Polygon repair package](https://kenohori.github.io/cgal-polygon-repair-docs/Polygon_repair/index.html). The parts related to the multipolygon support are instead in the [Polygon package](https://kenohori.github.io/cgal-polygon-repair-docs/Polygon/index.html). The manuals include all classes and functions meant to be used by general CGAL users, as well as a few simple examples:
* Constructing, traversing and drawing multipolygons (`multipolygon.cpp` and `draw_multipolygon_with_holes.cpp`)
* Using the simple public API to repair (multi)polygons (`repair_polygon_2.cpp`).

And some more complex pieces of code (in the `test` folder of `Polygon_repair`), which show how to use more advanced functionality that is currently not documented in the manuals:
* Drawing unit test polygons (`draw_test_polygons.cpp`),
* Repairing using exact kernels (`exact_test.cpp`),
* Running unit tests (`repair_polygon_2_test.cpp`),
* Validating all WKT files in a folder (`validate_wkt.cpp`),
* Repairing a file step by step and exporting the triangulation (`write_labeled_triangulation.cpp`),
* Exporting repaired polygons as GeoJSON (`write_repaired_polygons.cpp`).

## 3. Prospective development

The basic goals of the project have been met, but there are several directions in which future development could be taken.

Concerning the method, although the code is prepared to support multiple repair heuristics neatly, only one repair heuristic (odd-even) has been implemented so far. Implementing others, such as a set difference and winding number, would make the method applicable to more situations and could re-use most of the code built for the odd-even repair method.

The labelling and the reconstruction approaches implemented in this project differ from those in prepair and in theory should be faster since they don't rely on slower O(log n) data structures in key steps. However, within the timeframe of the GSoC, the code has not been optimised to the same extent as in prepair and comparisons have not been made.

Finally, it is worth noting that there are functions in the Polygon package that currently only work on polygons, not in polygons with holes or multipolygons. Extending these functions to work on polygons with holes and multipolygons would make this package more consistent and useful.

## 4. Challenges and accomplishments

The most challenging part of the GSoC was the implementation of the new labelling and reconstruction approaches, since these were devised for the project and not just re-implementations of the logic available in the paper or in prepair.