# Introduction

Nowadays, the supply chain is one of the key concepts when managing businesses dealing with food distribution. It is the ingredient that will determine the efficiency of a distributor. We would like to see if we can come up with some tools to optimise this distribution, using the Tesco grocery dataset.
Tesco grocery 1.0 is a large-scale dataset detailing food consumption in London areas at a different level of aggregation.  Such a dataset allows us to study the different trends around London when it comes to food consumption.  

In our study, we will focus on tools that could help a firm to deploy new products on the London market in the most efficient way possible. When supplying shops with trucks, one determining factor is the scheduled route used to go towards each shops. The longer the path is the costlier it gets to deliver the food. We won't work on how to design such a path. Instead, we will aim to determine if it is even possible to deploy a product in a given area such that the neighbouring areas would also be inclined to use the product. More specifically, we will analyse if individuals from the same location tend have similar food preferences. This task will be done by clustering the different areas of London according to their food characteristics ('sugar', 'protein', etc.). Secondly, a comparison will be made between the clustering and the geographical positions of the areas.

Throughout this analysis, we try to give an answer to the following questions:

1. Are similar areas (in terms of typical products) geographically close?
2. Can we naturally cluster areas geographically when it comes to food consumption in London?
3. How do those clusters differ when we let the aggregation level vary?
4. Is it possible to use techniques similar to the those presented in the Tescod paper to validate the clustering results? Does the clustering contain valid and usable information?
5. In the context of an advertising campaign, which locations should be associated with which kinds of products?

Before answering those questions, we will first visualise the dataset to understand if we could even hope to be able to demonstrate such a relationship between product and geography.

# Getting the data

