---
description: >-
  https://docs.splunk.com/Documentation/Splunk/8.1.4/Translated/SimplifiedChinesemanuals
---

# Splunk

![](<../.gitbook/assets/image (239).png>)

![](<../.gitbook/assets/image (241).png>)

**`  (APM:  `**`application performance monitoring`**`)`**

**`(RUM: `**`Real User Monitoring`**`)`**

Start from Splunk RUM (frontend traces) to Splunk APM (backend traces). 

**Data Visualization**\
**Problem Detection**\
**Data Exploration and Correlation**

collect metrics, logs, and spans

**Data Setup**

**Splunk Observability Cloud**

![](<../.gitbook/assets/image (242).png>)





* **Configuring Integrations **
* **Data Sources**\
  \
  Install the OpenTelemetry Connector first and then configure the agent to include the appropriate “receivers”.

![](<../.gitbook/assets/image (243).png>)



* Data point
* Metric (**Metric Names and Values**)

**Types of Metrics:**

1. Gauges, 
2. Counters, and 
3. Cumulative Counters

![](<../.gitbook/assets/image (244).png>)

###

### MetaData:

1. Dimensions
2. Properties
3. Tags

**Dimensions::: **A dimension is a key:value pair that provides additional information about the metric source. 

**Metric Time Series::: MTS**

**Properties::: kvp**

**Tags:::**Tags are a type of metadata used to define binary states.

**Filtering:::**

**Grouping:::**



## Dashboard Group:

1. From the create menu
2. When saving a dashboard
3. Importing a dashboard group

* Filter: default + second
* time range: specific, custom a time range

Share a dashboard  \[save as]

#### **Dashboard Variables**

** actions --> variables --> details --> variables display**

![](<../.gitbook/assets/image (245).png>)

**Create / Clone / Share --> dashboard**

****

## **Chart**

  create a chart from the dashboard

1. **Chart:** made up of one or more **plots**. 
2. **Plot**: is defined by the **signal, **composed of Metric Time Series 
3. **Signal: **the metric that you wish to plot
4. **Filter:** You can filter a plot on metadata
5. **Analytic Functions:** Add analytic functions to raw data to gain insights

![](<../.gitbook/assets/image (246).png>)



### **Other Ways to Create Charts**

* Using the REST API to create these objects(**Signalflow**)
* Using the Splunk Terraform provider to create these objects





## Creating a Detector

![](<../.gitbook/assets/image (249).png>)

* A detector is the object you create to detect issues. 
* A detector is made up of one or more rules. 
* Rules define the condition(s) for the alert.
* Each rule specifies the alert severity, notification recipients, and an alert message. 
*   When the conditions of the rule are satisfied, the alert is triggered. 

    * If you have defined recipients, notifications are sent.



![](<../.gitbook/assets/image (247).png>)

**Subscribing to alerts**\


![](<../.gitbook/assets/image (248).png>)

**Note: **if you have Splunk APM enabled in your Splunk Observability Cloud org, you will first have to select the type of detector.

### **Adding Rules to a Detector**

1. Open the detector and click New Rule. 
2. Select the signal to monitor. 
3. Select the alert condition, enter the information in the remaining tabs and activate the rule.

You can add multiple rules to a detector. 

![](<../.gitbook/assets/image (250).png>)

![How does splunk works???](<../.gitbook/assets/image (251).png>)

1. Splunk Enterprise
2. Splunk Cloud
3. Splunk Light

### Splunk App::: [https://splunkbase.splunk.com/](https://splunkbase.splunk.com)

![](<../.gitbook/assets/image (252).png>)

### Splunk Roles::: Admin / Power / User



## Processing Components:

![](<../.gitbook/assets/image (253).png>)

![](<../.gitbook/assets/image (254).png>)



![](<../.gitbook/assets/image (256).png>)

![](<../.gitbook/assets/image (257).png>)

![](<../.gitbook/assets/image (258).png>)

![](<../.gitbook/assets/image (259).png>)

![](<../.gitbook/assets/image (260).png>)

![](<../.gitbook/assets/image (261).png>)

![](<../.gitbook/assets/image (262).png>)

every search is also a job



## Fields Search (KVP)

![](<../.gitbook/assets/image (263).png>)

* host
* source
* sourceType

![](<../.gitbook/assets/image (264).png>)

![](<../.gitbook/assets/image (265).png>)

#### Search Mode: Fast / Smart / Verbose

![](<../.gitbook/assets/image (266).png>)

![](<../.gitbook/assets/image (267).png>)

### Use Index:  can use multiple indices at the beginning

![](<../.gitbook/assets/image (268).png>)

![](<../.gitbook/assets/image (269).png>)



## Search Language

![](<../.gitbook/assets/image (270).png>)

1. search terms
2. commands
3. functions
4. arguments
5. clauses

![separate by the pipe](<../.gitbook/assets/image (271).png>)

### Create a Table

![](<../.gitbook/assets/image (272).png>)

