Products clusters and interactive  visualization
======
[Data Story](https://giordano-lucas.github.io/ADA-2020-Tesco-Extension/)
## Abstract
While the paper establishes the validity of the Tesco 1.0 dataset we propose to use this dataset to study if we can find some similarity in the typical products consumed within geographically close areas. To do so we will study the clusters of areas computed out of their products consumption : areas with similar typical products consumption will be clustered together. And then we will compare these clusters with the geographic disposition of the areas. Such a clustering could help grocery shop compagnies to adapt their product according to the areas where they operate. Moreover, to provide a better understanding of the data we will create an interactive visualization. The latter will represent typical food consumption of London areas on a map of the city with different levels of aggregation, over different periods of time and with the possibility of choosing different attributes of the typical product. This visualisation will help assess the validity of our findings. Finally, we will spend some time analysing the actual output of the clustering in terms of the typical product of the clusters. Using the dataset of diabetes prevalence, we will try to quantify how much information is contained in the clusters and how usefull can it be (study the impact of the cluster assginment as features of a linear regression compared to the base regressor of the tesco paper).

## Research questions

1. Are similar areas in terms of typical products geographically close?
2. Can we naturally cluster areas geographically when it comes to food consumption in London?
3. How do those clusters differ when we vary the aggregation level?
4. Are we able to validate the clustering results in a similar way as in the Tesco paper? Do the clustering contains valid and usable information?
5. In the context of an advertising campaign, which areas of the city should be associated with which kind of products? 

## Proposed datasets

1. [Tesco grocery 1.0](https://figshare.com/articles/Area-level_grocery_purchases/7796666) -- This dataset contains all the necessary information to create the clusters described above. It is more general than the version made available in the [course](https://drive.google.com/drive/folders/19mY0rxtHkAXRuO3O4l__S2Ru2YgcJVIA) since it contains information for each month of 2015 as well as for the entire year.
2. [Diabetes estimates Osward](https://drive.google.com/drive/folders/19mY0rxtHkAXRuO3O4l__S2Ru2YgcJVIA) --  This is the dataset used in the course to perform the first replication.
3. [Statistical GIS Boundary Files for London](https://data.london.gov.uk/dataset/statistical-gis-boundary-files-london) -- This dataset will help us to relate the areas of the Tesco dataset with their real geographical positions, needed for the visualization.

## Methods :

1. For the visualization task, we will use the library ```geopandas``` and ```bokeh```. In this visualization we will include a button and check boxes for the time (month) variable of the represented data,  the aggregation (ward,lsoa,msoa) level and the feature of the typical product that will allow the user to vizualize only a subset of the dataset. 
2. For the clustering part, we will cluster the areas according to the product consumption of their products using the two following methods:
  2.1. Apply the k means algorithm on the data (need also to find suitable k)
  2.2. Try to geographicaly validate the clustering using the following methods:
    * Use techniques seen in class to select the righ value of ```k``` and  use dimentionality reduction to visulazie the clusters in 2D
    * Find revelant geographical related metrics to evluate the good-ness of fit. They will help us find formal evidence that there is (or not) a relation between geography and the clustering. For instance, we could use diffenre distance metrics when the silhouette score is computed. We also plan to use a graph based approach to compute such metrics (vertices are areas and edges link  two areas if they share a phyiscal border). 
3. Regarding the analyis of the clustering output, the following methods will be applied:
  3.1. For each cluster compute its typical product (average of the typical product of the areas contained in the cluster) and study the differences observed. We will then relate those differences in terms of the metabolic syndrome related to diabetes prevalence (found in the tesco paper).
  3.2. Quantify the predicte power of the clustering assignments to assess the information contained in the clustering. In order to do so, we will replicate the regression model (table 2. of the paper) on the number of diabetes prevalence in London and analyse the improvements (```R2``` for instance) when the clusters are added as dependent variable to the model compared to the base model of the paper. 
4. Conclude by proposing other possible practical usage of the clustering output.

## Timeline and contributions :

### Week 1 : vizualisation

| Task                                                                    | Team member(s)                  | work hours  |
| :-----------------------------------------------------------------------|:--------------------------------| -----------:|
| Literature research                                                     | Lucas Giordano                  | 2h          |
| Architecture and state (data handling)                                  | Lucas Giordano & Augustin       | 2h          |
| Vizual components (buttons,...)                                         | Lucas Gruaz                     | 2h          |
| Handling user interactions                                              | Augustin &  Lucas Gruaz         | 2h          |
| Putting everthing together and code cleaning                            | Lucas Giordano                  | 2h          |
| Trying to convert python code into  JS for data story (without sucess)  | Lucas Giordano                  | 6h          |


### Week 2 : cluster validation

| Task                                     | Team member(s)                  | work hours  |
| :----------------------------------------|:--------------------------------| -----------:|
| K-means + dimensionality reduction       | Augustin                        | 1h          |
| Naive Silhouette analysis                | Lucas Giordano                  | 1h          |
| Cluster vizualization                    | Lucas Giordano                  | 2h          |
| Geographical Silhouette                  | Lucas Giordano & Augustin       | 4h          |
| Geographical graph analysis              | Lucas Gruaz & Augustin          | 4h          |
| Border scores                            | Lucas Giordano & Augustin       | 4h          |
| Data cleaning + incorporate vizualization| Lucas Giordano & Gruaz          | 3h          |


### Week 3 : cluster analysis & data story

| Task                                    | Team member(s)                  | work hours  |
| :---------------------------------------|:--------------------------------| -----------:|
| Cluster typical product                 | Lucas Gruaz                     | 2h          |
| Replication of regression model (Tesco) | Lucas Giordano                  | 2h          |
| Cluster regression analysis             | Lucas Gruaz & Augustin          | 2h          |
| Conclusion                              | Lucas Gruaz                     | 1h          |
| Data cleaning                           | Lucas Gruaz                     | 2h          |

### Delivrables data story and presentation:

| Task                                    | Team member(s)                  | work hours  |
| :---------------------------------------|:--------------------------------| -----------:|
| Notebook comments and markdown          | Lucas Giordano & Gruaz          | 3h          |
| Github pages literature research        | Lucas Giordano & Augustin       | 2h          |
| Data story                              | Lucas Giordano & Augustin       | 5h          |
| Presentation                            | Lucas Gruaz    & Augustin       | 3h          |

### Total contribution:

| Team member                     | work hours   |
|:--------------------------------| ------------:|
| Lucas Giordano                  | 38h          |
| Lucas Gruaz                     | 24h          |
| Augustin Kapps                  | 35h          |


## Notes to the reader
### Organisation of the repository
In order to be able to run our notebook, you should have a folder structure similar to:

    .
    ├── data                                      # Data folder
    │ ├── diabetes_estimates_osward_2016.csv      # [Diabetes estimates Osward](https://drive.google.com/drive/folders/19mY0rxtHkAXRuO3O4l__S2Ru2YgcJVIA)
    │ ├── all                                     # folder containing [Tesco grocery 1.0](https://figshare.com/articles/Area-level_grocery_purchases/7796666)
    │ │  ├── Apr_borough_grocery.csv              # example of file
    │ │  ├── ...
    │ ├── statistical-gis-boundaries-london       # folder containing the unzipped [Statistical GIS Boundary Files for London](https://data.london.gov.uk/dataset/statistical-gis-boundary-files-london) 
    │ │  ├── ESRI                                 # contains the data to be loaded by geopandas
    │ │  │  ├── London_Borough_Excluding_MHW.dbf  # example of file
    │ │  │  ├── ...
    │ │  ├── ...
    ├── images                              # Contains the ouput images and html used for the data story
    ├── extension.ipynb                     # Deliverable notebook for our extension
    ├── vizu.ipynb                          # Notebook containing only the vizualisations (if the reader only was to see the interactive viz)
    └── README.md               
    

> ```extension.ipynb``` is therefore the "single" notebook that we are supposed to deliver for the P4 submission.

Note: ```vizu.ipynb``` is only there for the people who read our data story but still want to play with the interactive vizualisation without having to go through the entire deliverable notebook.

### Dependencies requirement

Furthermore, you should have the following additional libraries installed
| Library                         |
|:--------------------------------| 
| Geopandas                       |
| Bokeh                           | 
| Augustin Kapps                  |
