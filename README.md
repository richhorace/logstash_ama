# Elastic Community: Local Testing with Logstash AMA with Rich Horace
Here are the resources from the Elastic Community Event on September 21, 2021 <br>
Description: How do you ensure your logstash configurations work as expected before ingesting into an elastisearch cluster? <b>Test before you ingest!</b> 

#### [Just Enough Docker](#just-enough-docker)
#### [Configuration Examples Walkthrough](#configuration-examples-walkthrough)
<br>

## <a id="just-enough-docker"></a> Just Enough Docker

The Elastic Stack version is controllered in the `.env` file. If the image is not already local, docker will pull down the official Elastic docker image.  Docker Containers can be launched in detached mode by adding `-d` after up.

There are three docker-compose files:<br>
<b>docker-compose.yml</b>:<br> Consider this configuration purely for reviewing the latest Elastic Stack features.  This will launch Kibana and Elasticsearch containers with an ephermeral environment based on the version specified in `.env` and any changes will not be save after the environment is stopped. <br>

```
# Launch containers in detached mode with docker-compose.yml as default
docker-compose up -d

```

<b>docker-compose-persistent.yml</b>:<br> This configuration will launch Kibana and Elasticsearch containers and create a docker volume that will persist data and changes after the containers are stopped with down or stop commands. <br>

```
# Launch containers in detached mode with -f to specify an alternate docker-compose file
docker-compose -f docker-compose-persistent.yml up -d

# Will stop containers
docker-compose -f docker-compose-persistent.yml stop

# Will stop and remove containers and -v is used to specify deletion the docker volume
docker-compose -f docker-compose-persistent.yml down -v

```

<b>docker-compose-ingest.yml</b>:<br> This configuration will launch Kibana, Elasticsearch and Logstash containers create a docker volume that will persist data and changes after the containers are stopped with down or stop commands. <br>

```
# Launch containers in detached mode with -f to specify an alternate docker-compose file
docker-compose -f docker-compose-ingest.yml up -d

# Will stop the kibana container, but keep elasticsearch and logstash running
docker-compose -f docker-compose-ingest.yml stop kibana

# Will stop and remove containers and -v is used to specify deletion the docker volume
docker-compose -f docker-compose-ingest.yml down -v

# Will only launch logstash container based on the configuration for the logstash service in the docker-compose file.
docker-compose -f docker-compose-ingest.yml up -d logstash

```

Other useful docker commands

```
# List running containers
docker ps

# Docker exec you can access a running container. This example will access logstash container from the docker-compose file
docker exec -it logstash_ama_logstash_1 bash

# Docker run will allow you to specify a docker image and remove the container once you exit the bash session.
docker run --rm -it docker.elastic.co/logstash/logstash:7.14.1 bash
```

Additional docker resources:<br>
- [Docker Office Documents:](https://docs.docker.com)
- [Elastic Logstash Docker:](https://www.elastic.co/guide/en/logstash/current/docker.html)
- [Elastic Elasticearch Docker:](https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html)
- [Elastic Kibana Docker:](https://www.elastic.co/guide/en/kibana/current/docker.html)


## <a id="#configuration-examples-walkthrough"></a> Configuration Examples Walkthrough

This walkthrough will be driven by multiple logstash pipelines with examples of different configurations that will be enabled by uncommenting and commenting in the `elastic-stack/config/logstash/pipelines.yml` <br>

Each pipeline example will build on previous example, but can also be run by itself.
- `logstash-main.conf`: bare config mininum to run logstash without errors. <br>
- `1-generate-example`: Introduces the [Generator Input Plugin](https://www.elastic.co/guide/en/logstash/current/plugins-inputs-generator.html) which allows to pass message directly into input with count of events to produce. This is great for isolating testing to specific message for debugging. <br>
- `2-dissect-example`: Introduces [File Input Plugin](https://www.elastic.co/guide/en/logstash/current/plugins-inputs-file.html) with sincedb and starting position settings. Then use Dissect Filter Plugin and Date Filter Plugin to enrich the data. A site to test [Dissect Configuration](https://dissect-tester.jorgelbg.me/) Lastly, use [File Output Plugin](https://www.elastic.co/guide/en/logstash/current/plugins-outputs-file.html) with codec setting to producing a json output file.<br>
- `3-more-complicated-example`: Introduces the [Grok Filter Plugin](https://www.elastic.co/guide/en/logstash/current/plugins-filters-grok.html) and conditionals. <br>
- `4-multiple-config-example`: Takes logstash.conf from Example 3 and creates seperating files for Input/Filter/Output <br>
-- 100-input.conf <br>
-- 500-filter.conf <br>
-- 900-output.conf <br>
- `5-output-es`: Pulls it all together with multiple file inputs, conditional filtering and outputing to multiple indices using [Elasticsearch Output Plugin](https://www.elastic.co/guide/en/logstash/current/plugins-outputs-elasticsearch.html)

## Thanks a wrap!