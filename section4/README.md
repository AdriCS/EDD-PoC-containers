# Set up your swarm in different machines

As extracted from the documentation:

>A swarm is a group of machines that are running Docker and joined into a cluster. After that has happened, you continue to run the Docker commands you’re used to, but now they are executed on a cluster by a swarm manager. The machines in a swarm can be physical or virtual. After joining a swarm, they are referred to as nodes.

>Swarm managers can use several strategies to run containers, such as “emptiest node” -- which fills the least utilized machines with containers. Or “global”, which ensures that each machine gets exactly one instance of the specified container. You instruct the swarm manager to use these strategies in the Compose file, just like the one you have already been using.

>Swarm managers are the only machines in a swarm that can execute your commands, or authorize other machines to join the swarm as workers. Workers are just there to provide capacity and do not have the authority to tell any other machine what it can and cannot do.

>Up until now, you have been using Docker in a single-host mode on your local machine. But Docker also can be switched into swarm mode, and that’s what enables the use of swarms. Enabling swarm mode instantly makes the current machine a swarm manager. From then on, Docker runs the commands you execute on the swarm you’re managing, rather than just on the current machine.

## Creating the cluster and the vms

As in the previous section, we will create a cluster by running:

```
docker swarm init
```

And after that, the vms:
```
docker-machine create --driver virtualbox --virtualbox-hostonly-cidr "192.168.90.1/24" myvm1
docker-machine create --driver virtualbox --virtualbox-hostonly-cidr "192.168.90.1/24" myvm2
```

NOTE: If you get the following message:
```
WARNING >>>
This machine has been allocated an IP address, but Docker Machine could not
reach it successfully.
```
Remove the `virtualbox-hostonly-cidr` or try to find if your `vboxnet0` is up. If not, you can set it up as:
```
sudo ifconfig vboxnet0 up
```

See https://github.com/docker/machine/issues/2136 for workarounds

## Get their ip addresses

Now we have two VMs created, named myvm1 and myvm2.

Use this command to list the machines and get their IP addresses:

```
docker-machine ls
```
So once we have the IPs, we must set the vm1 as a manager by running:
```
docker-machine ssh myvm1 "docker swarm init --advertise-addr 192.168.90.100"
```
Then, we can bind the second vm to the cluster as a worker:
```
docker-machine ssh myvm2 "docker swarm join \\n--token SWMTKN-1-5evwn1486yozy24obt5887uwdn36wved0kj5smfw0k39t10vc5-dehmns93fewsi877vhs6xuxzh \\n192.168.90.100:2377"
```

So by running the following command, we will check its availability:
```
docker-machine ssh myvm1 "docker node ls"
```
So now, to be able to deploy inside the cluster, we will bind the manager into our terminal, to be able to then launch the app:
```
eval $(docker-machine env myvm1)
```
Finally, we can deploy the app in the cluster in both machines:

```
docker stack deploy -c docker-compose.yml testapp
```
Note that due to the high requirements of the other stack, we will use a simple test application and the visualizer.

For this example, the cool thing is the visualizer, and also how we can re-deploy the nodes and see how is it updated.

We will also add `volumes`, to provide persistence to the data into the vms:

```yaml
version: "3"
services:
  web:
    image: albertinisg/edd-sample-image
    deploy:
      replicas: 5
      restart_policy:
        condition: on-failure
      resources:
        limits:
          cpus: "0.1"
          memory: 50M
    ports:
      - "80:80"
    networks:
      - esnet
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
  redis:
    image: redis
    ports:
      - "6379:6379"
    volumes:
      - "/home/docker/data:/data"
    deploy:
      placement:
        constraints: [node.role == manager]
    command: redis-server --appendonly yes
    networks:
      - esnet
networks:
  esnet:
```

## Clean up

We must remove the app:

```
docker stack rm testapp
```

and unset the environment defined before:
```
  eval $(docker-machine env -u)
```

Finally we can stop and remove the VMs by running:
```
docker-machine stop myvm1 myvm2
docker-machine rm myvm1 myvm2
```
