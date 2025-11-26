+++
date = '2025-11-26T11:26:35-04:00'
draft = true
title = 'Network Traffic Anomaly Detection'
tags = ['AI', 'python']
categories = ['tech', 'writeup']
+++
Setting up a lab for Network Traffic Analysis is one of those projects that has been floating around my list of things to do. It feels like such a useful and genuinely insightful look into the connections that a computer interacts with throughout every day.

Being able to log, analyze, raise alerts and subsequently automate analysis and even take informed proactive action on a system based on the data; these are skills that feel infinitely valuable for any system exposed to the Internet or a Local Area Network.

So I had to finally tackle this challenge.

I set out to read, understand and learn as much as needed about anomaly detection, to implement a simple binary-tree-based algorithm under a sensible time constraint of two days.

This allows me to dip my toes into a wider goal of mine which is to bridge AI and cybersecurity using a practical project based approach.

In this blog entry I walk you through my process from scratch to implementation of an Isolation Tree approach to Anomaly Detection.

## Prerequisites

These are subjects of foundational knowledge that needs to be solidified before we tempt our luck with an implementation.
And of course, before we have a forest, we need a tree:

### Binary trees

Binary trees are [data structures]() that organize data in a traversable, hierarchical way.
Trees are composed of levels, also called depth, at each level we find nodes that branch off into two other nodes a level below it, in a way that the top node is the root and it branches off into two other nodes below.
The relationship between these nodes is the logic behind the data structure, it represents a requirement that to reach a node below, a path using the correct branches needs to be followed.
To traverse the binary tree an algorithm needs to evaluate the current node and the nodes below it, make a binary decision that gets it closer to its destination and, upon failure, be able to backtrack and choose a different path at a diverging point.
However, binary path traversal is not our problem to solve(verification needed: what is the use for binary trees in isolation forest), the way we will use binary trees is...

## Learning

### Isolation forest algorithm

The Isolation Forest approach to Anomaly Detection differs from previous methods that relied on profiling the normal activity via statistical analysis to then identify outliers, those methods produced higher rate of false positives, had non linear [time complexity]() limiting their application to smaller datasets and in simple terms they were just worse. See the references for sources.
With the iForest apporach we rely on two characteristics: anomalies are few and different. which allows us to separate anomalous data points from others using less number of partitions.

Partitions in this context can be explained by imagining a 2 dimensional box plot of our data points, where anomalies are few and far between, take for example a point to the far right because of this you could isolate one by drawing a line to the left of the anomalous point and it took 1 partition to isolate that point. Whereas to isolate a point that is surrounded by neighbors, you would have to draw four lines: above, below and one on each side.

With higher dimensional data space the principle still stands, and each dimension represents one attribute of our data, precisely, each data point within each conn.log record.

## Implementation

### Python script

#### Defining Requirements

Before we write any code, let's establish what we need and justify each dependency.

##### Data Understanding

Zeek's default configuration produces several log types. The most relevant for anomaly detection typically include:

- conn.log - Connection records (source/dest IPs, ports, duration, bytes transferred)
- dns.log - DNS queries and responses
- http.log - HTTP request/response metadata
- ssl.log - SSL/TLS handshake information

For this project, we'll focus on conn.log as it provides information about each and every connection reaching our Network Interface, and a bulk of this data is numerical, making for useful features that we can use for Isolation Forest.

Although dns.log is a dataset I would also like to explore since it could raise alerts for anomalous DNS resolutions useful for threat detection and hunting.

##### Libraries

- Pandas: Which helps with loading data, preprocessing and column manipulation. Zeek logs are structured tabular data. Pandas provides intuitive DataFrame operations for loading JSON/CSV input files, handling missing values, and adding our anomaly score column.

- Numpy: To handle numerical operations and array manipulation. Isolation Forest relies heavily on random sampling and tree traversal calculations. NumPy provides vectorized operations that are orders of magnitude faster than pure Python loops.

### Zeek Integration

### ElasticStack visualization and alerts

## References

[Isolation Forest Paper by Tony Liu et al](https://www.lamda.nju.edu.cn/publication/icdm08b.pdf)

review network-traffic-anomaly-detection.md and comment on the structure, direction of the walkthrough and narrative voice of the text, this is my personal style its very easy to read and guides through a story of the motivation behind the project, you may make suggestions in these aspects too.
Expand the technical details and correct what I get wrong, guide me to study or review any subjects that are relevant to successfully finish this project and with it its blogpost.
