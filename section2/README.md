# Running the whole framework in different isolated containers

Within this section we will run locally the whole Framework inside Different containers. We will split them in the following services, defined in a `docker-compose.yml` file:

```yaml
version: '3'
services:
  elasticsearch:
    image: bitergia/elasticsearch:6.1.0
    container_name: elasticsearch
    environment:
      - ES_JAVA_OPTS=-Xms512m -Xmx512m
    ports:
      - "9200:9200"
    networks:
      - esnet

  mariadb:
    image: mariadb:10.0
    container_name: mariadb
    command: --wait_timeout=2592000 --interactive_timeout=2592000 --max_connections=300
    environment:
      - MYSQL_ROOT_PASSWORD=
      - MYSQL_ALLOW_EMPTY_PASSWORD=yes
    networks:
      - esnet

  kibiter:
    image: bitergia/kibiter:6.1.0
    container_name: kibiter
    ports:
      - "5601:5601"
    environment:
      - ELASTICSEARCH_USER=bitergia
      - ELASTICSEARCH_PASSWORD=bitergia
      - ELASTICSEARCH_URL=http://elasticsearch:9200
      - PROJECT_NAME=Test20
      - NODE_OPTIONS=--max-old-space-size=1200
    networks:
      - esnet

  redis:
    image: redis:latest
    container_name: redis
    command: redis-server --save "" --appendonly no
    networks:
      - esnet

  arthur:
    image: acsdocker/arthur
    container_name: arthur
    networks:
      - esnet

  mordred:
    image: bitergia/mordred:18.04-04
    volumes:
      - ${PWD}/mytoken.cfg:/override.cfg
      - ${PWD}/mordred-infra.cfg:/infra.cfg
      - ${PWD}/mordred-project.cfg:/project.cfg
      - ${PWD}/mordred-dashboard.cfg:/dashboard.cfg
      - ${PWD}/projects.json:/projects.json
      - ${PWD}/identities.yaml:/identities.yaml
      - ${PWD}/menu.yaml:/home/bitergia/menu.yaml
      - ${PWD}/logs:/logs
    entrypoint: [ "/usr/local/bin/mordred" ]
    command: [ "-c", "/infra.cfg", "/dashboard.cfg", "/project.cfg", "/override.cfg"]
    networks:
      - esnet

networks:
  esnet:

```

The files will be the same, except for some minor changes in the configuration (to point the ES container and not localhost).

There are several important things here:

- `ports`: it maps the desired port to the host
- `environment`: sets ENV variables inside the container
- `networks`: define a network with the default settings (which is a load-balanced overlay network)
- `command`: set the command we are about to launch against the container
- `entrypoint`: overrides the entrypoint of the image
- `volumes`: maps a file from the host to the container


Again, we need first to generate a file with the your own github token:

```
$ cat mytoken.cfg
[github]
api-token=XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
```

So with that, we can execute the whole set of containers:

```
$ docker-compose up
```

> **NOTE**: We must run at Kibana startup the following query, to avoid issues with the indexPattern assignment:
```
curl -XPOST "http://127.0.0.1:5601/api/kibana/settings/indexPattern:placeholder" \
          -H 'Content-Type: application/json' -H 'kbn-version: '6.1.0-1 \
          -H 'Accept: application/json' -d '{"value": "*"}' \
          --silent --output /dev/null ;
```
