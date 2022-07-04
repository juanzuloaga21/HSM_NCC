# Habitat Suitability Models (HSMs)
### Collaborators
- Juan Zuloaga and Andrew Gonzalez (McGill University)
- Nikol Dimitrov and Richard Schuster (The Nature Conservancy of Canada)

### Objectives
Develop HSMs for a selected group of species in Canada as a complementary tool to help The Nature Conservancy of Canada in conservation planning.

### Scope/Extent
- This is an exploratory analysis using coarse resolution datasets (~1km2).
- Canada

### Data
- Observations (points): <a href="https://www.gbif.org/" target="_blank" rel="noopener"><span>GBIF</span> </a>
- Extent of the analysis: we used geogrpahic range maps when available; otherwise we created a minimum convex polygon (MCP) using all available observations and buffered it using a distance of 10Km (R function/package: <a href="https://rdrr.io/cran/adehabitatHR/man/mcp.html" target="_blank">mcp {adehabitatHR}</a>).
- Predictors (rasters):
  - **Topographic heterogeneity** developed by <a href="https://www.nature.com/articles/sdata201840" target="_blank">Amatulli et al 2018</a>. Access date (September 17, 20)  <a href="http://www.earthenv.org/topography" target="_blank">on this website</a>. `Vector Ruggedness Measure (VRM)`, `Roughness`, `Eastness`, `Northness`, `Slope`.
  - **The Dynamic Habitat Index (DHI)** is an integrated metric of vegetation production from satellite imagery that measures the fraction of photosynthetically active radiation (or fPAR) intercepted by vegetation (<a href="https://www.sciencedirect.com/science/article/pii/S1470160X08000071?casa_token=r7JKpy2f-ocAAAAA:MxkcwYeyPJx-n8_i4efA3gqAWuXOcebBwILc_faNT1oP2otQFxFiF_Zvzcq9As0n0wTBnW2ATA#bib53" target="_blank">Coops et al 2008</a>). `Cummulative annual productivity`, `the minimum level of vegetation cover`, and `the degree of seasonality`. Access date: November, 2019. Spatial resolution: 0.0083 decimal degrees (~ 1 Km resolution). Datasets downloaded from: (<a href="http://silvis.forest.wisc.edu/data/dhis/" target="_blank">DHI, from the Silvis Lab</a>).
  - **BIOCLIM (1970-2000)**. Nineteen (19) bioclimatic variables from <a href="https://www.worldclim.org/data/worldclim21.html" target="_blank">Worldclim</a> (<a href="https://rmets.onlinelibrary.wiley.com/doi/abs/10.1002/joc.1276" target="_blank">Hijmans et al 2005</a>); using R function/package (<a href="https://github.com/kapitzas/WorldClimTiles/" target="_blank">tile_name, tile_get and tile_merge {WorldClimTiles}</a>).
  - **Percentage of lakes**. We calculated the number of lake-pixels (100m) within a 1000 pixel size, using lakes datasset from <a href="https://hydrosheds.org/page/hydrolakes" target="_blank">HydroLakes v1</a>.

