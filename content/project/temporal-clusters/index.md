---
date: "2021-02-05T00:00:00Z"
image:
  caption: Screenshot of Application
  focal_point: Smart
summary: Shiny application buily to visualise temporal clusters in the State Papers network
title: Temporal Clusters Browswer
tags:
- Network Analysis
---

This application was built as a UI layer on top of the results of an algorithm developed by Sebastian Ahnert to detect the presence of short-term temporal clusters in network data.

The application displays all of the detected clusters on an interactive timeline, built using R and Plotly. Clicking on a cluster in the timeline displays an interactive network diagram in a separate pane.

The application also lists the letters belonging to the cluster in order, alongside links to the relevant manuscript images in State Papers Online (if these are available internally). It's also possible to filter the timeline by keyword or individual.


![](Screen%20Shot%202021-04-26%20at%2011.15.06.png)

![](Screen%20Shot%202021-04-26%20at%2011.15.26.png)
