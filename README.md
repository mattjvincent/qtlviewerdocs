
# QTL Viewer


This is a web application that renders QTL "type" data.  There are multiple components and moving parts, but 2 different main projects make up the QTL Viewer: **QTL API** and **QTL Web**.


## QTL API

### Overview
[Docker](http://www.docker.com/) and [Docker Compose](http://docs.docker.com/compose/) are the recommended way of running the QTL API so it can be easily managed.  A [Docker](http://www.docker.com/) image for the QTL API can be found on [Docker Hub]( https://hub.docker.com/) at [mattjvincent/rqtlapi](https://hub.docker.com/r/mattjvincent/rqtlapi/).

This image contains all the necessary code to run the QTL API. An example [docker-compose.yml](https://docs.docker.com/compose/compose-file/) can be found in the [churchill-lab/qtlapi](http://github.com/churchill-lab/qtlapi) [GitHub](http://github.com/) repository.

An example to start this [Docker](http://www.docker.com/)  image is:

    docker run --rm -p 8000:8000 --network netapi \
               -v /path/to/my.RData:/app/qtlapi/data/data.RData \
               -v /path/to/ccfoundersnps.sqlite:/app/qtlapi/data/ccfounders.sqlite \
               -v /path/to/qtlapi.R:/app/qtlapi/qtlapi.R \
               mattjvincent/qtlapi /app/qtlapi/qtlapi.R

For more in depth documentation, please refer to [churchill-lab/qtlapi](http://github.com/churchill-lab/qtlapi).

----------

### Tech Stack

* [Docker](http://www.docker.com/) - Build, Ship, and Run Any App, Anywhere 
* [R](http://www.r-project.org/) - number cruncher
* [R/qtl2](http://kbroman.org/qtl2/) - Analysis 
* [Plumber](http://www.rplumber.io/) - Help turn R code into Web based API


----------


### Data
Behind the QTL API is a **RData** file which MUST be formatted according to the following definition: [QTL API Data File Definition](https://github.com/churchill-lab/qtlapi/blob/master/docs/QTLAPIDataStructures.md)

----------

### Endpoints

The API generated is used by the QTL Web app.  The following API's are supplied:

#### /datasets

**Description**: This is the initial API called by QTL Web.  The data returned is dependent upon if the **dataType** is `mRNA`, `protein`, or `phenotype`. The purpose of this API is to define and describe the datasets used in the viewer.

**Parameters**: None

**Response**: The response will be JSON.

**Example response:**
```
{"dataSets": [{
    "id": "some.dataset.identifier",
    "displayName": "Some Dataset",
    "dataType": "mRNA",
    "gene_ids": [
         "ENSMUSG00000000001",
         "ENSMUSG00000000028",
         "ENSMUSG00010203821",
         ...
    ],
    "protein_ids": [],
    "phenotypes" : [],
    "covarFactors":[{
        "column.name":"sex",
        "display.name":"Sex"
    }, {
        "column.name":"gen",
        "display.name":"Generation"}]
    },
    ...
    ],
    "ensemblVersion": 75,
    "numMarkers": 68433    
}
```

----------


#### /lodscan

**Description**: Perform a LOD Scan.

**Parameters**:

| Parameter | Description |
| ------ | ------ |
| dataset | dataset identifier |
| id | the identifer to perform LOD scan on |
| regressLocal | TRUE to regress on local genotype, FALSE to not |
| nCores | number of cores to use |
| expand | TRUE to expand the JSON, FALSE to condense |

**Response**: The response will be JSON.

**Example Response**:
```
{"result": [
    {"id":"1_4530778","chr":"1","pos":4.5308,"lod":1.688},
    {"id":"1_4533435","chr":"1","pos":4.5334,"lod":1.6709},
    ...
    {"id":"1_4536092","chr":"1","pos":4.5361,"lod":1.6539}
],
 "time":5.3
}
```

----------

#### /foundercoefs

**Description**: Get founder coefficients.

**Parameters**:

| Parameter | Description |
| ------ | ------ |
| dataset | dataset identifier |
| id | the identifer to perform LOD scan on |
| chrom | the chromosome |
| regressLocal | TRUE to regress on local genotype, FALSE to not |
| blup | TRUE to perform Best Linear Unbiased Predictors  |
| center | TRUE to center around 0 |
| nCores | number of cores to use |
| expand | TRUE to expand the JSON, FALSE to condense |

**Response**: The response will be JSON.

**Example Response**:

```
{"result":[{"id":"2_4292516","chr":"2","pos":4.2925,
            "A":0.012,"B":-0.0267,"C":0.0345,"D":0.1994,
            "E":0.1305,"F":-0.4769,"G":0.1217,"H":0.0055},
           {"id":"2_4337139","chr":"2","pos":4.3371,
            "A":0.0114,"B":-0.0267,"C":0.0343,"D":0.2,
            "E":0.1307,"F":-0.4769,"G":0.1218,"H":0.0055},
            ...
           {"id":"2_4369263","chr":"2","pos":4.3693,
            "A":0.012,"B":-0.0259,"C":0.0407,"D":0.2006,
            "E":0.1323,"F":-0.4783,"G":0.1134,"H":0.0052}
        ],
 "time":2.3
}
```

----------

#### /expression

**Description**: Get the expression data.

**Parameters**:

| Parameter | Description |
| ------ | ------ |
| dataset | dataset identifier |
| id | the identifer to perform LOD scan on |

**Response**: The response will be JSON.

**Example Response**:

```
{"data":[{"mouse_id":"F01","Sex":"F","Generation":"G4","Litter":2,
          "Diet":"hf","Coat.Color":"agouti","GenerationLitter":"G4_2",
          "expression":0.2243,"_row":"F01"},
          ...
         {"mouse_id":"F02","Sex":"F","Generation":"G4","Litter":2,
          "Diet":"hf","Coat.Color":"black","GenerationLitter":"G4_2",
          "expression":0.3498,"_row":"F02"}
         ],
 "time":2.3
}
```
----------

#### /mediate

**Description**: Perform mediation analysis.

**Parameters**:

| Parameter | Description |
| ------ | ------ |
| dataset | dataset identifier |
| id | the identifer to perform LOD scan on |
| mid | marker identifier |
| expand | TRUE to expand the JSON, FALSE to condense |

**Response**: The response will be JSON.

**Example Response**:
```
{"result":[{"gene_id":"ENSMUSG00000000001","symbol":"Gnai3",
            "chr":"3","pos":108.1267,"LOD":1.3625},
           {"gene_id":"ENSMUSG00000000028","symbol":"Cdc45",
            "chr":"16","pos":18.7962,"LOD":1.605},
           ...
           {"gene_id":"ENSMUSG00000000037","symbol":"Scml2",
            "chr":"X","pos":161.1877,"LOD":1.2535}
     ],
 "time":10.3
}
```
----------

#### /snpassoc

**Description**: SNP Association Mappting

**Parameters**:

| Parameter | Description |
| ------ | ------ |
| dataset | dataset identifier |
| id | the identifer to perform LOD scan on |
| chrom | the chromosome |
| location | chromosomal location |
| windowSize | the window size, distance on each size of location |
| nCores | number of cores to use |
| expand | TRUE to expand the JSON, FALSE to condense |

**Response**: The response will be JSON.

**Example Response**:


```
{"result":[{"snp":"rs245710663","chr":"1","pos":14.5001,
            "alleles":"G|C","sdp":64,"ensembl_gene":"",
            "csq":"intergenic_variant","index":1,"interval":147,
            "on_map":false,"lod":0.0015},
           {"snp":"rs263649601","chr":"1","pos":14.5002,
            "alleles":"C|T","sdp":64,"ensembl_gene":"",
            "csq":"intergenic_variant","index":1,"interval":147,
            "on_map":false,"lod":0.0015},
           ...
           {"snp":"rs32279922","chr":"1","pos":14.5004,
            "alleles":"C|T","sdp":153,"ensembl_gene":"",
            "csq":"intergenic_variant","index":3,"interval":147,
            "on_map":false,"lod":3.5395e-06}
         ],
 "time":11.1
}
```


----------


#### /lodpeaks

**Description**: Peak LOD scors for a dataset.

**Parameters**:

| Parameter | Description |
| ------ | ------ |
| dataset | dataset identifier |
| expand | TRUE to expand the JSON, FALSE to condense |

**Response**: The response will be JSON.

**Example Response**:

```
 {"result":[
     {"marker":"1_100007442","chr":"1","pos":100.0074,
      "gene_id":"ENSMUSG00000028028","symbol":"Alpk1",
      "gene_chrom":"3","middle":127.7255,"lod":6.1147
     },
     {"marker":"1_100078094","chr":"1","pos":100.0781,
      "gene_id":"ENSMUSG00000024727","symbol":"Trpm6",
      "gene_chrom":"19","middle":18.8212,"lod":6.048
     },
     ...
     {"marker":"1_100148722","chr":"1","pos":100.1487,
      "gene_id":"ENSMUSG00000045725","symbol":"Prr15",
      "gene_chrom":"6","middle":54.3286,"lod":6.5079
     }],
  "time":3.9
}
```
----------

## QTL Web

### Overview
[Docker](http://www.docker.com/) and [Docker Compose](http://docs.docker.com/compose/) are the recommended way of running the QTL Web so it can be easily managed.  A [Docker](http://www.docker.com/) image for the QTL API can be found on [Docker Hub]( https://hub.docker.com/) at [mattjvincent/rqtlweb](https://hub.docker.com/r/mattjvincent/rqtlweb/).

This image contains all the necessary code to run the QTL Web. An example [docker-compose.yml](https://docs.docker.com/compose/compose-file/) can be found in the [churchill-lab/qtlapi](http://github.com/churchill-lab/qtlweb) [GitHub](http://github.com/) repository.

An example to start this [Docker](http://www.docker.com/)  image is:

    docker-compose up --scale api=3

For more in depth documentation, please refer to [churchill-lab/qtlweb](http://github.com/churchill-lab/qtlweb).


----------


### Tech Stack

* [Docker](http://www.docker.com/) - Build, Ship, and Run Any App, Anywhere 
* [Python](http://www.python.org/) - Web framework
* [Celery](http://www.celeryproject.org/) - Distributed Task Queue
* [Redis](http://redis.io/) - Message Broker
* [Bootstrap 4](http://getbootstrap.com/) - HTML enhanced for web apps!



