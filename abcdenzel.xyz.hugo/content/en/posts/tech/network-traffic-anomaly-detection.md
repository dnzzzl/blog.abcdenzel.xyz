+++
date = '2025-11-26T11:26:35-04:00'
draft = true
title = 'Network Traffic Anomaly Detection'
tags = ['AI', 'python', 'algorithms & data structures']
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

{{< details summary="See the details" >}}

Binary trees are [data structures]() that organize data in a traversable, hierarchical way.
Trees are composed of levels, also called depth, at each level we find nodes that branch off into two other nodes a level below it, in a way that the top node is the root and it branches off into two other nodes below.

The relationship between these nodes is the logic behind the data structure, it represents a requirement that to reach a node below, a path using the correct branches needs to be followed.

In the context of Isolation Forest, this becomes insightful: the path length itself tells us how many decisions were required to isolate a point.

To construct our binary trees we will split the data by choosing a random column or feature of the data, and a random value within the range of that column. This will give us a `_left` and `_right` side that we will go and recursively keep splitting, until we reach a preconfigured depth or a leaf node is reached.

{{< /details >}}

You know how binary trees create binary paths, let's see how Isolation Forests uses forests of these trees to isolate data points and detect anomalies, here is the theory:

### Isolation forest algorithm

#### Executive summary

In binary search, an algorithm needs to evaluate the current node and the nodes below it, make a binary decision that gets it closer to its destination and, upon failure, be able to backtrack and choose a different path at a diverging point.

However, binary search is not our problem to solve today, instead, we will quantify the average path length across an ensamble of `n` amount of trees based on our data; and then its a simple matter of seeing which data points have shorter-than-average or extremely short paths.

Of course, I could not have been that simple, there are nuances worth taking into account such as the problem of comparing path lengths across trees of different depths, the gist is, its not apples to apples.
_This is covered below if you are as interested as I was, and in the original IForest paper._

Having taken care of that, and applying the anomaly score formula you can then have a neat 0-1 score of how anomalous each connection your computer has made is.

{{< details summary="Backstory of the algorithm" >}}
The Isolation Forest approach to Anomaly Detection differs from previous methods that relied on profiling the normal activity via statistical analysis to then identify outliers, those methods produced higher rate of false positives, had non linear [time complexity]() limiting their application to smaller datasets and in simple terms they were just worse. See the references for sources.

With the iForest apporach we rely on two characteristics: anomalies are few and different. which allows us to separate anomalous data points from others using less number of partitions.

Partitions in this context can be explained by imagining a 2 dimensional scatter plot of our data points, where anomalies are few and far between, take for example a point to the far right, because of its position you could isolate it by drawing a line to the left of the anomalous point and it took 1 partition to isolate that point. Whereas to isolate a point that is surrounded by neighbors, you would have to draw four lines: above, below and one on each side.

With higher dimensional data space the principle still stands, and each dimension represents one attribute of our data, precisely, each data point collected within each conn.log record.

Lets go back to our binary tree data representation and consider this visually:

```plaintext
        [root]
        /    \
      [A]    [B]
      / \    / \
    [C] [D][E] [F]
```

To reach node D, you must: go left at root, then go right at A. This path (left → right) encodes a unique "address" for that data region. Anomalies, being isolated, get short addresses. Normal points, being clustered, require longer addresses to distinguish them from neighbors.

{{< /details >}}

{{< details summary="Mathematical Intuition" >}}

#### Euler's Constant, Harmonic Series and the Comparability Problem

This section is short and sweet, here is the math laid out simply:

"_When you calculate path lengths in isolation trees, you have a comparability problem. If I build a tree with 100 samples and you build one with 10,000 samples, your paths will naturally be longer—not because your anomalies are less anomalous, but simply because there's more data to partition through_"

The **expected path length** for a point in a binary search tree of n samples follows:

```LaTeX
c(n) = 2H(n-1) - (2(n-1)/n)
```

Where H(n) is the harmonic number (approximately ln(n) + 0.5772).
This gives us a baseline: if a point's average path length across many trees is much shorter than c(n), it's anomalous.

The anomaly score formula:

```LaTeX
s(x, n) = 2^(-E(h(x))/c(n))
```

Where:

- E(h(x)) = average path length for point x across all trees
- c(n) = normalization factor

Score close to 1 → anomaly
Score close to 0.5 → normal
Score close to 0 → very deep in normal cluster
{{</details>}}

### Conclusions

Anomalies are few, so they don't cluster.
Anomalies are different, so they lie in sparse regions.
Therefore: Random partitions isolate anomalies quickly while normal points require many partitions.
Path length becomes the anomaly signal.

## Implementation

### Connecting concepts to our Zeek conn.log

Zeek's default configuration produces several log types. The most relevant for anomaly detection typically include:

- conn.log - Connection records (source/dest IPs, ports, duration, bytes transferred)
- dns.log - DNS queries and responses
- http.log - HTTP request/response metadata
- ssl.log - SSL/TLS handshake information

