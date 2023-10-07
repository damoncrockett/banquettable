# Clustering with DBSCAN and the Banquet Table

Material analyses of collections often aim to **identify physically similar groups of items, or clusters**. When each item in a collection has a suitable numerical representation—in our case, a vector indicating its position in material space—this representation can be used to discern clusters.

Many traditional clustering algorithms, such as k-means, hierarchical, and spectral, are *exhaustive*. That means every item in the data is categorized into a cluster, and we decide the number of clusters in advance. This method works for some data distributions, but can misplace outliers in data that's evenly distributed.

In our scenario, the data distributes evenly throughout the expressive space. There are only a few evident clusters (as shown in Figure 1). Yet, small clusters of highly similar papers exist, and our objective is to uncover them.

> ![Figure 1](fig/fig1.png)
>
> **FIGURE 1.** A 3D scatterplot showing our print collection data's distribution in the expressive space. At first glance, grouping these data isn't straightforward. The point colors indicate an exhaustive algorithm's cluster assignments, illustrating the imperfect results.

For our needs, a **non-exhaustive clustering algorithm** is apt—one that allows outliers to remain unclustered. We also prefer to let the data distribution define the number of clusters. A current algorithm that fits this description is DBSCAN (Density-Based Spatial Clustering of Applications with Noise). However, the conventional application of DBSCAN might not be optimal for our low-dimensional spaces. It might yield clusters that stretch too far, causing items at the extremes to be too dissimilar (as visualized in Figure 2).

> ![Figure 2](fig/fig2.png)
>
> **FIGURE 2.** The same data points from Figure 4, but clustered using standard DBSCAN. The results are more satisfactory, but the extensive central cluster expands too much, leading to dissimilarity between its boundary points.

To resolve this, we've introduced a modification to DBSCAN: **the banquet table constraint**. This imagines collection items as attendees at banquet tables (clusters). The concept introduces "guests" for collection items, which are other papers from our reference collection within a predefined distance in the expressive space. For two collection items to belong to the same cluster, they must share at least one guest (explained further in Figure 3).

> ![Figure 3](fig/fig3.png)
>
> **FIGURE 3.** The banquet table constraint visualized. Banquet attendees (shown as yellow smileys) invite guests (small colored dots). Any two attendees seated together must share a guest. Shared guests are traced between attendees in this illustration.

Upon determining the value of `d`, the distance within which guests must be to count, we can establish which clusterings are permissible. The DBSCAN distance parameter, `epsilon`, can then help us determine the difference between allowable and non-allowable clusters. Our aim is to choose the highest value of `epsilon` that still results in a permissible clustering.

Three main categories of unclustered attendees emerge:

1. **Lonelies:** They can't have guests as no papers from the reference collection are within the distance `d`. Their uniqueness makes them noteworthy.
2. **Outsiders:** They have guests but share none with other attendees. They are unique within their collections.
3. **Noise:** These attendees might find a suitable cluster at certain `epsilon` values. They are harder to categorize and warrant close examination.