### Methods
- We used <a href="https://www.sciencedirect.com/science/article/pii/S030438000500267X" target="_blank">Maxent</a>  algorithm implemented in the `ENMeval` v.2.0.0 package (<a href="https://besjournals.onlinelibrary.wiley.com/doi/full/10.1111/2041-210X.13628?campaign=woletoc" target="_blank">Kass et al 2021</a>) in R.
- Spatial filtering (thining): To reduce spatial sampling bias and spatial autocorrelation in species observations we thinned the occurrence data set of each species using a thinning algorithm (implemented in the <a href="https://cran.r-project.org/web/packages/spThin/index.html" target="_blank">spThin</a> package by (<a href="https://onlinelibrary.wiley.com/doi/full/10.1111/ecog.01132" target="_blank">Aiello-Lammens et al 2015</a>), using a distance of 1 Km.
- Multicolinearity: we used a correlation coefficient threshold of 0.7 (<a href="https://onlinelibrary.wiley.com/doi/full/10.1111/j.1600-0587.2012.07348.x">Dormann et al 2012</a>) to select a set of uncorrelated variables, using <a href="http://127.0.0.1:19644/library/virtualspecies/html/removeCollinearity.html" target="_blank">removeCollinearity</a> fucntion from the <a href="https://onlinelibrary.wiley.com/doi/full/10.1111/ecog.01388" target="_blank">virtualspecies</a> package in R.
- Background points (bg): we selected 'bg' based on to number of pixels in study area:
   - 40% if study area contains < 100,000 pixels
   - 20% if study area contains > 100,000 and < 1,000,000 pixels
   -  5% if study area contains > 1,000,000 pixels
 - Bias layer: we created a bias raster layer using all observations, using  R function/package (<a href="https://rdrr.io/cran/MASS/" target="_blank">MASS::kde2d</a>).  We used this bias layer to select the background points.
 - Features (fc): “The idea of Maxent is to estimate a target probability distribution by finding the probability distribution of maximum entropy (i.e., that is most spread out, or closest to uniform), subject to a set of constraints that represent our incomplete information about the target distribution. The information available about the target distribution often presents itself as a set of real-valued variables, called “features”, and the constraints are that the expected value of each feature should match its empirical average (average value for a set of sample points taken from the target distribution). When Maxent is applied to presence-only species distribution modeling, the pixels of the study area make up the space on which the Maxent probability distribution is defined, pixels with known species occurrence records constitute the sample points, and the features are climatic variables, elevation, soil category, vegetation type or other environmental variables, and functions thereof." (<a href="https://www.sciencedirect.com/science/article/abs/pii/S030438000500267X" target="_blank">Phillips et al., 2006</a>). Available features are: Linear (L), Quadratic (Q), Product (P), Threshold (T), and Hinge (H). We selected the type of features according to the number of observations (<a href="https://onlinelibrary.wiley.com/doi/full/10.1111/j.1600-0587.2013.07872.x" target="_blank">Merow et al., 2013</a>):
  -  Less than 10 observations (L)
  - 11 - 15 observations (L, Q, LQ)
  - 16 - 80 observations (L, Q, LQ, H, QH)
  - More than 80 observations (Q, H, LQ, LQP, QPT)

- Regularization multiplier (rm): The 'rm' prevents model over-fittig and smooths the distribution, making it more regular. “A smaller RM value will result in a more localized output distribution that fits the given occurrence records better, but is more prone to overfitting. Overfitted models are poorly transferable to novel environments and, thus, not appropriate to project distribution changes under environmental change. A larger RM value will produce a prediction that can be applied more widely"  (<a href="https://www.mdpi.com/1999-4907/11/3/302" target="_blank">Li et al., 2020</a>).

### Outputs (rasters 1km2)
- Habitat suitability map 
- Uncertainty map (coeficient of variation 10 runs best model)

### Tasks
- Script basic structure (COMPLETED)
- Observations: for every single species include long and lat coordinates (csv file )or shapefile. GBIF dataset???? If so, cleaning module 
- Predictors (Juan will transfer data sets to Nikol). (COMPLETED)  climate:  (<a href="https://chelsa-climate.org/" target="_blank">CHELSA</a>) `bio` data set (1980-2010)??If so, download and mask (Nikol).
- Add to script (Juan)
  - Base maps: Canada boundary map (with provinces), Protected Areas,  Cities, Roads (COMPLETED)
  - Spatial thining (COMPLETED)
  - Add predictors: Bioclim(19), productivity (3), Topographic heterogeneity (5), distance to lakes(1) (COMPLETED)
  - Stack predictors (COMPLETED)
  - Remove collinear predictors (COMPLETED)
  - Define background points (COMPLETED)
  - Create sampling bias layer (COMPLETED)
  - Model settings (COMPELTED)
  - Select best model (COMPLETED)
  - Variable importance, performance metrics (COMPLETED)
  - Add If(polygon or GBIF)
  - Identify potencial high suitable areas
  - Uncertainty map
- Markdown document: Juan generates (COMPLETED) it and Nikol develop content.
  
 
