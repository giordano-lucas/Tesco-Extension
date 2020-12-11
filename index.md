# Link
[Paprr](https://giordano-lucas.github.io/ADA-2020-Tesco-Extension/)

# Introduction

We try to give an answer to the following questions:

1. Are similar areas in terms of typical products geographically close?
2. Can we naturally cluster areas geographically when it comes to food consumption in London?
3. How do those clusters differ when we vary the aggregation level?
4. In the context of an advertising campaign, which areas of the city should be associated with which kind of products?

Before answering those questions we will first visualise our dataset to understand if we could even hope to be able to demonstrate such a relationishp between product and geography.

# (a) Interactive visualization
### Data loading
## Statistical GIS Boundary Files dataset

In the section, we will use ```geopandas``` to load the [Statistical GIS Boundary Files for London](https://data.london.gov.uk/dataset/statistical-gis-boundary-files-london). Even if this dataset was made avialable in 2011, we should note that the boundaries haven't really changed compared to 2015 (date of Tesco Dataset) and it should not impact our visualisation.

For each aggregation level ```LSOA```, ```MSOA```, ```WARD```, ```BOROUGH``` we read the correponding file and retreive the following subset of columns: 

* ```area_id```: the id of the area considered
* ```name```: the name of the area considered
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

1. ```plot```: map of colors for the selected areas
2. ```btn_period```: button to select the period
3. ```btn_agg_level```: button to select the aggregaton level
4. ```select_feature```: button to select the feature of typical product to display (or clustering)

it then adds the respective event handers and links the components to the ```doc```

{% include_relative images/vizu.html %}

After playing a bit with the visualization, we can easily convince ourselves that we can indeed recognize some dependencies between typical product features and the geographical location of the areas. 
To capture the general trends (not by individual products) we will try to cluster the areas according to their typical product properties

# (b) Clustering Validation

We will start by studying the output clustering of a K-means algorithm to undertand which values of $k$ will produce the most natural clustering __*in the typical product space*__. We will then try to make some sense of out this clustering by performing a vizualization in 2D using several dimentionality reduction techniques. We might hope that we will already be able to see some geographical structure in those plots (recognize somehow the map of London). If not we will have to apply other techniques to help access the geographical validation of the clusters, if any.

## Naive Analysis
In this naive Analysis will not take into account the geographical aspect of the dataset. Since we only want to get a feeling about how k-means performs with the ```tesco``` data, we will select a subset of the dataset for this analysis. To be as precise as possible we study the smallest aggregation level for the entire year period.


### K-means 2D visualisation

#### Dimensionality Reduction : PCA and TNSE

In the previous point, we tried to formally understand the goodness of fit for the clusters found. Here, we will take a more visual approach. To do so we naively project the data on a 2D space using PCA and T-SNE algorithms. We plot the results and label them with the labels produced by the clustering (choose k=3 among the candidates we listed earlier), hoping that the obtained figure will look like the map of London. We obtained the following figure:

{% include_relative images/naive-k-means-plot.png %}

Here we understand that k = 2 might be a better choice if we think of TSNE. 

If we go back to our initial question, we tried to understand if there was a relationship between similar typical products and similar areas (geographical areas). At the beginning of this notebook, we thought that we might be able to recognize London from those simple dimensionality reduction 2D plots but this is clearly not the case. This does not mean that the previously mentioned relationship does not exist, it simply we have to think more carefully about how we are going to address the question. Thus we need to proceed with a deeper analysis of the clusterings in the geographic space.



