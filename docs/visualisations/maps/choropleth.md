---
title: Choropleth maps
parent: Maps
nav_order: 3
---
show differences in relative quantities with differences in luminosity/hue (+ hex bin)
NB: relative quantities only, using for absolute quantities is misleading
colour choice:
 - monochromatic: for sequential values
- diverging: 2 or more colours, implies average + deviation
Category breaks: There are three common choices for category breaks: natural, equal interval and quantile. Quantile breaks should be the default choice, unless you have a very good reason to do otherwise. Choosing a different break type can help if you wish to emphasise a specific pattern found in your data but make sure not to move from emphasis to manipulation. 
•	pattern type: continuous/sequential: blurs boundaries, use for hotspot effects; discrete/qualitative: emphasises boundaries, use for classification
