
# Service discovery with Consul and Registrator and Consul Template (for dev environments)

It is exaple how to generate and keep updated configs (e.g. Nginx) based on information from Consul

## Run consul and Registrator
Run consul server and save its container ID to CONSUL_CONTAINER variable

```
export CONSUL_CONTAINER=$(docker run -d --name=consul --net=host gliderlabs/consul-server -bootstrap -bind=172.30.0.225)

echo $CONSUL_CONTAINER
```

Check it is running and here is no registered services in Consul
```
curl 172.30.0.225:8500/v1/catalog/services ; echo
```

Run Registrator and save its container ID to REGISTRATOR_CONTAINER variable
```
export REGISTRATOR_CONTAINER=$(docker run -d --name=registrator --net=host --volume=/var/run/docker.sock:/tmp/docker.sock gliderlabs/registrator:latest consul://localhost:8500)

echo $REGISTRATOR_CONTAINER
```

## Demo
### Run 2 containers with Nginx ad 2 containers with Redis.
There are two different SERVICE_TAGS to be able to filter services.
```
export NGINX_1=$(docker run -d -P -e "SERVICE_TAGS=myenv1111" nginx)
echo $NGINX_1
export REDIS_1=$(docker run -d -P -e "SERVICE_TAGS=myenv1111" redis)
echo $REDIS_1

export NGINX_2=$(docker run -d -P -e "SERVICE_TAGS=myenv2222" nginx)
echo $NGINX_2
export REDIS_2=$(docker run -d -P -e "SERVICE_TAGS=myenv2222" redis)
echo $REDIS_2
```

All 4 sevices registered in Consul 
```
curl 172.30.0.225:8500/v1/catalog/services ; echo
```

Try to filter services by tag
```
curl 54.254.139.197:8500/v1/catalog/service/nginx\?tag=myenv1111 ; echo
```

### Generate and update confix for Nginx or HAProxy with *Consul Template* based on data from Consule

Run Consul Template and save its container ID to CONSUL_TEMPLATE variable.
Mount template for config inside the container and run consul-template to pull data from Consul and regenerate config after any update.
```
export CONSUL_TEMPLATE=$(docker run -d --rm -v "$(pwd):/tmp/config" hashicorp/consul-template:alpine consul-template -template "/tmp/config/config.ctmpl:/tmp/config/config.txt" -consul-addr=172.30.0.225:8500)
```

Check generated file
```
cat config.txt
```

Run one more set of containers.
```
export NGINX_3=$(docker run -d -P -e "SERVICE_TAGS=myenv3333" nginx)
echo $NGINX_3

export REDIS_3=$(docker run -d -P -e "SERVICE_TAGS=myenv3333" redis)
echo $REDIS_3
```
config.txt will be updated automatically.
```
cat config.txt
```

Clean you environment
```
docker stop $CONSUL_TEMPLATE
docker rm $CONSUL_TEMPLATE

docker stop $NGINX_1 $NGINX_2 $NGINX_3
docker rm $NGINX_1 $NGINX_2 $NGINX_3

docker stop $REDIS_1 $REDIS_2 $REDIS_3
docker rm $REDIS_1 $REDIS_2 $REDIS_3

docker stop $REGISTRATOR_CONTAINER
docker rm $REGISTRATOR_CONTAINER

docker stop $CONSUL_CONTAINER
docker rm $CONSUL_CONTAINER
```
