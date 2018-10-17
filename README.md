Where the pets are?

1. Introduction

Problem: A client wants to open a pet store in Toronto and has asked for advice about where it should be located.

Ideally we want to target an area that appears to be underserved currently by pet stores/services.  
To do this analysis, we will use the following information:
* the number of pets in each area in Toronto (using Forward Sortatation Areas (FSAs)) (estimated by the number of new dog/cat licenses issued per FSA)
* the neighbourhood name of each FSA (retrieved from Wikipedia)
* the popyulation of each each FSA from the 2016 census
* the foursquare api to retrieve the current number of pet stores/services per FSA

Using the above information, we will
* cluster neighborhoods to find areas with a relatively high number of pet licenses and low number of stores -- therefore possible candidates for a new pet store
* compare pet licenses from 2013 to 2017 to get an idea of which FSAs are showing a growth in pets -- and which are therefore also possible candidates for a new pet store
* look at the proportion of pets/population in each FSA

2. Data Sources

Data used in this project
---------------------------------
Population by FSA from Census 2016: https://www12.statcan.gc.ca/census-recensement/2016/dp-pd/hlt-fst/pd-pl/comprehensive.cfm
 * this information will be used to calculate the proportion of pet ownership per FSA
 * this data set has 3 columns of interest: FSA / Province / Population, 2016
 * We will filter this data set to only include the FSAs located in Toronto
 
Toronto Open Data: Licensed Dogs and Cats Reports for 2013 and 2017 https://www.toronto.ca/city-government/data-research-maps/open-data/open-data-catalogue/community-services/#a666d03a-bafe-943a-e256-3c2d14b07b10
* this information includes the number of dog and cat licenses issued per FSA in 2017.  Also planning ot look at same file for 2013 to look at which areas are experiencing growth in pet ownership
* this data has four columns: FSA / # Cat Licenses Issued / # Dog Licenses Issued / Total Licenses

FourSquare Api: 
* to get number of pet stores/services per FSA
* specifically, this query will be filtered to look for only the following two categoryIds:
5032897c91d4c4b30a586d69=pet services
4bf58dd8d48988d100951735= pet stores
* a sample query looks like this:
https://api.foursquare.com/v2/venues/explore?client_id=<id>&client_secret=<secret>E&v=X&ll=43.642960,-79.371613
&radius=400&limit=100&categoryId=5032897c91d4c4b30a586d69,4bf58dd8d48988d100951735

geopy: 
* to get lat/long coordinates for each FSA, which is used when calling Foursquare API for each FSA

Wikipedia: https://en.wikipedia.org/wiki/List_of_postal_codes_of_Canada:_M
* to get neighbourhood names for each FSA 

3. Methodology

Overview of clean up, EDA and machine learning processes used:

Data Clean up:
--------------
* check Datatypes and convert any fields that are string to float (i.e. the Dog and Total counts from the Toronto pet licensing data set were imported as object/string but need to be converted to float)
* check for and drop null/na rows
* dropped all the extraneous columns from the Population / Census data set except for the FSA and Population in 2016

