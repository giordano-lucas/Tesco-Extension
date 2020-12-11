

## Naive Analysis
In this naive Analysis will not take into account the geographical aspect of the dataset. Since we only want to get a feeling about how k-means performs with the ```tesco``` data, we will select a subset of the dataset for this analysis. To be as precise as possible we study the smallest aggregation level for the entire year period.

### Standard K-means analysis : silhouette and sse
The followings cells aim at choosing which value of ```k``` leads to a good clustering assignment. The standard procedure for doing so is to look at the ```silhouette``` and ```sse``` plots.

{% include_relative images/naive-metrics.png %}

The elbow method applied on the SSE graph seems to indicate that 2 and 3  might be relevant choices of ```k``` for this dataset.
Moreover, the silhouette graph also suggests that these values are fair tradeoffs between the goodness of the fit and the number of clusters. By curiosity, we will also keep k=4 as one of the best candidates.

To analyze deeper the values of the silhouettes scores obtained for our candidates we plot the silhouettes score obtained by each data points of each cluster.

{% include_relative images/naive-silhouette.png %}

Silhouette scores can be interpreted as follows :
 * 1  indicate that the sample is far away from the neighboring clusters
 * 0  indicates that the sample is on or very close to the decision boundary between two neighboring clusters
 * <0 indicate that those samples might have been assigned to the wrong cluster. 
 
Only for ```k = 4```, we observe higher variability scores (inside clusters) what indicates that the clustering might not be so good. However, for all k, we do not have the presence of clusters with below-average (red dotted line) silhouette scores which is a good point.

### K-means 2D visualisation

#### Dimensionality Reduction : PCA and TNSE

In the previous point, we tried to formally understand the goodness of fit for the clusters found. Here, we will take a more visual approach. To do so we naively project the data on a 2D space using PCA and T-SNE algorithms. We plot the results and label them with the labels produced by the clustering (choose k=3 among the candidates we listed earlier), hoping that the obtained figure will look like the map of London. We obtained the following figure:

{% include_relative images/naive-k-means-plot.png %}

Here we understand that k = 2 might be a better choice if we think of TSNE. 

If we go back to our initial question, we tried to understand if there was a relationship between similar typical products and similar areas (geographical areas). At the beginning of this notebook, we thought that we might be able to recognize London from those simple dimensionality reduction 2D plots but this is clearly not the case. This does not mean that the previously mentioned relationship does not exist, it simply we have to think more carefully about how we are going to address the question. Thus we need to proceed with a deeper analysis of the clusterings in the geographic space.



