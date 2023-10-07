# Usage

```python
import os,sys
sys.path.append(os.path.expanduser("~") + "/banquettable")
from banquet import banquet_tabling, plot_metrics, get_guest_lists

metrics_list = []
for i,d in enumerate(np.linspace(0.05,0.1,100)):
    _, metrics = banquet_tabling(collection_frame, 
                              reference_frame, 
                              featcols, 
                              d, 
                              eps=0.164, eps_increment=0.0001)
    metrics_list.append(metrics)

metrics_plot = plot_metrics(metrics_list,inflection_points=True,diff_threshold=0.025)

d = 0.0677 # example value for `d`
tabling, metrics = banquet_tabling(collection_frame, 
                              reference_frame, 
                              featcols, 
                              d, 
                              eps=0.164, eps_increment=0.0001)

collection_frame['cluster'] = tabling
collection_frame['guest_list'] = get_guest_lists(collection_frame, reference_frame, featcols, d)
```

# Clustering with DBSCAN and the Banquet Table

In case your analyses require a high degree of intracluster similarity, exhaustive algorithms like k-means can be inadequate, because they force outlier points into clusters where they don't strictly belong. DBSCAN is an excellent non-exhaustive alternative, but there is a problem with DBSCAN used “out-of-the-box”: it is ordinarily applied in cases where the native dimensionality of the data is very high and there is an abundance of empty space. In such contexts, clusters can be of any shape and remain clearly set apart from one another. But in lower-dimensional spaces, any cluster shapes that deviate significantly from “round” will contain points that are too far apart --- farther, in many cases, than points in other nearby clusters.

Accordingly, we modify DBSCAN by adding a constraint we call the _banquet table_. The constraint, as initially conceived, requires that, in addition to your primary collection --- the one you are clustering --- you also have a _reference collection_ that resides in the same feature space. The banquet table constraint conceives of collection items as banquet attendees that sit together at banquet tables (i.e., clusters). Attendees can invite _guests_; a guest for attendee `a` is any reference item within distance `d` of `a` in the feature space. In order for two collection items to sit at the same table, they must share at least one guest.

The distance `d` is either chosen ahead of time (e.g., ours is determined by an estimate of measurement error) or by finding inflection points across a range of metrics like Jaccard similarity, etc. Because the banquet constraint applies to every pair of collection items in a cluster, including those sitting “opposite” each other, the constraint enforces tight, round clusters and thus ensures high intracluster similarity.

Once we’ve chosen `d`, we have a well-defined concept of guest and can determine which tablings (i.e., clusterings) are allowable. But how do we choose a tabling? Luckily, DBSCAN uses its own distance parameter, `epsilon`, that, as it happens, can be used to define a threshold between permissible and impermissible tablings. We choose the maximum value of `epsilon` that still yields an allowable tabling. This will maximize the number of tabled attendees.

There are three categories of untabled attendees of interest to us. The first are lonelies. Lonelies cannot have guests, because there are no items in our reference collection within distance `d`. These are interesting because—assuming our reference collection is a suitably comprehensive—they are likely to be highly unique, both within their collection and elsewhere. The second group are outsiders, who have guests but share no guests with any other attendees. Outsiders are interesting because they are unique within their collections. 

The tabling of lonelies and outsiders is strictly impossible once we’ve chosen `d`, and thus we might say they are not in attendance at all. But members of the third group, noise, will be tableable for certain settings of epsilon, and can thus be viewed as attendees who show up to the banquet but never find a suitable table. These cases are the toughest to interpret and may sit on the borderline in some of our analyses—for example, it may make sense to consider, for a given noise item, which clusters are nearby.

# The Banquet Table without a Reference Collection

There is no special reason why a "roundness" criterion like the banquet table need involve a reference collection. Without it, the metaphor makes less sense --- e.g., there are no "guests" --- but because the reference collection figures in the algorithm primarily as a set of spatial anchors, we can enforce the constraint without the collection itself. We require, of any two points in a cluster, that their orbits of radius `d` intersect. In some cases, this ends up being a slightly weaker criterion than the original, because they original demands specific points of overlap, but the results will be similar.
