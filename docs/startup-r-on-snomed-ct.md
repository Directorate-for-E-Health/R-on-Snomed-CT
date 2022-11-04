

# R on Snolytical (Snomed Analytics)

Author: Ulrich Lichtenstein Sønderskov Andersen

## Introduction 

These guidelines show how to starting analyzing data, using SNOMED CT concepts. 

The data is generated with [Snolytical - Health Data Analytics Demonstrator - GitHub](https://github.com/IHTSDO/health-data-analytics), and fetched using the Elastic API.  

Then RStudio is used to work with the IHTSDO dataset.

## Prerequisites 

The size of the dataset requires 8GB RAM

 ![RAM required](images/startup%2001%20-%20RAM.png)

## Load the data from the elasticSearch instance, where the snolytic data are persisted
You need to install the package “jsonlite”
``` r
	install.packages("jsonlite")
```

…and load it to the memory:
``` r
		Library(jsonlite)
```

Now it is time to load the data:
``` r
	sct2 <- fromJSON("http://<elastic-server>:9200/patient/_search/?q=_id:*&size=1250000")
``` 

Transform the list object to a dataframe
``` r
	sct3 <- sct2$hits$hits
``` 

Now, your environment should look like this

![environment](images/startup%2002%20-%20environment%20sct2%20sct3.png)

Drill down to the patient data, and store in a new dataframe
``` r
	sct4<-sct3[[5]]
``` 

Cleaning the environment:
``` r
	rm(sct2)
	rm(sct3)
``` 

Now,  your environment should look like this

![clean environment](images/startup%2003%20-%20environment%20%20sct4.png)

The graphical presentation or statistical analysis can begin...
 
## How to create graphics
Ex. See guide: https://www.statmethods.net/graphs/bar.html

Accordingly, do
``` r
	gendobYearTable<-table(sct4$gender,sct4$dobYear)
	barplot(gendobYearTable,main = "dobYear/gender (n=1.248.322)",col=c("red","darkblue"),legend = rownames(gendobYearTable))
```  

..and get

![dob-gender](images/startup%2004%20-%20dobYear%20gender.png)

 ..or, do
 ``` r
	genNumEncountTable <- table(sct4$gender, sct4$numEncounters)
	barplot(genNumEncountTable,main = "numEnc/gender (n=1.248.322)",col=c("red","darkblue"),legend = rownames(gendobYearTable))
``` 

..and get

![numEnc-gender](images/startup%2005%20-%20numEnc%20gender.png)
 


….or, do
``` r
	boxplot(sct4$dobYear~sct4$numEncounters, main ="distribution", xlab = "number of encounters", ylab = "dobYEAR")
``` 

.. and get

 ![distribution](images/startup%2006%20-%20distribution.png)
 
## How to flatten the nested dataframe of encounters

``` r
install.packages("tidyr")
library(tidyr)
sct6<-unnest(sct4, cols= c('encounters'))
```  




 
## How to filter with subsets

Import from Snowstorm
``` r
	breastCancer <- fromJSON("https://<terminologi-server>/browser/MAIN%2FSNOMEDCT-NO/concepts/254837009/children?form=inferred&includeDescendantCount=false")
```  

Transform to vector
``` r
	BCvec<-unlist(breastCancer$conceptId)
``` 

Now the vector can filter:
``` r
	sct6$roleId[as.character(sct6$conceptId) %in% BCvec]
``` 

..listing roleid for all patients with <<Breast cancer:

``` 
 [1] "34019"   "49559"   "75571"   "136605"  "144382"  "148324"  "145944"  "164461"  "181952" 
[10] "257047"  "391963"  "404579"  "312698"  "361919"  "368950"  "237558"  "193729"  "225509" 
[19] "251505"  "246092"  "446600"  "450192"  "486403"  "494222"  "502018"  "512137"  "610970" 
[28] "536033"  "586818"  "652616"  "662658"  "693294"  "714607"  "788416"  "785586"  "778924" 
[37] "929906"  "942493"  "829142"  "829918"  "881261"  "824387"  "842672"  "874914"  "855532" 
[46] "1038569" "1054902" "1080471" "1075940" "1135138" "1135815" "1157941" "1153566" "1092175"
[55] "1085708" "1124887" "975796"  "1051655" "1175088" "1174994" "1186335" "1211815" "1227282"
[64] "1228370"
``` 

 
Now, the subset of Breast Cancer patients:

![dob-breastCancer](images/startup%2007%20-%20dobYear%20gender%20BreastCancer.png)

