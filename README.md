

This repo demonstrates monitoring a kSQL pipeline using a Chronograf dashboard.
JMX metrics, like kittens, require an element of mustering.

kSQLDB and Chronograf, paired with jmxtrans and Kafka offer a pipeline toolkit to capture, filter and depict JMX metrics at scale. 
JMXTrans sends all JVM metrics to an influxDB database. A Kafka broker has ~150 built in metrics; which can quite easily exceed the data throughput of messages for smaller kafka clusters. So with whiteisting and filtering for nominated JMX metrics types, JMX pipelines can be built to feed a dashboard with a specific, defined role.

One beneficiary of this is topic-level monitoring of streaming pipelines; where message produce metrics at the topic level can be used to determine the health of a stream processing application.
This topology offers other benefits:
* ship all JMX metrics into a kafka topic using jmxtrans wildcards; for all classes, objects and topic names.
* This requires regular jmxtrans container restarts to pick up new topics
* Use the native jmxtrans Kafka producer to stream JSON messages for wildcarded metric definitions, for each JVM server
* Create filter streams using kSQLDB to reduce the message pipelines to logical units: such as metrics for a stream processing applications. Largely using CASE and SUBSTR() predicates
* Use a kSQLDB Sink Connector to stream kSQLDB metric streams to influxDB measurements, and onwards to Chronograf
* See "runme" for the complete pipeline; including "GDE" - a news-story processing streaming application.

![ KSQL Topology ](images/topology.png)


```
It consists of:
docker-compose.yml - docker compose yaml for all containers
             runme - a shell script to initialize all objects, load data and create the streaming pipeline
           getNews - a shell script to pull more test data from data.gdeltproject.org
          loadNews - a shell script to load bundled test data from data/gdelt (102 files; 186,189 news stories)
 config/kafka.json - a jmxtrans config file to extract nominated jmx metrics and push to a kafka topic JMX
      ksqlapp.json - a Chronograf dashboard (not integrated with the current release)
              data - data subdirectories for postgres, influx and Chronograf
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
