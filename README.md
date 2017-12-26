# ElasticSearch_Notes
Notes on my experience with ElasticSearch. Serve as a reminder just in case I forgot something (especially some syntax).

#### Table of Contents

[1. Install MongoDB 6.x on Ubuntu 16 LTS](#tip1)  
[2. Import CSV Data with LogStash](#tip2)  
[3. Some common commands](#tip3)  

<a name="tip1"></a>
## 1. Install MongoDB 6.x on Ubuntu 16 LTS

```bash
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
sudo apt-get install apt-transport-https
echo "deb https://artifacts.elastic.co/packages/6.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-6.x.list
sudo apt-get update
sudo apt-get install elasticsearch
```

Start ElasticSearch when booting:

```bash
sudo /bin/systemctl daemon-reload
sudo /bin/systemctl enable elasticsearch.service
```

Elasticsearch can be started and stopped as follows:

```bash
sudo systemctl start elasticsearch.service
sudo systemctl stop elasticsearch.service
```

Use browser (or curl) to check the status:

![ElasticSearch About](./Images/ElasticSearch_About.png)

See [1] for original posting.

<a name="tip2"></a>
## 2. Import CSV Data with LogStash

Sample CSV:

```bash
33,sweet,"kinda oval"
17,regular,bumpy
91,regular,"perfectly round"
18,sweet,delightful
42,fried,crispy
37,"extra special",crispy
```

Sample logstash-simple.conf:

```bash
input {
  file {
    path => "/home/test/Desktop/simple.csv"
    start_position => "beginning"
  }
}
filter {
  csv {
      columns => ["potato_id", "potato_type", "description"]
      separator => ","
  }
}
output {
   elasticsearch {
     hosts => "127.0.0.1:9200"
     index => "allfruits"
  }
stdout {}
}
```

Import data:

```bash
logstash -f logstash-simple.conf
```

<a name="tip3"></a>
## 3. Some common commands

* Get all indexes

```bash
curl -X GET "http://localhost:9200/_cat/indices?v"
```

Result:

```bash
health status index     uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   allfruits Jy0851AXRxS1IeHo3erwNw   5   1          6            0     24.1kb         24.1kb
```

* Delete an index

```bash
curl -X DELETE http://localhost:9200/allfruits
```

Result:

```bash
{"acknowledged":true}
```

or

```bash
{"error":{"root_cause":[{"type":"index_not_found_exception","reason":"no such index","resource.type":"index_or_alias","resource.id":"allfruits","index_uuid":"_na_","index":"allfruits"}],"type":"index_not_found_exception","reason":"no such index","resource.type":"index_or_alias","resource.id":"allfruits","index_uuid":"_na_","index":"allfruits"},"status":404}
```

* Get all documents of an index

```bash
http://localhost:9200/allfruits/_search?pretty=true&q=*:*
```

```bash
{
  "took" : 11,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 6,
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "allfruits",
        "_type" : "doc",
        "_id" : "U24okGABwbx53hmVoKPE",
        "_score" : 1.0,
        "_source" : {
          "host" : "test-VirtualBox",
          "@version" : "1",
          "message" : "33,sweet,\"kinda oval\"",
          "potato_id" : "33",
          "description" : "kinda oval",
          "potato_type" : "sweet",
          "path" : "/home/test/Desktop/simple.csv",
          "@timestamp" : "2017-12-26T00:10:57.036Z"
        }
      }
      ...
```

* Configure how many documents to return with indentation

```bash
http://localhost:9200/allfruits/_search?pretty=true&size=100
```

* Partially match any of the fields

```bash
curl -X POST -H "Accept: application/json" -H "Content-Type: application/json" -d '{"query":{"query_string":{"fields":["potato_id","description","potato_type"],"query":"*nd*"}}}'  http://localhost:9200/allfruits/_search?pretty
```

Result:

```bash
{
  "took" : 21,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 2,
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "allfruits",
        "_type" : "doc",
        "_id" : "U24okGABwbx53hmVoKPE",
        "_score" : 1.0,
        "_source" : {
          "host" : "test-VirtualBox",
          "@version" : "1",
          "message" : "33,sweet,\"kinda oval\"",
          "potato_id" : "33",
          "description" : "kinda oval",
          "potato_type" : "sweet",
          "path" : "/home/test/Desktop/simple.csv",
          "@timestamp" : "2017-12-26T00:10:57.036Z"
        }
      },
      {
        "_index" : "allfruits",
        "_type" : "doc",
        "_id" : "VW4okGABwbx53hmVo6Os",
        "_score" : 1.0,
        "_source" : {
          "host" : "test-VirtualBox",
          "@version" : "1",
          "message" : "91,regular,\"perfectly round\"",
          "potato_id" : "91",
          "description" : "perfectly round",
          "potato_type" : "regular",
          "path" : "/home/test/Desktop/simple.csv",
          "@timestamp" : "2017-12-26T00:10:57.098Z"
        }
      }
    ]
  }
}
```

In the query, we can use:

```bash
"query":"*nd* AND *ee*" # return 1 hit
"query":"*nd* OR *ful*" # return 3 hits
```

# References

[1] https://www.elastic.co/guide/en/elasticsearch/reference/current/deb.html
