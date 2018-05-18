# Docker ELK stack
Run the latest version of the ELK (Elasticsearch, Logstash, Kibana) stack with Docker and Docker Compose.
It will give you the ability to analyze any data set by using the searching/aggregation capabilities of Elasticsearch
and the visualization power of Kibana.

Based on the official Docker images:
* [elasticsearch](https://github.com/elastic/elasticsearch-docker)
* [logstash](https://github.com/elastic/logstash-docker)
* [kibana](https://github.com/elastic/kibana-docker)

**Note**: Other branches in this project are available:

* ELK 6 with X-Pack support: https://github.com/deviantony/docker-elk/tree/x-pack
* ELK 6 in Vagrant: https://github.com/deviantony/docker-elk/tree/vagrant
* ELK 6 with Search Guard: https://github.com/deviantony/docker-elk/tree/searchguard

## Requirements

### Host setup

1. Install [Docker](https://www.docker.com/community-edition#/download) version **1.10.0+**
2. Install [Docker Compose](https://docs.docker.com/compose/install/) version **1.6.0+**
3. Clone this repository

### Bringing up the stack

**Note**: In case you switched branch or updated a base image - you may need to run `docker-compose build` first

Start the ELK stack using `docker-compose`:

```console
$ docker-compose up
```

You can also choose to run it in background (detached mode):

```console
$ docker-compose up -d
```

Give Kibana a few seconds to initialize, then access the Kibana web UI by hitting
[http://localhost:5601](http://localhost:5601) with a web browser.

By default, the stack exposes the following ports:
* 5000: Logstash TCP input.
* 9200: Elasticsearch HTTP
* 9300: Elasticsearch TCP transport
* 5601: Kibana

**WARNING**: If you're using `boot2docker`, you must access it via the `boot2docker` IP address instead of `localhost`.

**WARNING**: If you're using *Docker Toolbox*, you must access it via the `docker-machine` IP address instead of
`localhost`.

Now that the stack is running, you will want to inject some log entries. The shipped Logstash configuration allows you
to send content via TCP:

```console
$ nc localhost 5000 < /path/to/logfile.log
```

## Initial setup

### Default Kibana index pattern creation

When Kibana launches for the first time, it is not configured with any index pattern.

#### Via the Kibana web UI

**NOTE**: You need to inject data into Logstash before being able to configure a Logstash index pattern via the Kibana web
UI. Then all you have to do is hit the *Create* button.

Refer to [Connect Kibana with
Elasticsearch](https://www.elastic.co/guide/en/kibana/current/connect-to-elasticsearch.html) for detailed instructions
about the index pattern configuration.

#### On the command line

Create an index pattern via the Kibana API:

```console
$ curl -XPOST -D- 'http://localhost:5601/api/saved_objects/index-pattern' \
    -H 'Content-Type: application/json' \
    -H 'kbn-version: 6.2.3' \
    -d '{"attributes":{"title":"logstash-*","timeFieldName":"@timestamp"}}'
```

The created pattern will automatically be marked as the default index pattern as soon as the Kibana UI is opened for the first time.

## Configuration

**NOTE**: Configuration is not dynamically reloaded, you will need to restart the stack after any change in the
configuration of a component.

### How can I tune the Kibana configuration?

The Kibana default configuration is stored in `kibana/config/kibana.yml`.

It is also possible to map the entire `config` directory instead of a single file.

### How can I tune the Logstash configuration?

The Logstash configuration is stored in `logstash/config/logstash.yml`.

It is also possible to map the entire `config` directory instead of a single file, however you must be aware that
Logstash will be expecting a
[`log4j2.properties`](https://github.com/elastic/logstash-docker/tree/master/build/logstash/config) file for its own
logging.

### How can I tune the Elasticsearch configuration?

The Elasticsearch configuration is stored in `elasticsearch/config/elasticsearch.yml`.

You can also specify the options you want to override directly via environment variables:

```yml
elasticsearch:

  environment:
    network.host: "_non_loopback_"
    cluster.name: "my-cluster"
```

### How can I scale out the Elasticsearch cluster?

Follow the instructions from the Wiki: [Scaling out
Elasticsearch](https://github.com/deviantony/docker-elk/wiki/Elasticsearch-cluster)

## Storage

### How can I persist Elasticsearch data?

The data stored in Elasticsearch will be persisted after container reboot but not after container removal.

In order to persist Elasticsearch data even after removing the Elasticsearch container, you'll have to mount a volume on
your Docker host. Update the `elasticsearch` service declaration to:

```yml
elasticsearch:

  volumes:
    - /path/to/storage:/usr/share/elasticsearch/data
```
This will store Elasticsearch data inside `/path/to/storage`.

## Development Guidelines 

* Checkout into master git checkout master
`git checkout master`
* Create a branch from master for your work `git checkout -b <Ticket no + description>`
`git checkout  -b <Ticket no + description>`
Once you are done, create MR against develop branch; put the label needs review and get code reviewed
* This is optional step: if there is conflict when merging to develop, create an Temp branch from your feature branch `git checkout -b <Ticket no + description + temp.dev>` 
*   Pull latest develop `git checkout develop && git pull && git checkout <Ticket no + description + temp.dev>` && `git merge develop`
*   Resolve conflict and create MR against develop from this Temp branch. Temp branch should be marked for delete while creating MR against develop.
*   Please follow these steps in case of conflicts with Develop
    ```
    git checkout  -b <Temp + Ticket no + description>
    git checkout develop
    git pull origin develop
    git checkout  -b <Temp + Ticket no + description>
    git merge develop
    ```
* Put the label ready for testing and Needs Master Merge and Feature branch for MR against master  always be updated to latest code on master.
* Once things are good at master branch raise Merge request to production branch from master



