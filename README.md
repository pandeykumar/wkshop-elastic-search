
### Players in the space
1) https://www.elastic.co/  
Original developers of Elasticsearch.
Provides a hosted service with lots of features such as machine learning etc.
But expensive.

2) https://opendistro.github.io/for-elasticsearch/
Recent fork of Elasticsearch by Amazon that is open source with some additional features mostly around security, monitoring and alerting to run in enterprise environment. Aws also provides hosted version **Elasticsearch** based on this version.

### What is Elasticsearch and how is it being used
Open source **distributed**, RESTful search and analytics engine capable of solving a growing number of use cases. Based on proven text search engine Lucene but upgraded to support distributed computing for scale.

### What are some of the use cases ?

#### 1. Log Analytics
    Analysis of infrastructure and application log files. Alerts , dashboards and summary reports etc.
    Improves incident reponse time.
    Provide visibility to the state of the systems
    Efficient search over large volume of data.
    Follow a transaction from end to end
    Centralized access to log with simple functionality like tail.

    #### Examples   
    1. web log -  http error codes and response time analytics and alerting
     ![http status code](https://drive.google.com/uc?id=1i5Ay91MB6kzi4pWVmvrs92vjK_f1RNko)

    2. [Winlogbeat](https://www.elastic.co/guide/en/beats/winlogbeat/current_winlogbeat_overview.html#_winlogbeat_overview)
      - Send Windows event logs to Elasticsearch
    3. [Import IIS logs into elasticsearch](https://blog.sstorie.com/importing-iis-logs-into-elasticsearch-with-logstash/)

#### 2. Business Analytics
    Use Elasticsearch to manage business data and events to provide business functionality

    #### Examples   
    1. Centralized all customer related data from differnt business units into ES and provide report access
    to users via customer portal

    2. Social Media data - source data from various social media accounts for a company and explore using
    Kibana. Get top hashtags, count the number of times a company's handle was in the tweet and posts

    3. Example E-commerce data to get analytics.  For example for Ecomerce orders you can easily create
    analytics to explore sales by manufacturers. No of orders per day , sales by category , total revenue,
    weekly trends, monthly trends etc. Basically replaces a datawarehouse with more like setting alerts etc.

    4. Product catalogs and inventory can be uploaded and queried via dedicated apis for the app.
    We powered a game virtual goods shop with items listed under categories and sub categories.

#### 3. APM ( Application performance monitoring)
    Application performance monitoring. Agents that integrate into the runtime and collects metrics from
    inside the application. For example how many objects created, how many garbage collected, cpu cycles
    on each methods calls etc...
    [APM for ASP.net](https://github.com/elastic/apm-agent-dotnet/blob/master/docs/intro.asciidoc)


#### 4. Enterprise search
    Manage your traditional document search across devisions and integrate with other enterprise data.
    For example search for a KYC document for a customer along with interaction history all in one place.  

#### 5. Metrics
    Operating systems and services metrics. Disk IO, RAM, Cache , Swap etc.
    Apache metrics, Logstash mertric, RabbitMQ metrics. There are log of plugins available.

#### 6. Alerts
    Raise alerts via slack, email etc when certain conditions are met on data in the indexes. Raise alert
    if you see more than 5 htto status code 500 in 5 minute interval.
    Raise alerts if avg response time for /createOrder endpoint is more than 2 sec.
    Usually enterprises pay lot of money to integrate with pager duty etc.
#### 7. Performace Analyzer
    Provided by the Open Distro. Provides detailed visibility into netwrok, disk and OS stats.


## Lets get some hands on experience with Elasticsearch cluster and Kibana
We will launch a 2 node cluster of Elasticsearch and 1 node of Kibana.
Then we will upload some data and play with Kibana.

We will be using [**Docker compose**](https://docs.docker.com/compose/) to coordinate this locally. Eventually on production you can use Kubernetes to manage the cluster.
Docker compose is used to configure and run multi-container setup. In the example above each Elasticsearch node and Kibana instance will run in its own docker container.
Docker compose uses a YAML file to configure the services.\

Lets look at ***open-distro/docker-conmpose.yml*** file in this repo

```
version: '3'
services:
  odfe-node1:
    image: amazon/opendistro-for-elasticsearch:0.9.0
    container_name: odfe-node1
    environment:
      - cluster.name=odfe-cluster
      - bootstrap.memory_lock=true # along with the memlock settings below, disables swapping
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m" # minimum and maximum Java heap size, recommend setting both to 50% of system RAM            
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - odfe-data1:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
      - 9600:9600 # required for Performance Analyzer
    networks:
      - odfe-net
  odfe-node2:
    image: amazon/opendistro-for-elasticsearch:0.9.0
    container_name: odfe-node2
    environment:
      - cluster.name=odfe-cluster
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - discovery.zen.ping.unicast.hosts=odfe-node1
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - odfe-data2:/usr/share/elasticsearch/data
    networks:
      - odfe-net
  kibana:
    image: amazon/opendistro-for-elasticsearch-kibana:0.9.0
    container_name: odfe-kibana
    ports:
      - 5601:5601
    expose:
      - "5601"
    environment:
      ELASTICSEARCH_URL: https://odfe-node1:9200
      ELASTICSEARCH_HOSTS: https://odfe-node1:9200
    networks:
      - odfe-net

volumes:
  odfe-data1:
  odfe-data2:

networks:
  odfe-net:
```
Under Services we have defined 3 nodes. **odfe-node1** and **odfe-node2** are Elasticsearch nodes that uses the docker image provided by Amazon.
**Kibana** is anoher node that uses the Kibana image from Amazon.
Compose will instantiate these 3 nodes and sets them up on a single network **odfe-net**. These nodes are reachable and discoverable within this network.

Lets bring the nodes up

```docker-compose up -d```

If all goes well you should see the three nodes come up
```
Creating odfe-kibana ... done
Creating odfe-node1  ... done
Creating odfe-node2  ... dotne
```

Once its up you can verify the nodes using ``` docker ps -a ````

```
CONTAINER ID        IMAGE                                              COMMAND                  CREATED             STATUS                      PORTS                                                      NAMES
49ed3907b888        amazon/opendistro-for-elasticsearch-kibana:0.9.0   "/usr/local/bin/kiba…"   17 seconds ago      Up 15 seconds               0.0.0.0:5601->5601/tcp                                     odfe-kibana
dd18c8dd034e        amazon/opendistro-for-elasticsearch:0.9.0          "/usr/local/bin/dock…"   17 seconds ago      Up 15 seconds               0.0.0.0:9200->9200/tcp, 0.0.0.0:9600->9600/tcp, 9300/tcp   odfe-node1
572cd43b706a        amazon/opendistro-for-elasticsearch:0.9.0          "/usr/local/bin/dock…"   17 seconds ago      Up 15 seconds               9200/tcp, 9300/tcp, 9600/tcp                               odfe-node2
```

Verify that Elasticsearch is running

```
curl -XGET --insecure https://localhost:9200 -u admin:admin
```
You should see something like this --

```
{
  "name" : "7eP4aI5",
  "cluster_name" : "odfe-cluster",
  "cluster_uuid" : "NJyd3FbZQ7-YYD_ujF9bzA",
  "version" : {
    "number" : "6.7.1",
    "build_flavor" : "oss",
    "build_type" : "tar",
    "build_hash" : "2f32220",
    "build_date" : "2019-04-02T15:59:27.961366Z",
    "build_snapshot" : false,
    "lucene_version" : "7.7.0",
    "minimum_wire_compatibility_version" : "5.6.0",
    "minimum_index_compatibility_version" : "5.0.0"
  },
  "tagline" : "You Know, for Search"
}
```
Note the cluster name **odfe-cluster** it picked up from the YAML file.

List the nodes

```curl -XGET https://localhost:9200/_cat/nodes?v -u admin:admin --insecure```


```
ip         heap.percent ram.percent cpu load_1m load_5m load_15m node.role master name
172.18.0.4           53          81   5    0.96    1.39     0.75 mdi       -      7eP4aI5
172.18.0.3           33          81   5    0.96    1.39     0.75 mdi       *      5nWB3jt
```

Show all the plugins

```curl -XGET https://localhost:9200/_cat/plugins?v -u admin:admin --insecure```

```
name    component                       version
7eP4aI5 opendistro_alerting             0.9.0.0
7eP4aI5 opendistro_performance_analyzer 0.9.0.0
7eP4aI5 opendistro_security             0.9.0.0
7eP4aI5 opendistro_sql                  0.9.0.0
5nWB3jt opendistro_alerting             0.9.0.0
5nWB3jt opendistro_performance_analyzer 0.9.0.0
5nWB3jt opendistro_security             0.9.0.0
5nWB3jt opendistro_sql                  0.9.0.0
```

## Kibana
Kibana is an open-source data visualization and exploration tool used for log and time-series analytics, application monitoring, and operational intelligence use cases

Now lets access Kibana at  http://localhost:5601/
Use ```admin``` and ```admin``` as username and password.

You should see a welcome to Kibana page

![](https://drive.google.com/uc?id=1aZIsnd2DXxYV3EwKw5n9Qk0JrPUuPlCI)

Lets select the link **Explore on my own**

After exploring some of the menu items now select kibana at top left to go to home.

Here selet **Add log data** and then select **Sample Data**

Select **Sample web Logs**
![](https://drive.google.com/uc?id=11s9xF1x5emAcRzAiK5ATJCSkDTh4fnPB)

Now lets go back to Discover and notice entry under  Add a filter that there is an indexes - **kibana_sample_data_logs** with it available fields.

![](https://drive.google.com/uc?id=1GFBJKjxHwvGClzk4OXCCIvSHcNJfJ9WL)

In the query box at the top try out running different queries

> tag:*
> tags:error
> response:400
> response:4*
   Drill down on the chart to narrow the search visually

 Use Flight data to wall through and understand dashboard , visualize and Discover.
[Explore Flight Data sample in Kinaba](https://www.elastic.co/guide/en/kibana/current/getting-started.html)

### Lets create a new index.

Each index is a collection of JSON documents

Here is a sample log data from ApiGateway

```
02:49:12, 127.0.0.1, GET, /, 200
02:49:35, 127.0.0.1, GET, /index.html, 200
03:01:06, 127.0.0.1, GET, /images/sponsered.gif, 304
03:52:36, 127.0.0.1, GET, /search.php, 200
04:17:03, 127.0.0.1, GET, /admin/style.css, 200
05:04:54, 127.0.0.1, GET, /favicon.ico, 404
05:38:07, 127.0.0.1, GET, /js/ads.js, 200
```

The data injesters like logstash converts this into json and posts it to ES

```
{"time":"02:49:12", "ip":"127.0.0.1", "verb":"GET", "path":"/", "response":"200"}
{"time":"02:49:35", "ip":"127.0.0.1", "verb":"GET", "path":"/index.html", "response":"200"}
```
Lets post the first data


```
CURL -v --user admin:admin -H "Content-Type: application/json" -d '{"time":"02:49:12", "ip":"127.0.0.1", "verb":"GET", "path":"/", "response":"200"}' -XPOST https://localhost:9200/my_app_weblog/_doc --insecure
```


Result
```
{
  "_id": "X_nnLmsBlQG17HpdUQUb",
  "_index": "my_app_weblog",
  "_primary_term": 1,
  "_seq_no": 0,
  "_shards": {
    "failed": 0,
    "successful": 1,
    "total": 2
  },
  "_type": "_doc",
  "_version": 1,
  "result": "created"
}
```

Creates a new index called my_app_weblog.

Lets post the second data

```
CURL -v --user admin:admin -H "Content-Type: application/json" -d '{"time":"02:49:35", "ip":"127.0.0.1", "verb":"GET", "path":"/index.html", "response":"200"}' -XPOST https://localhost:9200/my_app_weblog/_doc --insecure
```

Lets view this from Discover

Hit **Management** tab on Main menu

Hit **Create Index patte...** button

In create index pattern UI type your index name **my_app_weblog**

It should search and show a success message

![](https://drive.google.com/uc?id=14bXBh0GtyLrdOPeBXvXAUcKKiwXL_mw4)

Hit Next step and select **Create Index Pattern**

This will then show you the index details like fields , type etc.

Now go to **Discover** and you should see my_app_log index

Select that index and you will see the two rows listed .

### How does applications send logs to ES ?

One popular tool is Logstash - member of ELK stack (Elasticsearch - LogStash - Kibana)

![](https://drive.google.com/uc?id=1XSqtJhFHwD2Lki9uw9gCDbYnzFIwq46E)


In java world log4j is a very popular logging tool that has appender to push data to ES

In dotnet please investigate Serilog , Log4Net etc.


### Cleanup!!
To clean up the docker instance and delete all volumes
```docker-compose down -v```



#### Resources
1. [Sample installation example to play with Open Distro Elastic search in cluster mode with Kibana ](https://devopstar.com/2019/03/13/open-distro-for-elasticsearch-kickstart-guide/)
2. [Similar sample that also compares with Elasticsearch.co] (https://medium.com/@maxy_ermayank/tl-dr-aws-open-distro-elasticsearch-fc642f0e592a)
3. [Explore Kibana](https://www.elastic.co/guide/en/kibana/3.0/using-kibana-for-the-first-time.html)
4. [Kibana Users Guide](https://www.elastic.co/guide/en/kibana/current/index.html)
5. [Explore Flight Data sample in Kinaba](https://www.elastic.co/guide/en/kibana/current/getting-started.html)
6. [ELK .NET Docker](https://www.danclarke.com/elk-docker)
6. [Elasticsearch kibana in dotnet](https://www.humankode.com/asp-net-core/logging-with-elasticsearch-kibana-asp-net-core-and-docker)
