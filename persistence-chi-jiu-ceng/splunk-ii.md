---
description: From the Splunk training course
---

# Splunk II

![](<../.gitbook/assets/image (341).png>)

input / parsing / indexing / searching

 index cluster (single instance VS multi instances)

 search head: need CPU power

 virtual env

 time sync (same time zone, different systems)

 splunkd port: **8000** (must be with splunk web) (http, 80: load balance)

 default **8089** (search, communication, REST API, command line)

![](<../.gitbook/assets/image (342).png>)

![](<../.gitbook/assets/image (343).png>)

![](<../.gitbook/assets/image (344).png>)

![](<../.gitbook/assets/image (345).png>)

![universal & heavy forwarded](<../.gitbook/assets/image (346).png>)

Adding Forwardeds and Search Heads --> to expand and growing the system.

 DMC: distributed management console















## **Final Review**

```
Eval command

Tostring function: convert string to another string

Fieldformat command

Multiple Eval command

if(x, y(if true), z(if false))

Eval case function

Stats :double quote required

search command: + conditions
    filter the result and then search

where command:
    isnull / isnotnull
    
 "" , '' --> Splunk will treat as field
 
 fillnull: fill with 0
```

![](<../.gitbook/assets/image (338).png>)

![](<../.gitbook/assets/image (339).png>)

![](<../.gitbook/assets/image (340).png>)

```
Correlating Events:::
    
    maxpause
    maxspan
    startswith
    endwith
    
    transactions VS stats (group & calculation & fast & use this as much as possible)

```

**Knowledge Objects::: (private, specific app, all apps) --> 6 segment keys**

* **naming conventions: group/type/platform/category/time/description**
* **permisssions: admin & power user & user**
* **CIM info(common information model): user, duration, model**

#### Field Extractions

* regex: 
* delimiter: 

**Field Aliases: --> for normalize data**

sourcetype/source/host

#### Tag & Event Type:

 tag value case sensitive

 event type no time range

#### Macro

 for the search string, frequency

 store entire search strings, time range independence, pass arguments

 “fieldformat” & "eval" --> argument validation

 search expansion (ctl + shift + E)

#### Workflow Actions

 GET / POST  / to index Splunk

#### Data Models

 pivot(interface to the data): data set --> structed data set  (use pivot UI rather than command)

 root object / root event / root search / root transaction / child 

**CIM (common information management)**

** **data coming from different fields

 use when: field extractions / aliases / event types / tags

CIM add-on`(Admin)`: only install on `Search Head & Splunk Instance`

****

****

****

****

****

****

**Keywords**: anything, can be with wildcard

**Booleans**: NOT, OR, AND (must be uppercase)

**Fields**: name sensitive, values not 

**Wildcards**: anywhere, beginning of the fields

**Comparisons**: = , <, >, !=, <=

**Common Commands:** Fields, Table, Rename as, Dedup, Sort, Lookup (CSV file)

**Transforming Commands: **top, rare, (default 10) stats

![index, sourcetype, host](<../.gitbook/assets/image (305).png>)

![](<../.gitbook/assets/image (306).png>)

![](<../.gitbook/assets/image (307).png>)

![](<../.gitbook/assets/image (308).png>)

### **Time is the most efficient filter, and then: host, source, sourcetype**

* **inclusion is generally better than exclusion**
* **filter as early as possible**
* **use the appropriate search mode (fas, smart=default, verbose)**

### **Transforming commands:**

** top / rare / chart / timechart / stats / geostats**

#### **1. Smart Mode (default)**

![](<../.gitbook/assets/image (309).png>)

#### **2. Verbose Mode**

![](<../.gitbook/assets/image (311).png>)

#### **3. Fast Mode**

![](<../.gitbook/assets/image (312).png>)

### **Search Job Inspector**

* **header**
* **execution costs: index/filter/rawdata**
* **search job properties: EPS events per second**

###

### Command for Visualization

not all search can be visualizated

**chart types:** line, area, column, bar, bubble, scatter, pie

![](<../.gitbook/assets/image (313).png>)

```
# to handle the null value
-useother=f
-usenull=f
```

![](<../.gitbook/assets/image (314).png>)

![](<../.gitbook/assets/image (316).png>)

![](<../.gitbook/assets/image (317).png>)

![](<../.gitbook/assets/image (318).png>)

![eg1](<../.gitbook/assets/image (319).png>)

![eg2](<../.gitbook/assets/image (320).png>)

### **Filtering and Formatting Data**

```
eval command:
    perform calculations
    convert values
    round values
    format values
    use conditional statements

search
where
fullnull
```

###

### Correlating Events

```
Transaction: 
    is any group of related events that span time
    can come from multiple applications or hosts
    
    -duration
    -eventcount

-filed-list
maxspan
maxpause
startswith
endswith

JessionID

index=web sourcetype=access_combine
| transaction JESSIONID
| search status=404
| highlight JESSIONID, 404

index=web sourcetype=access_combine
status=200 action=purchase
| transaction clientip maxspan=10m
| chart count BY duration span=log2

-stat : will enhance the search time
by default, 1000 events per transaction, but no such limit to stats
admin change change: max_events_per_bucket (limits.conf)


```

![](<../.gitbook/assets/image (321).png>)

###

### Knowledge Objects

![](<../.gitbook/assets/image (322).png>)

```
CIM: common information model

Knowledge Objects:
    tools you use to discover and anylyze various aspects of your data
    
    -data interpretation: fields and field extactions
    -data classification: event types
    -data enrichment
    -normalization: tags and field aliases
    -datasets: data models

    Shareable
    Reuseable
    Searchable

Define the naming conventions:
    group
    object type
    description




```

![](<../.gitbook/assets/image (323).png>)



### Creating and Managing Fields

```
FX: field extractor


Performing Field Extractions:::
    regex / RegEx
    delimiter
    
```

![](<../.gitbook/assets/image (324).png>)

![](<../.gitbook/assets/image (325).png>)



### Create Field Aliases and Calculated Field

![](<../.gitbook/assets/image (326).png>)

![](<../.gitbook/assets/image (327).png>)

###

### Working with Tags and Event Types

```
tag=privileged
tag::user=privileged
tag=p*

Event Types
    
```

![](<../.gitbook/assets/image (328).png>)

### Creating Macro

 settings --> advanced search --> search macros

```
backtick (or grave accent) character
-`macroname`!='macroname'

macro can be validated
```

![](<../.gitbook/assets/image (329).png>)



### Creating and Using Workflow Actions

```
GET / POST / Search workflow action




```

![](<../.gitbook/assets/image (330).png>)

### Creating Data Models

![](<../.gitbook/assets/image (331).png>)

![](<../.gitbook/assets/image (332).png>)

![](<../.gitbook/assets/image (333).png>)

![](<../.gitbook/assets/image (334).png>)

```
Adding Fields:
    Auto-Extracted
    Eval Expression
    Lookup
    Regular expression
    Geo IP

Adding a transaction

Set Permissions

Accelerating a data model
```



### Using the Common Information Model (CIM) add-on

![](<../.gitbook/assets/image (335).png>)

![](<../.gitbook/assets/image (336).png>)

```
Normalized Field:::
    Email Data :
    Network Traffic :
    Web Data :
    
```











****

****

****

****