![](<../.gitbook/assets/image (273).png>)

![](<../.gitbook/assets/image (274).png>)

![](<../.gitbook/assets/image (275).png>)

![](<../.gitbook/assets/image (276).png>)



## Transforming Command

![](<../.gitbook/assets/image (278).png>)

![getting top values](<../.gitbook/assets/image (277).png>)

![](<../.gitbook/assets/image (279).png>)

![](<../.gitbook/assets/image (280).png>)

![](<../.gitbook/assets/image (281).png>)

## Creating Reports

![](<../.gitbook/assets/image (282).png>)

![](<../.gitbook/assets/image (283).png>)

## Pivit & DataSet

## Creating and Using Lookup (data not indexed)

![](<../.gitbook/assets/image (284).png>)

![](<../.gitbook/assets/image (285).png>)



## Creating Alert and Report

![](<../.gitbook/assets/image (286).png>)

### Actions:::

![](<../.gitbook/assets/image (287).png>)

![](<../.gitbook/assets/image (288).png>)

![](<../.gitbook/assets/image (289).png>)

![](<../.gitbook/assets/image (290).png>)



3 Elements: indexer, search header, forwarders

sorted data by **age** -->directory

Search Head: handle search request --> distribute to indexers

Forwarder: require minimal resource

Deployment:::

Single instance: input / parsing / indexing / searching  

 searcher and indexer can be cluster

```
cd /Applications/splunk/bin
./splunk start
./splunk stop
./splunk restart
./splunk help
```

Splunk Cloud (default 5 GB = 30days)

**Roles**: Admin/ Power/ User

```
On Boarding New Data:::
Upload / Monitor / Forward(from forwarder to indexer: main data source)

Web data index
main index
security index

Splunk uses SOURCE TYPEs to categorize the data being indexed.

UI::: Splunk bar / app bar/ search bar

limit search by time--> key factor to save searching time

Save as --> knowledge object

Transforming commands: create statistics and visalizations

by default, search job remain active for 10 minutes
shared search job remain active for 7 days

Export: Raw/ CSV/ XML/ JSON
```

Field **Name**: case sensitive, Field **Value**: case not sensitive

 Additional fields: at least 20% of the data value

a dest 4: it's a string and containing 4 values

Using Time: most effieict

Apply Filter as eariler as possible

Time abbreviations tell Splunk what time to search

```
soucetype=access_combined earliest=01/08/2021:12:00:00

search by time(advanced): @

Use Indexes:::
Web Data & Security Data ==> can limit the access for different users
index=ma*
```

Search Language:::

* search terms （sourcetype=acc\*）
* commands
* functions (sum(xxx))
* arguments
* clauses

```
sourcetype=acc* status=200 | stats list(product_name) as "Games Haha" | top "HAHA"

```

 Pipe: 

Visual tools 

![](<../.gitbook/assets/image (291).png>)

```
Field command extraction (most costly parts) <-- need to limit this 
Table command read table
Rename command rename the table (rename ... as ...)
Dedup command remove the dup part
Sort command generate the report by order
(String-> alphanumerically)
(Numeric-> numerically)

```

![](<../.gitbook/assets/image (293).png>)

```
Transforming::: transform based on the search result 
Top commands: default by 10 showing
Rare commands: defualt showing 10
Stats commands： count, distinct count, sum, avg, min, max, list, values
List commands:
Value functions：
```

![](<../.gitbook/assets/image (295).png>)

![](<../.gitbook/assets/image (296).png>)

![](<../.gitbook/assets/image (297).png>)

![](<../.gitbook/assets/image (298).png>)

![](<../.gitbook/assets/image (299).png>)

![](<../.gitbook/assets/image (300).png>)

![](<../.gitbook/assets/image (301).png>)

![](<../.gitbook/assets/image (302).png>)

Save AS --> report (title: convention)

![](<../.gitbook/assets/image (303).png>)

chart： by number, location ... (Visualizations)

Save as--> dashboard panel

```
Using Pivot
admin + power only

report valid for the last 7 days

Data Model
    Web Requests
        Successful Response

Piovt default time : all time

instant pivot tool

DataSet addon


```

![](<../.gitbook/assets/image (304).png>)

Lookups:::  --> dataSet

 CVS Files, Script, Geospatial Data

```
Two steps to set up lookups:
1. define a lookup table
2. define the lookup

case sensitive

eg: http status code mapping

fixed lookup + automatically lookup

| inputlookup http_status.csv (first row defaultly field name) 
```

### Scheduled Report::: 

```
schedule options: run every week/day --> add detailed time
time range
Schedule priority> only admin users

include a schedule window only if the report doesn't have to start at a specific time

alert actions --> also can build your own


```

index: process search request

search head: send search result

 Splunk use sourceType for indexing

 data model make by **dataset**

###  Splunk Search Terms: Keywords, Booleans, Phrases, Fields, Wildcards, Comparisons.





####













