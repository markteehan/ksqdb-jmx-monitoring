

This repo demonstrates monitoring a kSQL pipeline using a Chronograf dashboard.
JMX metrics, like kittens, require an element of mustering.

kSQLDB and Chronograf, paired with jmxtrans and Kafka offer a pipeline toolkit to capture, filter and depict JMX metrics at scale. 
JMXTrans sends all JVM metrics to an influxDB database. A Kafka broker has ~150 built in metrics; which can quite easily exceed the data throughput of messages for smaller kafka clusters. So with whiteisting and filtering for nominated JMX metrics types, JMX pipelines can be built to feed a dashboard with a specific, defined role.

One possible use of a filtered JMX dashboard is topic-level monitoring of streaming pipelines; where message produce metrics at the topic level can be used to determine the health of a stream processing application.

This demo uses a public dataset GDELT. The data endpoint publishes a CSV file every fifteen minutes with one CSV line for a news story that was published online, during the last 15 minutes. The file does not contain the news story itsef; rather it contains metadata about the new story: such as the URL, various categorizations (type, country, etc), lat/long of the principal actors, an AI estimation of the tone of the story, among others. In total each line contains a total of 61 attributes for each story. There are typically 3000-10000 stories records every fifteen minutes; millions per day; and billions downloadable as CSVs online.

![ KSQL Topology ](images/topology.png)


```
It consists of:
   docker-compose.yml - docker compose yaml for all containers
                runme - a shell script to initialize all objects, load data and create the streaming pipeline
              getNews - a shell script to pull more test data from data.gdeltproject.org
  config/connect.json - a jmxtrans config file to extract nominated jmx metrics from Kafka Connect
      config/gde.json - a jmxtrans config file to extract topic producer metrics from nominated topics in teh streaming app
    config/kafka.json - a jmxtrans config file to extract nominated jmx metrics from Kafka brokers (not used yet)
 config/countries.sql - an init file used to populate a countries table in postgres
         ksqlapp.json - a Chronograf dashboard (not integrated with the current release)
                 data - persistent data subdirectories for postgres, influx and Chronograf
Instructions:
Clone/download the repo
docker-compose up
Login to Chronograf using the URL below:
* The default username/password is admin/admin
* Create a Data Source for influxdb. URL=influxdb:8086, username=root, password=root, database=influxdb.
* Import dashboard ksqlapp.json. No data is visible yet.

Run the setup scripts
./runme This will execute the steps in sequence with Pauses so that you can observe
It does the following:
* truncate/drop/create the gdelt_event table in postgres
* load 20190712181500.export.csv - 2203 rows into postgres table gdelt_event
* in kSQLDB - create JDBC Source connector sourcepostgres_<TS>
* CREATE SOURCE CONNECTOR source_jdbc_gdelt_event WITH ...
* CREATE SOURCE CONNECTOR source_jdbc_countries WITH ...
* CREATE STREAM GDE_A010_STR WITH (kafka_topic='GDE_000_gdelt_event', value_format='avro',partitions=1);
* CREATE TABLE GDE_B020_TAB AS SELECT cast(count(*) as bigint) as C_COUNT FROM GDE_A010_STR GROUP BY 0;
* CREATE STREAM GDE_A020_STR AS SELECT * FROM GDE_A010_STR PARTITION BY ACTOR1COUNTRYCODE;
* CREATE STREAM GDE_C010_STR AS SELECT * FROM GDE_000_COUNTRIES PARTITION BY ISO3;
* CREATE STREAM GDE_D010_STR AS SELECT EVENTID ...  FROM GDE_A020_STR JOIN GDE_C010_STR WITHIN 60 MINUTES ON (ACTOR1COUNTRYCODE=ISO3);
* CREATE TABLE GDE_D020_TAB  AS SELECT ACTOR1_COUNTRYNAME as CTRY, cast(count(*) as bigint) as C_COUNT ... FROM GDE_D010_STR GROUP BY ACTOR1_COUNTRYNAME;
* CREATE STREAM GDE_D030_STR AS SELECT replace(replace(replace(replace(' {"schema":{"type":"struct","fields":[{" ...  as influx_json_row FROM GDE_D020_TAB
* CREATE SINK CONNECTOR sinkinflux_gde_${DT2} ...

URLs:
Confluent Control Center: http://localhost:9021
                 Chronograf: http://localhost:8888
```      
			
 

![ Chronograf Dashboard for a kSQL App](images/screenshot.png)
