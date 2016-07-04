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
1.1. Pre-requisites: You must have installed docker and docker-compose locally.
1.2. To deploy the stack locally, you need to be into the compose folder (cwd) and then you must run:
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

#### 2. With kubernetes (k8s)
Using kubernetes and images pulled from docker hub, and defining the stack as yaml files with k8s syntax, to automate and manage the stack as a production environment.