Retrieval of Neighbourhood names
---------------------------------
* the Toronto Pet Licensing data came organized by FSA but assigned no names to those areas.  To get that information, 
I used Beautiful Soup to scrape the Wikipedia page (https://en.wikipedia.org/wiki/List_of_postal_codes_of_Canada:_M)
* the Wikipedia page has many rows for each FSA (Since many of them span multiple neighbourhoods) and I wanted to have one row per FSA
with all the included neighborhood names.  So I looped through the rows in the html table, checked if the FSA was already captured"
   * yes: append to list of neighhourhoods for that FSA
   * no: start a new row
* when merging the various datasets together, I merged on FSA and again dropped any null rows

Retrieval of Latitude/Longitude for each FSA
--------------------------------------------
* used GeoPy to retrieve lat/long for each FSA

Check for Outliers
------------------
* I found one outlier in the dataset, which was FSA=M5W that had a population of only 15 people.  This is significantly lower than 
   the population of any other FSA (the next lowest had a population of 2000) and would skew the results.  So I dropped this row
   
   
Exploratory Data Analysis
--------------------------
* In the EDA, I calculated and added a number of new columns:
   PropTotal_2017: proportion of the licenese granted in Toronto in 2017 that were granted to this FSA
   PropDogs: of the licenses granted in this FSA, what percentage were granted for Dogs vs Cats?
   PetChange: Total Pets 2017 - Total Pets 2013
   PropTotal_2013: proportion of the liceneses granted in Toronto in 2017 that were granted to this FSA
   PetChange_Prop: PropTotal_2017 - PropTotal_2013 (change in proportion of pets belong to this FSA)
   PerCapitaPetsTotal: pets/pop in FSA (number of new pets registered in FSA per person
   Venues: retrieved from FourSquare (total pet stores and services in FSA)
   PetServicesPerPerCapitaPet: Venues/PerCapitaPetsTotal rough estimate of how many services exist per PerCapitapet
* I produced a number of Choropleth diagrams showing the above data (comparing licenses issued across the FSAs in 2017, changes
in license counts between 2013 and 2017 and per capita pets)

Retrieval of FourSquare data
----------------------------
* After retrieving the FourSquare data on pet servivces/stores, I did a check to see how many pet stores were being counted as
belonging to more than one FSA.  I found that there were some stores  being counted twice, but decided that it doesn't matter for the
purposes of this analysis.  If a store is within a 1000m of more than 1 FSA, then it's still local and usable by people in those
FSAs and that's what we care about here.  
* when generating the count of venues per FSA, I realized that I was losing the FSAs with zero venues from the dataframe.  so I had 
to take special steps to preserve those data points and merge them back in later


Machine Learning
---------------
For this analysis, I used the KMeans algorithm to attempt to cluster FSA areas based on three factors:
* Population
* Total Pets registered in 2017
* number of Existing Pet store/services

I wanted to try to isolate FSAs which share characteristics suggesting that they're good candidates for a new pet store/service: 
fairly high population, high number of pets and low number of existing services

Since population, total pets and number of venues are all on very different scales, I used StandardScaler to scale the results based on 
standard deviations from the mean

To find the appropriate number of clusters, I used the visual Elbow method to find the point at which the score levels off -- meaning 
that adding more clusters doesn't dramatically reduce the error any further.  I found that 7 clusters looked to be correct for this data

4. Results

Pet Ownership in Toronto 2017:
* Largest # Licenses issued to areas that are outside downtown core: East York (M4C) and Etobicoke (M8V) rank highest
* Lowest # of Licenses issued to areas in the downtown core: M5C, M5H and M5W rank lowest

Cat vs Doc Licensing:
* only three areas in City have less than 60% dog licensing: East York, North York and Scarborough
* The highest proporiton of dog ownership is in Downtown Toronto (M5H) at 86.4%. 6 areas in Dowtown and Central Toronto all have ratios over 80%

Changes in Pet Licensing between 2013 and 2017:
* Downtown Toronto (M5A and M5V) experienced the largest pet growth (+ 399 and 362 respectively) (this could be due to population growth downtown with all the new condos 
* Scarborough and Etobicoke (M1C and M9B) experienced the biggest decrease in pet licensing (-282, -203)

Pet Capita Pet Licensing (2017):
* The highest per capita ownership of pets is in 
    * The Beaches (M4E): 
    * Parkdale/ Roncesvalles (M6R) 
    * Alderwood/Long Branch(M8W)

Candidates for a new pet store/service
---------------------------------------
Identified 2 clusters that appear like good candidates for opening a new Pet Store:
* these areas have 
    * Low services per capita pet
    * High number of licensed pets in 2017
    * Medium to High populations
* the FSAs in these areas are located outside of the downtown core
* Within these clusters, 5 FSAs jump out as particularly ideal for our purposes
   cluster 4: 
      M2J (Fairview, Henry Farm, Oriole) (north west of city)
      M9V (Albion Gardens, Beaumond Heights, Humbergate, Jamestown, Mount Olive, Silverstone, South Steeles, Thistletown) (North east of city)
      M1W (L'Amoreaux West, Steeles West) (north west of city)
      M6M (Del Ray, Keelsdale, Mount Dennis, Silverthorn) (East end of city)
   cluster 1: 
      M6E (Caledonia-Fairbank) (East end of city)
      M9B (Cloverdale, Islington, Martin Grove, Princess Gardens, West Deane Park) (East end of city)

5. Discussion

Caveats
-------
Since we don't have a way to accurately measure actual numbers of pets living in Toronto, I'm using newly issued licenses as a proxy. 
But this may not be a perfect measure since 
1) not every owner registers their pet 
2) not all pet types are registered (i.e. there could be a cluster of rabbits living in one FSA that we have no idea about!) 
3) we're really only looking at licenses issued in 2017 which doesn't cover pets registered in the past 10–20 years

Recommendations
---------------
My recommendation would be to target the FSAs located in Clusters 1 and 4 when looking for a location to open a new pet service/store
These areas have 
    * Low number of existing services/venues
    * Med-high number of new pet licenses issued in 2017
    * Medium to High populations
 
    
Within these clusters, 5 FSAs jump out as particularly ideal for our purposes
   cluster 4: 
      M2J (Fairview, Henry Farm, Oriole) (north west of city)
      M9V (Albion Gardens, Beaumond Heights, Humbergate, Jamestown, Mount Olive, Silverstone, South Steeles, Thistletown) (North east of city)
      M1W (L'Amoreaux West, Steeles West) (north west of city)
      M6M (Del Ray, Keelsdale, Mount Dennis, Silverthorn) (East end of city)
   cluster 1: 
      M6E (Caledonia-Fairbank) (East end of city)
      M9B (Cloverdale, Islington, Martin Grove, Princess Gardens, West Deane Park) (East end of city)

Of these, only M2J shows an increase in pets over last 5 years and may be particularly worth a closer look

6. Conclusions

In this project, I wanted to identify areas in Toronto that might be good candidates for opening a new pet store.  I pulled together
information from a variety of sources - downloading csvs from Open Data portals, calling apis and scraping a website.  Compiling all 
this diverse information provided a more complete picture about the various Toronto neighbourhoods, including population, number of new 
pet licenses issued and number of existing venues.  Based on this data, I used the KMeans algorithm to try to cluster the FSAs and 
identify areas that had high population, a high number of pet licenses and a low number of existing services. 

This analysis suggested 5 FSAs (M2J, M9V,M1W, M6M, M6E, M9B) were the best candidates for a new pet store/service

In particular, I would recommend focusing on M2J, located in the north-west of Toronto



* this project was completed for the IBM Applied Data Science Capstone on Coursera: https://www.coursera.org/learn/applied-data-science-capstone
