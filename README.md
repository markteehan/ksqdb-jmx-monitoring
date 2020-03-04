# ksql-grafana-monitoring

This repo demonstrates monitoring a kSQL pipeline using a Grafana dashboard.

It consits of:
docker-compose.yml - docker compose yaml for all containers
             runme - a shell script to initialize all objects, load data and create the streaming pipeline
           getNews - a shell script to pull more test data from data.gdeltproject.org
          loadNews - a shell script to load bundled test data from data/gdelt (102 files; 186,189 news stories)
 config/kafka.json - a jmxtrans config file to extract nominated jmx metrics from the kafka brokers.
      ksqlapp.json - a grafana dashboard
      
      
			
 