For this project, we'll focus on conn.log as it provides information about each and every connection reaching our Network Interface, and a bulk of this data is numerical, making for useful features that we can use for Isolation Forest.

Although dns.log is a dataset I would also like to explore since it could raise alerts for anomalous DNS resolutions useful for threat detection and hunting.

Each connection record has multiple features forming a high-dimensional point:

| **Feature**      | **Role in Isolation**                                       |
|------------------|------------------------------------------------------------|
| **Duration**     | Long-lived connections may isolate quickly.                |
| **Orig Bytes**   | Unusual transfer sizes stand out.                          |
| **Resp Bytes**   | Asymmetric transfers may be anomalous.                     |
| **Orig Pkts**    | Anomalies in the number of packets exchanged can hint at unusual traffic patterns or potential attacks. Monitoring packet counts helps differentiate normal from suspicious behavior.            |
| **Conn State**   | Unusual states (S0, REJ) may indicate issues.             |

When we build an isolation tree, a random split on orig_bytes might immediately separate a data exfiltration attempt (gigabytes transferred) from normal browsing (kilobytes). That single split isolates the anomaly—hence a short path length, hence a high anomaly score.

### Python script

The theory was fun to read, but now I have to actually write the code.
Before we write any code, let's establish what we need and justify each dependency.

#### Libraries

- Pandas: Which helps with loading data, preprocessing and column manipulation. Zeek logs are structured tabular data. Pandas provides intuitive DataFrame operations for loading JSON/CSV input files, handling missing values, and adding our anomaly score column.

- Numpy: To handle numerical operations and array manipulation. Isolation Forest relies heavily on random sampling and tree traversal calculations. NumPy provides vectorized operations that are orders of magnitude faster than pure Python loops.

- SciKitLearn: Includes algorithm implementations already coded, it is the go-to alternative for production code instead of implementing your own for blogpost content.

#### Strategy

First thing that comes to mind when I problem solve is Domain Modeling. The first thing I try to figure out is an account of all the moving pieces in our solution, their properties and actions.

I landed in having an `IsolationForest` class that can recursively create other `IsolationTree`s by partitioning the data, the root node and any internal node is itself a tree for other nodes.
And an orchestrator `IsolationForest` that manages the overall process and handles the initial data sampling and other properties such as the total number of trees to create.

- `IsolationTree`:
  All the properties of the tree:

    partitioning
  - The `feature` selected for partitioning
  - The `value` selected for partitioning
  - `right` pointer to the right `IsolationTree` child node.
  - `left` pointer to the left `IsolationTree` child node.
    structure
  - The `max_height` to limit the depth of the tree
  - The  `current_depth` this tree is in.

  The methods or actions that the tree can perform:
  - `fit(X: np.array, nt: int = 1)`:
    Given a sample of shape (n_samples, n_features), build the tree recursively.
  - `calculate_path_length(x: np.array)`:
    Return the depth of a sample by traversing the trees.

- `IsolationForest`:
  The methods or actions that the tree can perform:
  - `fit(X: np.array, nt: int = 1)`:
    Given a sample of shape  (n_samples, n_features), build the tree recursively. Store in self.trees.
  - `predict()`:
    Using depth information within each tree, use the anomaly formula to calculate the score for each sample in X.
  - `_calculate_avg`:
    Compute the average value of a given feature across all trees, used for comparison within the anomaly calculation.

### Zeek Integration

#### Zeek Installation

#### Automating alerts

1 - elastic + kibana + elastic agent: Full SIEM solution with the ability to set up alerts, higher complexity

2 - set up a script to run the model and send alerts on anomalies, low complexity, manual data exploration skills required to investigate events.

## Opportunities for improvement

1. **Feature Engineering**: Explore additional features that could enhance the model's ability to detect anomalies. This could include time-based features, such as the duration of connections or the frequency of requests from a particular IP address. Encoding categorical features seems like an easy step that can prove to be immediately useful (One Hot Encoding and other methods).

2. **Tuning Parameters**: Experiment with different parameters for the Isolation Forest algorithm itself, such as the number of trees, the maximum depth of the trees, and the contamination parameter. This could help improve the model's performance on specific datasets.

3. **Testing and Model Evaluation**: Admittedly, I am not the best at Test Driven Development. Implementing a testing and evaluation framework to assess the model's performance would allow us to track the precision of the anomaly detection. This could include mainly cross-validation with a control datasets where anomalies are known,to better understand the trade-offs between false positives and false negatives.

44. **Integration with Other Tools**: This part involves crafting a light microservices system where we enrich the data with the anomaly calculation before being shipped to a monitoring solution such as Kibana + Elastic which is what I personally use. This would integrate the Isolation Forest model with other security tools and frameworks such as a SIEM or IDS to raise alert or take action. This would also accomplish **Real-time Monitoring**.

## References

[Isolation Forest Paper by Tony Liu et al](https://www.lamda.nju.edu.cn/publication/icdm08b.pdf)
