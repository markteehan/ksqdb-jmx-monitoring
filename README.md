

This repo demonstrates monitoring a kSQL pipeline using a Grafana dashboard.
JMX metrics, like kittens, require an element of mustering.

kSQLDB and Grafana, paired with jmxtrans and Kafka offer a pipeline toolkit to capture, filter and depict JMX metrics at scale. 
JMXTrans sends all JVM metrics to a kafka topic where kSQLDB filters, transforms and sinks whats needed to influxdb.

One beneficiary of this is topic-level monitoring of streaming pipelines; where message produce metrics at the topic level can be used to determine the health of a stream processing application.
This topology offers other benefits:
* ship all JMX metrics into a kafka topic using jmxtrans wildcards; for all classes, objects and topic names.
* This requires regular jmxtrans container restarts to pick up new topics
* Use the native jmxtrans Kafka producer to stream JSON messages for wildcarded metric definitions, for each JVM server
* Create filter streams using kSQLDB to reduce the message pipelines to logical units: such as metrics for a stream processing applications. Largely using CASE and SUBSTR() predicates
* Use a kSQLDB Sink Connector to stream kSQLDB metric streams to influxDB measurements, and onwards to Grafana
* See "runme" for the complete pipeline; including "GDE" - a news-story processing streaming application.

```
It consists of:
docker-compose.yml - docker compose yaml for all containers
             runme - a shell script to initialize all objects, load data and create the streaming pipeline
           getNews - a shell script to pull more test data from data.gdeltproject.org
          loadNews - a shell script to load bundled test data from data/gdelt (102 files; 186,189 news stories)
 config/kafka.json - a jmxtrans config file to extract nominated jmx metrics and push to a kafka topic JMX
      ksqlapp.json - a grafana dashboard (not integrated with the current release)
              data - data subdirectories for postgres, influx and grafana
Instructions:
Clone/download the repo
docker-compose up
Login to Grafana using the URL below:
* The default username/password is admin/admin
* Create a Data Source for influxdb. URL=influxdb:8086, username=root, password=root, database=influxdb.
* Import dashboard ksqlapp.json. No data is visible yet.

Run the setup scripts
./runme This will execute the steps in sequence with Pauses so that you can observe
It does the following:
*  truncate/drop/create the gdelt_event table in postgres
* load 20190712181500.export.csv - 2203 rows into postgres table gdelt_event
* in kSQLDB - create JDBC Source connector sourcepostgres_<TS>
* CREATE STREAM GDE_010_A_STR_V01 WITH (kafka_topic='GDE_000_A_<ts>-gdelt_event', value_format='avro',partitions=1);
* CREATE TABLE  GDE_020_B_TAB_V01 AS SELECT ACTOR1COUNTRYCODE as CTRY, cast(count(*) as bigint) as C_COUNT ....
* CREATE STREAM GDE_030_C_RKY_V01 WITH (KAFKA_TOPIC='GDE_020_B_TAB_V01', VALUE_FORMAT='AVRO');
* CREATE STREAM GDE_040_C_RKY_V01 AS SELECT CTRY,C_COUNT,C_AVGTONE,C_MAXTONE,C_MINTONE,LAST_EVENTID FROM GDE_030_C_RKY_V01;
* CREATE STREAM GDE_050_C_RKY_V01 AS SELECT * FROM GDE_040_C_RKY_V01 PARTITION BY LAST_EVENTID;
* CREATE STREAM GDE_060_D_STR_V01 AS SELECT EVENTID as EVENTID, MONTHYEAR , YEAR , FRACTIONDATE , ACTOR1CODE , ACTOR1NAME ...
URLs:
Confluent Control Center: http://localhost:9021
                 Chronograf: http://localhost:8888
```      
			
 

![ Grafana Dashboard for a kSQL App](images/screenshot.png)
