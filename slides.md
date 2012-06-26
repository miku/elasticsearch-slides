# elasticsearch-slides

!SLIDE

# 5 minutes intro to Elasticsearch

}}} images/tree.jpg

!SLIDE left

# Elasticsearch <span style="color:gray">is a</span>

* multitenant,
* real-time,
* distributed

index engine based on **Lucene**, that

* adapts to your domain model (not the other way around)
* is easy to setup and
* is fully configurable through an HTTP API

!SLIDE left

# Elasticsearch

* <span style="color:gray">was started</span> by the author of Compass, Shay Banon
* <span style="color:gray">is written</span> in Java
* <span style="color:gray">is actively developed</span> @[github.com/elasticsearch/elasticsearch](https://github.com/elasticsearch/elasticsearch)
    * 0.5 (2 years ago)
    * 0.18 (8 month ago)
    * 0.19 (4 month ago)
    * 0.19.4 (1 month ago)
    * 0.19.5 (12h ago)

!SLIDE left

# Multitenancy

* have as many indexes as you want
* graylog uses one index per day
* finc uses one index per data source

``` sh
$ curl -XPUT    localhost:9200/sample_index # create index
$ curl -XPUT    localhost:9200/throwaway    # create another index
$ curl -XDELETE localhost:9200/throwaway    # destroy index
```

* search over one or more indexes

``` sh
$ curl -XGET    localhost:9200/idx1,idx2/_search?q=Leipzig # search for Leipzig in idx1 and idx2
```
!SLIDE left

# Real-time / `refresh_interval` at 1s by default

}}} images/rt.jpg

!SLIDE left

# Distributed

* start with one node, add more on the go
* control the number of replicas

``` sh
$ curl -XPUT localhost:9200/sample_index/_settings -d '{ "index" : { "number_of_replicas" : 2 }}'
```

* automatic load balancing (index, search)

!SLIDE left

# Adapts to your domain

}}} images/domain.png

!SLIDE left

# Adapts to your domain

* JSON documents
* dynamic mapping (without overhead)
* mapping templates

!SLIDE left

# Easy to setup

* you'll need a Java Virtual Machine
* then

``` shell
$ wget https://github.com/downloads/elasticsearch/elasticsearch/elasticsearch-0.19.4.zip
$ unzip elasticsearch-0.19.4.zip
$ cd elasticsearch-0.19.4
$ bin/elasticsearch -f
[16:25:21,766][INFO ][node      ] [Dracula] {0.19.4}[4028]: initializing ...
[16:25:21,775][INFO ][plugins   ] [Dracula] loaded [], sites []
[16:25:24,015][INFO ][node      ] [Dracula] {0.19.4}[4028]: initialized
[16:25:24,015][INFO ][node      ] [Dracula] {0.19.4}[4028]: starting ...   
...
[16:25:27,324][INFO ][gateway   ] [Dracula] recovered [3] indices into cluster_state
```

!SLIDE left

# Easy to setup

* index a doc

``` shell
$ curl -XPUT 'http://localhost:9200/twitter/tweet/1' -d '{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elastic Search"
}'
```

* search a doc

``` shell
$ curl -XGET 'http://localhost:9200/twitter/tweet/_search?q=user:kimchy'
```


!SLIDE left

# Configuration via HTTP API

* prominent settings:
    * `index.number_of_replicas`
    * `index.refresh_interval`
    * `index.blocks.read_only`
    * `index.merge.policy.merge_factor`

* and a lot more (ES + exposed Lucene settings)

!SLIDE left

# Tradeoffs

* no autowarming (see: [https://github.com/elasticsearch/elasticsearch/issues/1006](https://github.com/elasticsearch/elasticsearch/issues/1006))
* while active, the community is much smaller than SOLR's
* versioned documents would be nice, but it's not built-in (as we discovered)

!SLIDE left

# ES in a library context

* Linking Open Bibliographic Data [http://lobid.org](http://lobid.org)
    * see also: [http://www.dipf.de/de/bildungsinformation/pdf/pohl-linked-open-data-services-im-hbz-fis-tagung-2012](http://www.dipf.de/de/bildungsinformation/pdf/pohl-linked-open-data-services-im-hbz-fis-tagung-2012)
* finc
    * raw data storage

!SLIDE left

# What others say

* Solr may be the weapon of choice when building standard search applications, but Elasticsearch takes it to the next level with an architecture for creating modern realtime search applications. 

* Percolation is an exciting and innovative feature that singlehandedly blows Solr right out of the water. Elasticsearch is scalable, speedy and a dream to integrate with. (Ryan Sonnek, socialcast)

* more: [http://stackoverflow.com/questions/10213009/solr-vs-elasticsearch](http://stackoverflow.com/questions/10213009/solr-vs-elasticsearch), [http://www.quora.com/What-are-the-main-differences-between-ElasticSearch-Apache-Solr-and-SolrCloud](http://www.quora.com/What-are-the-main-differences-between-ElasticSearch-Apache-Solr-and-SolrCloud)

!SLIDE left

# Percolation?

}}} images/percolation.jpg

!SLIDE left

# Percolation

* Think of it as the reverse operation of indexing and then searching 
* Instead of sending docs, indexing them, and then running queries
* One sends queries, registers them, and then sends docs and finds out which queries match that doc


!SLIDE left

# Wrap up

* more features not covered:
    * search / query language
    * facets
    * scrolls

* language bindings:
    * Elasticsearch.pm, PHP, Ruby, Python, Java (native), .NET, Clojure, Erlang


!SLIDE left

# Credits

* http://www.flickr.com/photos/catdancing/4652800293/
* http://www.flickr.com/photos/andreaskoller/154706613/
* http://www.flickr.com/photos/ddebold/3738709697/

