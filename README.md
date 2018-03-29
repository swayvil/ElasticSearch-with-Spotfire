# ElasticSearch with TIBCO Spotfire
## 1. Introduction
ElasticSearch is a distributed, RESTful search and analytics engine and TIBCO Spotfire is a powerful analytics platform. This document explains how to use ElasticSearch with TIBCO Spotfire.
## 2. ElasticSearch mapping
ElasticSearch index used for this example:
```json
PUT tibco-audit-servicecontext
{
    "settings" : {
        "number_of_shards" : 1,
		"number_of_replicas" : 1
    },
    "mappings": {
		"serviceContext": {
			"_all": {"enabled": false},
			"properties": {
				"contextTimestamp": {"type": "date"},
				"flowCode": {"type": "keyword", "index": "true"},
				"logEvent": {
					"properties": {
						"role": {"type": "keyword", "index": "true"}
						}
				}
			}
		}
	}
}
```
## 3. Enable R in Spotfire
### 3.1. TERR
[TIBCO® Enterprise Runtime for R (TERR)](https://docs.tibco.com/pub/spotfire/7.0.0/doc/html/terr/terr_how_to_use_terr_tools.htm) is a high-performance statistical engine that is compatible with open-source R. It is provided in your installation of Spotfire so you can script and run data functions or create predictive models. TERR Tools are provided to give you access to the TERR console to test scripts and functions, to launch the RStudio interactive development environment for script authoring, and to the TERR Language Reference for help with installed packages.

![1-TERR-Tools](https://github.com/swayvil/ElasticSearch-with-Spotfire/blob/master/images/1-TERR-Tools.png)

### 3.2. Install R curl and jsonlite libraries
Check that you are connected to Internet, then in Tools/TERR tools/Package Management:
* Select the CRAN package repository;
* Click on the "Load" button.
![2-TERR-Tools](https://github.com/swayvil/ElasticSearch-with-Spotfire/blob/master/images/2-TERR-Tools.png)

# 4. Example
## 4.1. Add a new Data Function
### 4.1.1. Audit-flowNames
In Tools/Register Data Functions, select "R script" type.
![3-Register-Data-Functions](https://github.com/swayvil/ElasticSearch-with-Spotfire/blob/master/images/3-Register-Data-Functions.png)

Write a R script using elastic and json libraries methods, for example:
```
elastic::connect()
aggs <- '{
	"size": 0,
	"aggs": {
		"distinct_flowNames": {
			"terms": {
				"field": "flowName"
			}
		}
	}
}'
result <- elastic::Search(index="audit-servicecontext", type="serviceContext", body = aggs, raw = TRUE)
json <- jsonlite::fromJSON(result)
flowNameList <- json$aggregations$distinct_flowNames$buckets
```

In this example, the result is stored in flowNameList.

Optionally you can add input parameter:
![4-Register-Data-Functions](https://github.com/swayvil/ElasticSearch-with-Spotfire/blob/master/images/4-Register-Data-Functions.png)

And output parameters:
![5-Register-Data-Functions](https://github.com/swayvil/ElasticSearch-with-Spotfire/blob/master/images/5-Register-Data-Functions.png)

The name of the parameter "flowNameList" is the same name that the one used in the script.
## 4.2. Run the script
Click on the Run button. And add the inputs/outputs. In the example, you need to add a new Data table.
![6-Edit-Parameters](https://github.com/swayvil/ElasticSearch-with-Spotfire/blob/master/images/6-Edit-Parameters.png)

Also, you can select "Refresh function automatically".
### 4.2.1. Audit-rolesCount
```
elastic::connect()
flowNamesVector = c(flowNames)
flowNamesStr = paste0('"', paste(flowNamesVector, collapse = '","'), '"')

aggs <- paste0('{
   "size": 0,
   "query": {
      "terms": {
         "flowName":[', flowNamesStr,']}
   },
   "aggs": {
      "roles_count": {
         "terms": {
            "field": "logEvent.role"
         }
      }
   }
}')

result <- elastic::Search(index="audit-servicecontext", type="serviceContext", body = aggs, raw = TRUE)
json <- jsonlite::fromJSON(result)
roles <- json$aggregations$roles_count$buckets
```

![7-Register-Data-Functions](https://github.com/swayvil/ElasticSearch-with-Spotfire/blob/master/images/7-Register-Data-Functions.png)

![8-Register-Data-Functions](https://github.com/swayvil/ElasticSearch-with-Spotfire/blob/master/images/8-Register-Data-Functions.png)

![9-Register-Data-Functions](https://github.com/swayvil/ElasticSearch-with-Spotfire/blob/master/images/9-Register-Data-Functions.png)

![10-Edit-Parameters](https://github.com/swayvil/ElasticSearch-with-Spotfire/blob/master/images/10-Edit-Parameters.png)

![11-Edit-Parameters](https://github.com/swayvil/ElasticSearch-with-Spotfire/blob/master/images/11-Edit-Parameters.png)

## 4.3. Add Data Functions to your dashboard
Select Edit/Data Function Properties and click on "Insert".
![12-Data-Funtions-Properties](https://github.com/swayvil/ElasticSearch-with-Spotfire/blob/master/images/12-Data-Funtions-Properties.png)

## 4.4. Build a dashboard
Add Text Area
In Data table, select the data table used in output of the data function.
![13-Add-Text](https://github.com/swayvil/ElasticSearch-with-Spotfire/blob/master/images/13-Add-Text.png)

## 4.5. Add pie chart
In Data/Data table select rolesCount
![14-Add-Pie-Chart](https://github.com/swayvil/ElasticSearch-with-Spotfire/blob/master/images/14-Add-Pie-Chart.png)

# 5. Result
![15-Result](https://github.com/swayvil/ElasticSearch-with-Spotfire/blob/master/images/15-Result.png)

# 6. Limitations
It is not possible to use several data functions which update the same table.
