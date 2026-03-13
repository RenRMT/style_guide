---
title: Map layout specification
parent: Maps
nav_order: 1
---
3/30/3
3 seconds: locator maps, (single) event highlight
30: seconds: dat

# Map elements

Very important here: determine which are required, which are optional + provide instructions on when to include or omit the optional elements.
Additionally, determine which elements must/should/could have a fixed position and which can be placed best on best judgement + again provide instructions on what basis to make those judgement calls. Add instructions how to avoid redundancy of information. Which (optional) elements can be either/or. 

m = mandatory, a = allowed, d = disallowed

| Element |	Locator map (3)| Narrative visual (30)|	Data exploration(300) |
|:--------|:--------------:|:--------------------:|:---------------------:|
| Main map | m	| m	| m |
| smaller-scale location map |d	|a	|a|
| larger-scale inset maps showing detail	|d	|a	|a|
| insets of locations outside the area of the main map	|d	|a	|a|
| title / subtitles	|a	|m	|m|
| legends	|d	|m	|m|
| scale indicators	|d	|a	|a|
| orientation indicators	|d	|a	|a|
| graticule (lines of latitude and longitude)			
| extent indicators			
| explanatory notes			
| source note / publisher credit and copyright notice	|d	|a	|m|
| author credit| d	|a	|m|
| neatline	|d	|a	|a|
| additional graphics (photos, graphs	|d	|d	|a|
