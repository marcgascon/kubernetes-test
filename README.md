# kubernetes/docker example

## Abstract
The main purpose of this repo is to learn about how to use docker containers and kubernetes.

To do it, we will provide service to a simple hello world ruby app that also connects to a redis. The ruby app has to be scalable (and load balanced). The deploy of the containers has to ba automated as much as possible. It has to be a simple and lightweight example but as much similar as we can to a production environment.

```
                              +--------------+
                    +--------->              +----------+
                    |         |   RUBY APP   |          |
             +------+-+       |              |     +----v------+
             |        |       +--------------+     |           |
    +-------->   LB   |                            |   REDIS   |
             |        |                            |           |
             +------+-+       +--------------+     +----^------+
                    |         |              |          |
                    |         |   RUBY APP   |          |
                    +--------->              +----------+
                              +--------------+
```

## How to deploy
In this repo we will provide 2 ways to deploy the ruby app and the redis-server to containers. The 2 ways are automated as much as I can and you only need to follow these steps to be able to deploy the full stack:
#### 1. With Docker and docker-compose (locally)
**Pre-requisites**: You must have installed docker and docker-compose locally.
To deploy the stack locally, you need to be into the compose folder (cwd) and then you must run:
```bash
# Launch the containers defined in the docker-compose.yml daemonized
docker-compose up -d

# Start/Stop the containers:
docker-compose start/stop

# Delete the stack
docker-compose down
```

With the stack running you can then make a request to the app and see if everything is working as expected. First, run this command to know the ipaddress from where to access the app:
```bash
$ docker inspect --format='{{.Name}} - {{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' $(docker ps -aq)
/compose_ruby_app_1 - 172.20.0.3
/compose_redis_1 - 172.20.0.2
```
Then test it:
```bash
$ curl http://172.20.0.3:4567/
Hello World!
```

Note that with docker compose we didn't provide high availability with the ruby app, as defined in the abstract. We only have one container for the ruby app and another one for the redis. This is just for dev purposes.

#### 2. With kubernetes (k8s)
Using kubernetes and images pulled from docker hub, and defining the stack as yaml files with k8s syntax, to automate and manage the stack as a production environment.

In the k8s folder you can find 5 files. The Dockerfile doesn't define the env variables, because kubernetes provides you the endpoint and port as environment variables of each service running in it. That's the reason I updated the hello-world.rb, when doing the redis connection, the environment variables for host and port are write to fit with the ones that k8s provides us.

To deploy this stack we need:
1- The kubectl CLI that interacts with kubernetes service (I have the service running locally inside docker containers but you can also use the service in cloud environments).
2- The container images that I defined in the yaml's must be available in a registry. (In my case I uploaded the hello-world ruby app image into docker hub and I used a default redis image available also in docker hub)

Testing the deploy of the stack:
```bash
# Create a deployment that creates one pod running redis.
$ kubectl create -f run-redis.yaml 
deployment "redis" created

# Expose the redis service to be accessible:
$ kubectl create -f svc-redis.yaml 
service "redis" created

# Create another deployment that creates 2 replicas of a pod that runs our ruby app image:
$ kubectl create -f run-rubyapp.yaml 
deployment "app-hello-world" created

# Expose the app deployment as a service. This provides us an IP that load balance the traffic to the 2 pods
$ kubectl create -f svc-rubyapp.yaml 
service "hello-world" created
```
And this is the final result of the deploy:
```bash
$ kubectl get pods
NAME                               READY     STATUS             RESTARTS   AGE
app-hello-world-4217925319-47f39   0/1       ImagePullBackOff   0          3m
app-hello-world-4217925319-wdac4   0/1       ImagePullBackOff   0          3m
k8s-etcd-127.0.0.1                 1/1       Running            0          6d
k8s-master-127.0.0.1               4/4       Running            0          6d
k8s-proxy-127.0.0.1                1/1       Running            0          6d
redis-1793418757-oo2rg             1/1       Running            0          6m

$ kubectl get deployments
NAME              DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
app-hello-world   2         2         2            2           3m
redis             1         1         1            1           6m

$ kubectl get services
NAME          CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
hello-world   10.0.0.9     <none>        4567/TCP   3m
kubernetes    10.0.0.1     <none>        443/TCP    6d
redis         10.0.0.239   <none>        6379/TCP   4m

$ curl http://10.0.0.9:4567/
Hello World!
```
