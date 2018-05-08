export CONSUL_CONTAINER=$(docker run -d --name=consul --net=host gliderlabs/consul-server -bootstrap -bind=172.30.0.225)
echo $CONSUL_CONTAINER

curl 172.30.0.225:8500/v1/catalog/services ; echo

export REGISTRATOR_CONTAINER=$(docker run -d --name=registrator --net=host --volume=/var/run/docker.sock:/tmp/docker.sock gliderlabs/registrator:latest consul://localhost:8500)
echo $REGISTRATOR_CONTAINER

export NGINX_1=$(docker run -d -P -e "SERVICE_TAGS=pr1111" nginx)
echo $NGINX_1
export NGINX_2=$(docker run -d -P -e "SERVICE_TAGS=pr2222" nginx)
echo $NGINX_2

export REDIS_1=$(docker run -d -P -e "SERVICE_TAGS=pr1111" redis)
echo $REDIS_1

export REDIS_2=$(docker run -d -P -e "SERVICE_TAGS=pr2222" redis)
echo $REDIS_2

curl 172.30.0.225:8500/v1/catalog/services ; echo
curl 54.254.139.197:8500/v1/catalog/service/nginx\?tag=pr1111 ; echo

export CONSUL_TEMPLATE=$(docker run -d --rm -v "$(pwd):/tmp/config" hashicorp/consul-template:alpine consul-template -template "/tmp/config/config.ctmpl:/tmp/config/config.txt" -consul-addr=172.30.0.225:8500)
cat config.txt

export NGINX_3=$(docker run -d -P -e "SERVICE_TAGS=pr3333" nginx)
echo $NGINX_3

export REDIS_3=$(docker run -d -P -e "SERVICE_TAGS=pr333" redis)
echo $REDIS_3

cat config.txt

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
