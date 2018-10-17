Where the pets are?

1. Introduction
----------------
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


* this project was completed for the IBM Applied Data Science Capstone on Coursera: https://www.coursera.org/learn/applied-data-science-capstone