The first thing is needed to be done is to obtain a list of geographical coordinates and shapes associated to each area (and for each aggregation level ```LSOA```, ```MSOA```, ```WARD```, ```BOROUGH```).  We use the ["Statistical GIS Boundary Files for London"](https://data.london.gov.uk/dataset/statistical-gis-boundary-files-london) dataset for this purpose. Even if this dataset was made available in 2011, we should note that the boundaries haven't really changed compared to 2015 (date of Tesco Dataset) and it should not impact our visualisation.

We can proceed further by reading the [Tesco grocery dataset](https://figshare.com/articles/Area-level_grocery_purchases/7796666). This dataset was shown to be ecologically valid and can, for intance, be used to compare food purchase volumes with official statistics of food-related illnesses. During 2015, 420M food items purchased by 1.6M fidelity card owners in 411 Tesco stores across London have been recorded. For each purchase, the area in which the customer lives, the global trade item number, and the timestamp are stored. To to preserve anonymity, the data is aggregated at area level instead of storing customer information. The aggreation of all the product bought in a single area is called the __*typical product*__ of this area. Those products are identified by nutrition information (e.g.,energy,fats) and are classified into 17 non-overlapping classes (e.g.,grains,red meat). 

For the sake of this analysis, we will only consider the following  __*typical product*__  features:

> ```fat, saturate, sugar, protein, carbohydrate, fibre, energy, calories, etc.```

Locally, interactive visualisation is built. It that allows the user to explore the  data at several levels of geographical granularity, periods of time (month of 2015). It is also possible to select which typical product features are visualised. A snapshot can be found right below. 

Due to our limited knowledge of web development, we could not manage to directly embed the full interaction aspect in this website. Nevertheless, the reader is welcomed to [download the full vizualisation](https://github.com/giordano-lucas/ADA-2020-Tesco-Extension/tree/gh-pages) and play with it offline.

{% include_relative images/vizu.html %}
{% include_relative images/vizu-2.html %}

From the plot, several dependencies between typical product features and geographical locations can be established. For instance, individuals living in the center of London seem to be less enclined to eat producs with high level of  ```sugar```. More generaly, the further we are from the center, the higher is the consumption of suger, espcially in the south east border. Additionaly, a similar center cluster can be observed when it comes to ```h_nutrients_calories```.

To capture the general trends, areas will be clustered according to their typical product properties. Secondly, we will formalise this geographical dependence.

# Clustering Validation

First, we will start by studying the output of a standard clustering algorithm named [K-means](https://en.wikipedia.org/wiki/K-means_clustering). The aim is to determine which values of ```k```(the number of clusters) produce the most natural clustering __*in the typical product space*__. In the next step, we will try to make some sense out of the clustering by performing a visualisation in 2D. To achieve this goal several dimensionality reduction techniques will be applied. We might hope to be able to dectect some geographical structure in those plots (maybe recognize the map of London somehow). If not, we will have to apply other techniques to determine wether or not the clusters are valid.

## Naive Analysis
In the naive analysis, the geographical aspect of the dataset will not be taken into account. Since we only want to get a feeling about how the clustering algorithm performs with the ```Tesco``` data, we select a small subset for this analysis. To be as precise as possible, the smallest aggregation level (```lsoa```) is chosen for the entire year period. 

 As stated before, the starting point of the analysis is to understand which values of ```k``` lead to a good clustering assignment. ```silhouette``` and ```sse``` plots are the standard way to go.  

![Alt text](images/naive-metrics.png){:class="img-responsive"}

The elbow method applied on the SSE graph seems to indicate that 2 and 3 might be relevant choices of ```k```. Moreover, the silhouette graph also suggests that these values are fair tradeoffs between the goodness of the fit and the number of clusters. By curiosity, we will also keep ```k = 4``` as one of the best candidates.

""" To deeper analyse the values of the silhouettes scores obtained for the candidates,  silhouettes score are   by each data point of each cluster."""

![Alt text](/images/naive-silhouette.png){:class="img-responsive"}

Silhouette scores can be interpreted as follows :
* 1  indicates that the sample is far away from the neighbouring clusters
* 0  indicates that the sample is on or very close to the decision boundary between two neighbouring clusters
* <0 indicates that those samples might have been assigned to the wrong cluster. 

Only for ```k = 4```, we observe higher variability scores (inside clusters) what indicates that the clustering might not be so good. However, for all ```k```, we do not have any clusters with below average (red dotted lines) silhouette scores which is a good point.

In the previous paragraph, we tried to formally understand the goodness of fit for the clusters found. Here, we will take a more visual approach. To do so, we naively project the data on a 2D space using PCA and T-SNE algorithms. We plot the results and label them with the clustering assignments (choose ```k = 3``` among the candidates we listed earlier). We obtained the following figure:

![Alt text](/images/naive-k-means-plot.png){:class="img-responsive"}

Now be can back to our initial question about the potential existance of arelationship between similar typical products and similar areas. At the beginning of the story, we thought that we might be able to recognize London from those simple 2D plots but it is clearly not the case. Nevertheless, it does not mean that the latter does not exist. We simply have to think more carefully about how we are going to address the question. Thus, we need to proceed with a deeper analysis of the clustering in the geographic space.

## Geographical Clustering visualisation 

In the previous paragraphs, we failed to show the presence of a geographical relationship between products and areas. To have a better idea of how the clusters look like, another interactive visualisation is designed. The latter based on the real London map. Again, to find the interactivity feel free to take a look at the offline visualisation.

{% include_relative images/cluster-vizu.html %}
{% include_relative images/cluster-vizu-2.html %}

Surprisingly, for many values of ```k```, we can actually make some geographical sense out of the clustering! Let's analyse further some of the plots:

* For ```k = 2```, across all aggregation levels, we can generally observe that the centre and top east part of London tend to be clustered together. If we take a look at the ```lsoa``` plot (the most precise aggregation level), we clearly see several __concentric__ circle-shaped clusters. A further analysis could try to relate these clusters to actual classes of people (for instance, densely populated / wealthy areas, who knows?).
* For larger ```k```, this concentric cluster shapes tendency keeps appearing, which is really interesting given the fact that the K-Means algorithm didn't take any geographical metric into account.
* We can almost always distinguish a central cluster.

In the end, we see that the clustering looks quite decent and comforts us in the idea that we are able to give geographical meaning to them. Overall, these results are exciting and it would be interesting to formally quantify them.

## Formal Geographical Validation

In this section, we want to provide the reader evidence that the clustering contain usefull information. Thus, we have to find special metrics that could reflect the goodness of the clustering in the geographic space. 

### Geographical Silhouette

As a first step, we will try to use the silhouette score again but in a more geographical fashion. To do so, we will the score as done at the beginning of the study. However, this time the geographical distance will be used. It is defined as the flight distance between areas. In order to be able to compare the future results, we will also compute the same metric but produced by a random clustering. 
In fact, we expected the clustering to perform significantly better than the random one. 

![Alt text](/images/validation-geographical-distance-silhouette.png){:class="img-responsive"}

Unfortunately, the metric seems to indicate that the random clustering performs better. Obviously, something went wrong. Compared to the values obtained in the naive analysis, the scores are quite small. Negative values even arise, meaning that, on average, points tend to be closer to other clusters than to their own, or at least in the geographical sense. 

However, we need to take a step back and try to fully grasp why such result could occur. Indeed, we should remind ourselves how the silhouette score is computated.  It heavily relies on the difference between the average distance to other points in the same cluster and the smallest mean distance to points assigned to other clusters. This metric is thus likely to produce high scores for dense clusters distributed such as a Gaussian centred at the centroid. However, this is not the shape that can be observed in our visualisation! We rather saw concentric circles, scattered around the map. They do not form unique blocks. Indeed, if we were to run a K-means algorithm in the 2D geographical vector space, we would never get such concentric shaped clusters.  

If the silhouette metric remains a reliable tool when we want to evaluate clusters that are described in the previous paragraphs, we understood that we should probably try another approach to evaluate our clusters.

### Graph analysis

This time, a slightly different approach will be taken. We again use the silhouette score but it will be computed using the following metric: 

> the distance between two areas will be defined as the shortest path (smallest number of areas to cross) to go from one area to the other. 

The idea is that centroids of similar areas might be far from each other even if the areas are located next to each other. By giving a slightly more geographical meaning to the distance measure, we hope to be able to obtain a better cluster evaluation. Expect from this element, the procedure remains the same: we calculate the scores for various values of ```k``` and compared them to a random clustering assignment.

Note that, again, the analysis is only performed on the lsoa aggregation level. 

![Alt text](/images/validation-graph-silhouette.png){:class="img-responsive"}

The output are very similar to the previous analysis. We might have put to much hope on the changes made by the distance metric. The shortest paths is simply too close the flight distance, especially for the smallest aggregation level. The lack of evidence could be due to the fact that the silhouette score is not well suited for this application. Therefore, a more appropriate score function needs to defined. Note that despite the fact that the silhouette score is low, we are still convinced that the clustering is meaningful in the geographic space. Otherwise, the visualation would have given different results.

### Border Scores

In this third attempt, we forget about ```silhouette``` and a brand new metric is designed. For each area, we take a look at all its direct geographic neighbours and record how many how those belong to the same cluster as the original area. We illustrated this in the following figure :

![alt.png](/images/example-border-score-2.png)

> Here the score computed for all ```green``` areas is two: two neighboring areas that belongs to the same cluster. 

Once we have computed this score for every area of the dataset, we group the result per cluster by averaging the previously computed scores. Basically, the output of this process is the mean number of neighbors whithin each cluster. For a given cluster is allows to make the following statement : 
>  For each area ```A```, on average, we expect to see .... neighbouring areas that belong to the same cluster as ```A```.

This new score function is defined more locally. A cluster can be splited into two parts located at opposite sides of London and still obtain a good score (what we want) while still being fairly general (we do not want to design a metric that will always return a good score for what we are trying to show). To have a comparison basics, a parallel randomized procedure is also run, as done many times previously.

![Alt text](/images/validation-border-scores.png){:class="img-responsive"}

At first sight, it already looks quite promising. Let's dive into the details:

1. For each ```k```, almost all clusters do better than random.
2. When ```k``` increases, the clusters tend to diverge more and more from to the random one, on average.
3. When ```k``` increases, clusters with a score extremely low (close to 0) appear.  

Using this metric and the visualisation, we are finally able to somehow demonstrate that there is indeed some evidence that the clustering on typical product produces, at the same time geographically similar clusters. 

If this result at least shows that we can come up with some metric to evaluate such a clustering task, it would be better if we got the same result using a different approach, just to assess the validity of our findings. However, due to the time constrains implied by this assignment, we decided not to take further steps in that direction.

Regarding the third observation, one might wander why such cluster appear. The answer is actually quite straitforward:

> there is not a single point inside some clusters

This not so surprising result can be explained by the clustering algorithm itself. Indeed, due to bad intialisation or bad luck in the assignment phase, some clusters can end up being empty. When ```k``` increases, the likelihood of observing such cases increases. That's exactly why we should not worry about them.

# Cluster analysis

## Cluster typical products 

Now that we managed to assess the geographical validity of our clustering, let's try to use it! In this section, we will analyse our clustering through the differences in the average typical product between the clusters. We choose ```k = 4``` and the ```Ward``` aggregation level in order to be able to relate our analysis to the number of [diabetes](https://drive.google.com/drive/folders/19mY0rxtHkAXRuO3O4l__S2Ru2YgcJVIA) within each cluster later on (data is only available for the ```Ward```)

![Alt text](/images/analysis-cluster-typical-product.png){:class="img-responsive"}

We see from both the graphs that ```carbohydrates```, ```sugar``` and ```energy_tot``` are the most dissimilar nutrient classes. Indeed, we have more information about the cluster if we know about the amount of carbohydrates than any other nutrient class. Conversely, the number of proteins doesn't give much information.

Such facts can be used to answer one of our first questions: 
> "In the context of an advertising campaign, which areas of the city should be associated with which kinds of products?"

For example, a company selling a product that has a high concentration of carbohydrates and sugar can focus their advertising campaign on areas that belong to the 2nd cluster. Many other similar conclusions can be drawn from the plot and there is not point in listing them all.

## Diabetes prevalence comparison

According to the World Health Organization, the three best dietary habits to prevent conditions associated with the metabolic syndrome are:
1. limiting the intake of calories
2. having a nutrient-diverse diet   
3. favouring the consumption of fibres and proteins over sugars, carbohydrates, and fat.

Given the previous plot and the metabolic syndrome conditions that are strongly linked to food consumption habits, we could expect to be able to observe differences in terms of the number of diabetes within each cluster.

Indeed, cluster 2 is likely to have a higher diabetes prevalence, followed by cluster 0 since their typical product's amount of sugar, fat and total energy is higher than for the two other clusters. Let's verify this with proper data analysis. We will use the [dataset](https://drive.google.com/drive/folders/19mY0rxtHkAXRuO3O4l__S2Ru2YgcJVIA) that was used in the Tesco paper to perform this task. Let's plot the diabetes prevalence average for each cluster.

![Alt text](/images/analysis-cluster-diabetes.png){:class="img-responsive"}


Our intuition was good: there is a clear difference between clusters. Even though the errors bars between cluster 0 and cluster 2 seem to overlap, both are significantly higher than the two other clusters. 

# Practical usage of clustering : a feature for Machine Learning models

For the last section, a replication of the obesity regression model defined in ```Tesco``` paper will be done. Then, we will try to improve it by including a new categorical feature: the clustering. Given the previous results found earlier, we hope to be able to increase the amount of variability explained by the model (```R2```). Obviously, we will have to quantify the improvement and choose a ```k``` high enough to have a meaningful impact on the model. 

First, we run a simple OLS model that only takes the clustering features as dependant variable. It will directly tell us if there is any hope for them to constitute good features. Let's choose arbitrarily ```k = 4``` for this simple test.

| Variables             | Coef          | p-value  |
| :---------------------|--------------:|---------:|
| Intercept             | 0.2729        | 0.000    |
| C(cluster_4)[T.1]     | 0.0973        |   0.000  |
| C(cluster_4)[T.2]     |0.2820         |    0.000 |
| C(cluster_4)[T.3]     |0.1604         |    0.000 |

It is confirmed: there is a significant difference between all clusters (p-value < 0.05). The R-squared is not very high though (```R2 = 0.36```), which means that a lot about diabetes prevalence is not explained by this very simple model. That's completely understandable given the simplicity of the model. But still, we managed to show some important differences between the clusters, which suggests that our clustering is meaningful.

We are now ready to perform the actual regression task. We will first try to replicate the model summarised in table 2 of the Tesco paper in order to have a good comparison basis.

| Variables             | Coef            | p-value  |
| :---------------------|----------------:|---------:|
| Intercept             |  0.9357         | 0.000    |
| energy_carb           | 0.1887          |   0.004  |
| entropy               |-1.0245          |    0.000 |
| avg_age               | -0.1192         |    0.004 |
| female                | -0.1237         |   0.012  |
| transactions          |0.2851           |    0.000 |
| density               | -0.0766         |     0.025|

If we compare our result to table 2 in the paper, we find that our model is almost equal to the one used in the paper (```coef``` are all equal up to the 3rd decimal, standard errors are all equal and the ```R2``` matches). We have thus managed to create a similar set up as the one in the paper. Since we know that the variable used are relevant for this prediction task, we can refer to this model when we study the impact of including the clustering assignments in the model. The important metric to keep in mind is the ```Adj R2 = 0.613```. It will be used to compare with the models defined in the following cells.

To evaluate the predictive power of the clustering assignments, the following experiment is run:

> For each value of ```k```, an OLS is fitted using the same independent variables as in the paper. On top of them, categorical variable ```cluster_k``` will be included.

We record the following output metrics:

1. ```Adj R2```: standard metric of a regression task
2. ````mean significant clusters````: the number of clusters having a p-value smaller than ```5%```, meaning that we can reject the hypothesis that the fitted correspondent clusters actually have a 0 value (and therefore have some predictive power). We simply take the ```mean``` to be able to compare this between various values of ```k```

![Alt text](/images/analysis-cluster-regression.png){:class="img-responsive"}

The results look interesting, we observe an increase in ```Adj R2``` when ```k``` grows, reaching a peak at ```k = 8```. The maximum value ```Adj_R2* = 0.685``` constitutes a significant improvement compared to the value of the base model (```Adj_R2* = 0.613```). With the fact that ```71%``` of the clusters are statistically significant, we can conclude that the added clusters have indeed some non-negligible predictive power. This could be explained by the fact that, as seen previously, the clustering assignments in themselves encode some geographical value which could have been exploited by the OLS model.

The previous plot also tells us that a certain number of clusters  is required in order to observe a good increase in ```Adj R2```. Note that the proportion of significant cluster is somehow high for values of ```k < 8 (optimum here)```. A possible explanation might be that the predictive power hidden in the clusters is there but the too small number of clusters does not allow the OLS algorithm to fully decode all the geographical information. Increasing the granularity appears to solve this problem, until a peak is reached, after which the clusters start to lose predictive power (more noise and less predicable when ```k``` grows).

![Alt text](images/analysis-cluster-regression-error.png){:class="img-responsive"}

If we compare the error made by the best model (with ```k = 8```) and the base model, we observe similar behavior in the general trend. The only point where they differ is the number of really bad classified areas (points far away from the tendency line). Indeed, especially for areas having a number of diabetes higher than 0.7, it can be seen that the model including the clusters manages to capture more variablity than the base one (points closer to the tendency line). Fortunately, that's something that can be related to the cluster assignments! We observe that only the ```green``` and ```red``` clusters for large value of ```diabtes```, this means that the model was given a slight advantge when we include the clusters as features.

#  Conclusion

Through this data story, we created some tools, such as an interactive map, to visualise London's areas and their corresponding typical product as given in the Tesco dataset. To get deeper into this analysis, we decided to use some unsupervised machine leaning techniques such a K-means clustering. We saw that clustering can be used to visualise the high dimensional typical product in 2D. We then displayed the resulting clusters on the map, which helped us highlight some link between the typical product of various areas and their geographical location. To formally verify this relation, we tried different approaches and, after some inconclusive attempts, we finally managed to formally support our claims.

We then analysed our clustering a bit deeper. We assessed the difference between the clusters in terms of the average typical product of each cluster. We showed some huge difference and explained how this could be used in finding the optimal target region in the context of an advertising campaign for example (which answered one of our first questions). The clustering data was also cross-referenced with London's data about diabetes prevalence in order to assess some underlying link. It was successful and some significant differences in terms of diabetes prevalence were stressed out between the clusters.

It is believed that the our analysis answers all of the initial questions, by the (interactive) visualisations or the formal validation/analysis. Of cours, there are many more fields in which such results can be applied, such as
* Adapt product offer/prices for grocery shops
* Supply chain management improvements: optimise truck journey
* Insurances: adapt prices according to the level of risk induced by food on health based on geographical areas
* Marketing / Advertisement
* Trash: create new installations/facilities to better fit the demand at geographical levels. For example one applicatoin cours be to add more plastic trashes in clusters with high consumption of sugar.

There are many fields of application of such a study. In the end, we can easily imagine how we could extend our work further. All of our study raises more food-related questions: can we establish a relationship between wealthy areas in terms of the clustering found, etc.
