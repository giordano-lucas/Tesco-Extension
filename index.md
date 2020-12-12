# Introduction

We try to give an answer to the following questions:

1. Are similar areas in terms of typical products geographically close?
2. Can we naturally cluster areas geographically when it comes to food consumption in London?
3. How do those clusters differ when we vary the aggregation level?
4. In the context of an advertising campaign, which areas of the city should be associated with which kind of products?

Before answering those questions we will first visualise our dataset to understand if we could even hope to be able to demonstrate such a relationishp between product and geography.

# (a) Interactive visualization
### Data loading
### Statistical GIS Boundary Files dataset

In the section, we will use ```geopandas``` to load the [Statistical GIS Boundary Files for London](https://data.london.gov.uk/dataset/statistical-gis-boundary-files-london). Even if this dataset was made avialable in 2011, we should note that the boundaries haven't really changed compared to 2015 (date of Tesco Dataset) and it should not impact our visualisation.

For each aggregation level ```LSOA```, ```MSOA```, ```WARD```, ```BOROUGH``` we read the correponding file and retreive the following subset of columns: 

* ```area_id ```: the id of the area considered
* ```name    ```: the name of the area considered
* ```geometry```: the geometric shape of the area in the 2D london map (polygon or multipolygon if the area is definied in multiple pieces)

We then store the resulting geopandas dataframe into a dictionary for later usage.

### Tesco data set

We can proceed further by reading the tesco dataset. This dataset contains a subset of the orginal tesco dataset containing some of typical product features:

> ```fat,saturate,sugar,protein,carb,fibre,energy_tot,h_nutriments_calories```

Moreover, it stores these features for each combinaison of 

 * ```aggretation level``` : lsoa, msoa, ward, borough 
 * ```period```            : January, February,..., December as well as the yearly aggregation (Year)

 ### Creation of the actual figure and components
Here is are the actual components defined. We created a ```bkapp``` (bokeh application) function that given the ```doc``` (bokeh document), creates the components :

1. ```plot          ```: map of colors for the selected areas
2. ```btn_period    ```: button to select the period
3. ```btn_agg_level ```: button to select the aggregaton level
4. ```select_feature```: button to select the feature of typical product to display (or clustering)

it then adds the respective event handers and links the components to the ```doc```

{% include_relative images/vizu.html %}

After playing a bit with the visualization, we can easily convince ourselves that we can indeed recognize some dependencies between typical product features and the geographical location of the areas. 
To capture the general trends (not by individual products) we will try to cluster the areas according to their typical product properties

# (b) Clustering Validation

We will start by studying the output clustering of a K-means algorithm to undertand which values of $k$ will produce the most natural clustering __*in the typical product space*__. We will then try to make some sense of out this clustering by performing a vizualization in 2D using several dimentionality reduction techniques. We might hope that we will already be able to see some geographical structure in those plots (recognize somehow the map of London). If not we will have to apply other techniques to help access the geographical validation of the clusters, if any.

## Naive Analysis
In this naive Analysis will not take into account the geographical aspect of the dataset. Since we only want to get a feeling about how k-means performs with the ```tesco``` data, we will select a subset of the dataset for this analysis. To be as precise as possible we study the smallest aggregation level for the entire year period.

### Standard K-means analysis : silhouette and sse
The followings cells aim at choosing which value of ```k``` leads to a good clustering assignment. The standard procedure for doing so is to look at the ```silhouette``` and ```sse``` plots.  

![Alt text](images/naive-metrics.png){:class="img-responsive"}

The elbow method applied on the SSE graph seems to indicate that 2 and 3  might be relevant choices of ```k``` for this dataset.
Moreover, the silhouette graph also suggests that these values are fair tradeoffs between the goodness of the fit and the number of clusters. By curiosity, we will also keep k=4 as one of the best candidates.

To analyze deeper the values of the silhouettes scores obtained for our candidates we plot the silhouettes score obtained by each data points of each cluster.

![Alt text](/images/naive-silhouette.png){:class="img-responsive"}

Silhouette scores can be interpreted as follows :
 * 1  indicate that the sample is far away from the neighboring clusters
 * 0  indicates that the sample is on or very close to the decision boundary between two neighboring clusters
 * <0 indicate that those samples might have been assigned to the wrong cluster. 

Only for ```k = 4```, we observe higher variability scores (inside clusters) what indicates that the clustering might not be so good. However, for all k, we do not have the presence of clusters with below-average (red dotted line) silhouette scores which is a good point.

### K-means 2D visualisation

#### Dimensionality Reduction : PCA and TNSE

In the previous point, we tried to formally understand the goodness of fit for the clusters found. Here, we will take a more visual approach. To do so we naively project the data on a 2D space using PCA and T-SNE algorithms. We plot the results and label them with the labels produced by the clustering (choose k=3 among the candidates we listed earlier), hoping that the obtained figure will look like the map of London. We obtained the following figure:

![Alt text](/images/naive-k-means-plot.png){:class="img-responsive"}

Here we understand that k = 2 might be a better choice if we think of TSNE. 

If we go back to our initial question, we tried to understand if there was a relationship between similar typical products and similar areas (geographical areas). At the beginning of this notebook, we thought that we might be able to recognize London from those simple dimensionality reduction 2D plots but this is clearly not the case. This does not mean that the previously mentioned relationship does not exist, it simply we have to think more carefully about how we are going to address the question. Thus we need to proceed with a deeper analysis of the clusterings in the geographic space.
## Geographical Clustering visualization 

In the previous cells, we failed to show the presence of a geographical relationship between products and area. To have a better idea of how the clusters look like in the geographical space we create an interactive visualization of these clusters on the real map of London. Using geopandas we represent each area of the dataset on a proper map and we label them using the possible clusterings. 
### Creation of DataFrame of clusters
As a first step, we create a new dataframe ```tesco_year``` that contains only the data for the complete year. In addition, we will add the clustering assignments for various values of k. We choose k to be in the range [2,8]. We do so because we don't want to restrict ourselves to the previously computed candidates and miss a good geographical clustering.

### Interactive Clustering visualisation
Now that all the needed clustering are computed and saved we can set up the visualization. We obtain the following result :

{% include_relative images/cluster-vizu.html %}

Surprisingly, for many values of ```k```, we can actually make some geographical sense out of the clustering! Let's analyse further some of the plots:

* For ```k = 2```, across all aggreation levels, we can generaly observe that the center and top east part of London tend to be clustered together. If we take a look at the ```lsoa``` plot (the most precise aggregation level), we clearly see two __concentric__ circle-shaped clusters. A further analysis could try to relate these clusters to actual classes of people (for instance densily populated / wealthy areas, who knows?).

* For largers ```k```, this concentric cluster shapes tendency keeps appearing, which is really interesting given the fact that the K-Means algorithm didn't take any geographical metric into account.
* We can almost always distinguish a central cluster.

In the end, we see that the clustering looks quite decent and conforts us in the idea that we are able to give geographical meaning to them. Overall, these results are exciting but it would be interesting to formally quantify them.

## Formal Geographical Validation
In this section, we want to have a piece of more formal evidence that some of our clusterings are good (what we saw on the visualization). Thus we have to find metrics that could reflect the goodness of the clustering in the geographic space. 

### Geographical Silhouette
As a first step we will try to use the silhouette score again but in the geographical domain. To do so we will compute the silhouette score as we did at the beginning of the study but this time we will use the geographical distance as the distance between data points. In order to be able to compare the future results, we will also compute the silhouette score produced by a random clustering. We do so hoping that our clustering will perform significantly better than the random one. 

![Alt text](/images/validation-geographical-distance-silhouette.png){:class="img-responsive"}

We see that unfortunately, this does not go in the direction of a good evidence of fit. The scores are quite small compared to the ones we got in the naive analysis. They even goes to negative values, meaning that, on average, points tend to be closer to other clusters than to their centroid, or at least the geographical sense. 

However, we need to take a step back and try to fully undertand why such result have arisen. Indeed, we should remind ourselves that the silhouette score computation relies on the average distance to other points in the same cluster, to which we substract the smallest mean distance to other points in other clusters. This metric is thus likely to produce high scores for dense clusters distributed as a gaussian centered in the centroid. This is is not the shape that we observed in our visualization : we rather saw concentric circles, scattered around the map (they do not form unique blocks)! Indeed, if we were to run a K-means algorithm in the 2D geographical vector space, we would never get such concentric shaped clusters. 

If the sihouette metric is good when we want to evaluate clusters as described in the previous paragraphs, we understood that we should probably try another approach to evaluate our clusters.

### Graph analysis

In this section, we will use a slightly different approach than the previous one. We again use the silhouette score but we will compute it using the following distance metric: the distance between two areas will be defined as the shortest path (smallest number of areas to cross) to go from one area to another. We do so because the centroids of similar areas might be far from each other even if the areas are located next to each other. We hope that by giving a slightly more geographical meaning to the distance measure, we will be able to better evaluate the clusters.

####  Creation of the graph

This task would clearly be better solved using a graph approach. We will therefore define a graph G = (V,E) where:

* V : the set of vertices is the set of areas in London.
* E : the set of edges is such that we connect to areas if they have a common border.

Then using this graph, we will compute all the shortest paths possible and use them as the distance metric for the silhouette score. In python, the ```networkx``` package is naturally suited for this task.

Note that we perform this analysis only on the lsoa aggregation level. 

#### Creation of the distance matrix
We now create a distance matrix such that it's $(i,j)$ entry contains the length of the shortest path between area $i$ and area $j$. This matrix is thus symmetric.

#### Silhouette scores computation
We again compute silhouette score as above for different values of k, and compare it to a random clustering.

![Alt text](/images/validation-graph-silhouette.png){:class="img-responsive"}

The obtained results are very similar to the previous analysis. We might have put to much hope on the changes made by the distance metric. This might be due to the fact that the silhouette score is not well suited for our application. Thus we need to define a more appropriate metric to evaluate the graph. Note that despite the fact that the silhouette score is bad we are still convinced that the clustering is meaningful in the geographic space (because of the visualization).

### Border Scores
In this section, we create a new metric to evaluate if geographically close areas tend to belong to the same cluster of type. We proceed as following: for each area of the dataset we look at all its direct geographic neighbors and compute the number of neighbors belonging to the same cluster as the original area. We illustrated this in the following figure : 
![image.png](attachment:image.png)
> Here the score computed for area 1 is two, area 1 belongs to the red cluster and has two red neighbors. 

Once we have computed this score for every area of the dataset we group the result per cluster by averaging the previously computed scores. The obtained result is the following: for a given cluster (between 0 and k)  we know the expected number of neighbors belonging to the same given cluster. Since this metric depends on the average number of neighbors in the graph we cannot simply say that the obtained number is high or low. To have a comparison basis we repeated the procedures described above but using random assignments for the labels. 

This new metric is defined more locally. A cluster can be split into two parts located at opposite sides of London and still obtain a good score (what we want) while still being fairly general (we do not want to design a metric that will always return a good score for what we are trying to show).

#### Data preparation

In this section, we will simply preprocess our dataframe so that it contains the number of border neighbors for each area as an extra column:

1. The first step will be to add k-means and random clusters assignments to our original geopandas lsoa dataframe
2. Next, we will compute the set of neighbors for each area and from it we will compute the number of those neighbors that have the same cluster as the considered area.

> The added columns are : ```cluster_border_neighbors_k``` and ```cluster_border_neighbors_random_k``` for all considered ```k```.

#### Border Scores computation

Now that, we have a functional dataframe, we can use a ```groupby``` operation to compute for each cluster, the mean border scores. 
Additionaly, we will compute the same scores for the random clusters assignments. 

Since for a given ```k```, on average we should have the same number on points in each cluster for a random clustering, we will aggregate one more time the result (using a second ```mean```) to get an even more realiable estimate of the real random border score. We can do that since we do not care about the differences induced by the clusters in a randomized set-up (in theory they should be none).

####  Results

It would be nice to have a good plot to help us undersand the previous matrix. In the next cells, we are going to produce several ```barplots``` (one for each value of ```k```). We will then be able to compare the performance of our clustering compared to a random one.


At first sight, this already looks quite promising but let's dive into the details:

1. For each ```k```, almost all clusters do better than random.
2. When ```k``` increases, the clusters tend to ressemble less and less to the random one, on average.
3. When ```k``` increases, clusters with a score extremely low (close to 0) appear


Using this metric and the visualisation, we are finally able to somehow demonstrate that there is indeed some evidence that the clustering on typical product produces, at the same time geographically similar clusters. 

If this result at least shows that we can come up with some metric to evaluate such a clustering task, it would be better if we got the same result using a different approach, just to assess the validity of our findings. However, due to the time constrains implied by this assignment, we decided not to take further steps in that direction.

# (c) Cluster analysis
## Cluster typical products 
In this section we will analyze our clustering through the differences in the average typical product between the clusters. We choose k=4 and the ```Ward``` aggregation level in order to be able to relate our analysis to the number of diabetes within each cluster later on (data in ```diabetes_estimates_osward_2016.csv``` is only available for the ```Ward```)

We start by loading the data and put it in a nice form for comparison between clusters.

We can now plot the differences in the nutrients between the clusters. We also compute the entropy of these to detect which nutrient class is more determining and important to separate the clusters. The smaller the entropy, the larger the importance of the corresponding nutrient.

![Alt text](/images/analysis-cluster-typical-product.png){:class="img-responsive"}

We see from both the graphs and the entropies that ```carbohydrates```, ```sugar``` and ```energy_tot``` are the most determining nutrient classes to separate the clusters (we have more information about the cluster of a given area if we know about the amount of carbohydrates its typical product has than any other nutrient class). On the opposite side, the amout of proteins doesn't give much information.

This information can be used to answer one of our first questions : 
> "In the context of an advertising campaign, which areas of the city should be associated with which kind of products?"

For example, a company selling a product that has a high concentration of carbohydrates and sugar can focus their advertising campaign on areas that belong to the 2nd cluster. 


## Diabetes prevalence comparison

According to the World Health Organization, the three best dietary habits to prevent conditions associated with the metabolic syndrome are:
 1. limiting the intake of calories; 
 2. having a nutrient-diverse diet; and 
 3. favoring the consumption of fibers and proteins over sugars, carbohy- drates, and fat.

Given the previous plot and the metabolic syndrome conditions that are strongly linked to food consumption habits, we could expect to be able to observe differences in terms of the number of diabetes within each cluster.

Indeed, we could expect the cluster 2 to have a higher diabetes prevalence, followed by cluster 0 since their typical product's amount of sugar, fat and total energy is higher than for the two other clusters (this is at least what common sense could let us think). Let's verify this with proper data analysis. We will use the dataset that was used in the tesco paper to perform this task: ```diabetes_estimates_osward_2016.csv```.

Let's first plot the diabetes prevalence average for each cluster.

![Alt text](/images/analysis-cluster-diabetes.png){:class="img-responsive"}

It looks pretty clear on the plot that our intuition was good : there is a clear difference between clusters. Even though the errors bars between cluster 0 and cluster 2 seem to overlap, both are significantly higher than the two other clusters.

# (d) Practical usage of clustering
## A feature for ML models
In this section, we will do a replication of the obesity regression model definied in ```Tesco``` paper and we will try to improve it by including a new categorical feature : the clustering. We hope to be able increase the amount of variablity explained by the model (```R2```). Obsivously, we will have to quantify the improvement and choose a ```k``` high enough to have a meaningful impact on the model.
### Data loading and scaling
In the following cell, we will simply ready the same data avialable for the replication

1. ```osward  ```: osward tesco data for the entire year 
2. ```diabetes```: diabete prevalence data for all area_ids

We will study the impact of the following values of ```k```

> 2,3,...,9

Therefore, we will compute the clustering assigments as done many times before and store them in the ```cluster``` variable. 

### First try  
As a first step, we could run a simple OLS model that only takes the clustering features as dependant variable. This will directly tell us if there is any hope for them to constitute good features. Let's choose arbitrarely ```k = 4``` for this simple test.

| Variables             | Coef          | p-value  |
| --------------------- |:-------------:| --------:|
| Intercept             | 0.2729        | 0.000    |
| C(cluster_4)[T.1]     | 0.0973        |   0.000  |
| C(cluster_4)[T.2]     |0.2820         |    0.000 |
| C(cluster_4)[T.3]     |0.1604         |    0.000 |

This is confirmed : there is a significant difference between all clusters (p-value < 0.05). The R-squared is not very high though (```R2 = 0.36```), which means that a lot about diabetes prevalence is not explained by this very simple model (which is natural). But still, we managed to show some important differences between the clusters, which suggests that our clustering is meaningful.

### Replication : Table 2 (OLS on the number of diabetes prevalence) 
We are ready to perform the actual regression task. Will first try to replicate the model summarised in table 2 of the Tesco paper in order to have a comparison basis

| Variables             | Coef            | p-value  |
| --------------------- |:---------------:| --------:|
| Intercept             |  0.9357         | 0.000    |
| energy_carb           | 0.1887          |   0.004  |
| entropy               |-1.0245          |    0.000 |
| avg_age               | -0.1192         |    0.004 |
| female                | -0.1237         |   0.012  |
| transactions          |0.2851           |    0.000 |
| density               | -0.0766         |     0.025|

If we compare our result to table 2 in the paper, we find that our model is almost equal to the one used in the paper (```coef``` are all equal up to the 3rd decimal, standard errors are all equal and the ```R2``` matches). We have thus managed to create a similar set up as in the paper. Since we know that the variable used are relevant for this prediction task, we can refer to this model when we will study the impact including the clustering assignments as extra features to the model. 

The import metric to remember is the ```Adj R2 = 0.613```, we will use this to compare with the models definied in the following cells.

### Cluster Features 

To evaluate the preditive power of te clustering assignments, we will simply run the following experiment :

> For each value of ```k```, we fit a OLS using the same independent variables as in the paper but we will also add the categorical variable ```cluster_k```.

We record the the following output metrics:

1. ```Adj R2```: standard metric of a regression task
2. ````mean significant clusters````: the number of clusters having a p-value smaller than ```5%```, meaning that we can reject the hypothesis that the fitted correspondant clusters actually have a 0 value (and therefore have some predictive power). We simply take the ```mean``` to be able to compare this between between various values of ```k```

![Alt text](/images/analysis-cluster-regression.png){:class="img-responsive"}

The results look interesting, we observe an increase in ```Adj R2``` when ```k``` grows, reaching a peak at ```k = 8```. The maximum value ```Adj_R2* = 0.685``` constitutes a significant improvement compared to the value of the base model (```Adj_R2* = 0.613```). With the fact that ```71%``` of the clusters are statisfically signicant, we can conclude that the added clusters have indeed some non negligeable predictive power. This could be explained by the fact that, as seen previously, the clustering assignments in themselves encode some geographical value which could have been exploited by the OLS model.

The previous plots also tells us that a certain number of clusters (not to low value of ```k```) is required in order to observe a good increase in ```Adj R2```. Using the fact that the proportion of signicant cluster is somehow high for values of ```k < 8 (optimum here)```, a possible explanation might be that the preditive power hidden in the clusters is there but the too small number of clusters does not allow the OLS algorithm to fully decode all the geographical information. Augmenting the granularity appears to solve this problem, until we reach a peak, after which the clusters start to lose predictive power (more noise and less predicable when ```k``` grows).

#  (e) Conclusion
Through this project we created some tools, such as an interactive map, to visualize London's areas and their corresponding typical product as given in the Tesco dataset. To get deeper into this analysis, we decided to use some unsupervised machine leaning techniques such a K-means clustering. We saw that clustering can be used to visualize the high dimensional typical product in 2D. We then displayed the resulting clusters on the map, which helped us to highlight some link between typical product of the different areas and their geographical location. To formally verify this link, we tried different approaches and, after some inconclusive attempts, we finally managed to formally support our claims.

We then analyzed our clustering a bit deeper. We assessed the difference between the clusters in term of the average typical product of each cluster. We showed some huge difference and explained how this could be used in finding the optimal target region in the context of an advertising campaign for example (which answered one of our first questions). We also cross-referenced this clustering data with London's data about diabetes prevalence to try to assess some underlying link. This was successful and some significant differences in the diabetes prevalence was highlighted between the clusters.

We believe that the notebook answers all of the initial questions, by the (interactive) visualizations and the formal analysis. There are other areas in which such an analysis can be useful, such as :
* Adapt product offer/prices for grocery shops
* Supply chain management improvements : optimize truck journey
* Insurance : adapt prices according level of risk induced by food on health according to geographical areas
* Marketing / Advertisement
* Trash : create new installations/facilities to better fit the demand at geographical level: add more plastic trashes in clusters with high consumption of sugar for instance

There are many fields of application of such a study. In the end, we can easily imagine how we could extend our work further. All of our study raises more question related on food : can we relate wealthier area in terms of the clustering found, etc.
