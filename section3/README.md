# Running the whole framework in different isolated services. The cluster

Within this section we will scale our application and enable load-balancing. To do this, we must go one level up in the hierarchy of a distributed application using Services.

Let's consider that we will need a high availability of the Kibana instance. We will take the `docker-compose.yml` file from section 2 and modify the service as follows:

```yaml
kibiter:
  image: bitergia/kibiter:6.1.0-optimized
  ports:
    - "5601:5601"
  deploy:
    replicas: 2
    resources:
      limits:
        cpus: "0.4"
        memory: 256M
    restart_policy:
      condition: on-failure
  environment:
    - ELASTICSEARCH_USER=bitergia
    - ELASTICSEARCH_PASSWORD=bitergia
    - ELASTICSEARCH_URL=http://elasticsearch:9200
    - NODE_OPTIONS=--max-old-space-size=1200
  networks:
    - esnet
```

Those changes will do the following:

- `replicas`: Run 2 instances of that image as a service called kibiter, limiting each one to use, at most, 40% of the CPU (across all cores), and 256MB of RAM.

- `restart_policy`: Immediately restart containers if one fails.

Before starting, we will need a master node for deploying the stack (a local one, and we will dig into it later):
```
docker swarm init
```

Once done, we will deploy the `grimoirelab` app using the file:

```
docker stack deploy -c docker-compose.yml grimoirelab
```

Our single service stack is running 2 container instances of our deployed image on one host.

Get the service ID for the one service in our application:

```
docker service ls
```
We can see there that there are two replicas, binded to the 2 containers:

```
docker service ps grimoirelab_kibiter
```

Finally we can reach the Kibiter at `0.0.0.0:5601`

Last but not least, we can see the status of the cluster by binding the visualizer container we've added:

```
visualizer:
  image: dockersamples/visualizer:stable
  ports:
    - "8080:8080"
  volumes:
    - "/var/run/docker.sock:/var/run/docker.sock"
  deploy:
    placement:
      constraints: [node.role == manager]
  networks:
    - esnet
```

As we can see, the laptop itself is running out of memory. For the next examples we are going to use some other lightweight examples to see its usage.

It's important to mention that, we can also add more replicas and re-deploy by launching the same command:

```
docker stack deploy -c docker-compose.yml grimoirelab
```

## Clean up

To continue, we will clean up the things by running:

```
docker stack rm grimoirelab
```

And finally:

```
docker swarm leave --force
```
