# ksql-grafana-monitoring

This repo demonstrates monitoring a kSQL pipeline using a Grafana dashboard.
```
It consists of:
docker-compose.yml - docker compose yaml for all containers
             runme - a shell script to initialize all objects, load data and create the streaming pipeline
           getNews - a shell script to pull more test data from data.gdeltproject.org
          loadNews - a shell script to load bundled test data from data/gdelt (102 files; 186,189 news stories)
 config/kafka.json - a jmxtrans config file to extract nominated jmx metrics from the kafka brokers.
      ksqlapp.json - a grafana dashboard
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
                 Grafana: http://localhost:3000

```      
			
 

![ Grafana Dashboard for a kSQL App](/images/ksql-grafana.png)
