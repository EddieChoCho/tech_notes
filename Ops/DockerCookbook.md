# Docker Cookbook
- by Sébastien Goasguen

### 3 Docker Networking

#### 3.1 Finding the IP Address of a Container
* $docker inspect
	```
	$ docker run -d --name nginx nginx
	$ docker inspect --format '{{ .NetworkSettings.IPAddress }}' nginx
	172.17.0.2
	```
* $docker exe
	```
	$ docker exec -ti nginx ip add | grep global
 	inet 172.17.0.2/16 scope global eth0
	```
* check the /etc/hosts file in the container
	```
	$ docker run -d --name foobar -h foobar busybox sleep 300
	$ docker exec -ti foobar cat /etc/hosts | grep foobar
	172.17.0.4 foobar
	```
* enter a shell in the container and issue standard Linux commands at the prompt
	```
	$ docker exec -ti nginx bash
	root@a3c1f7edb00a:/# cat /etc/hosts
	```
#### 3.2 Exposing a Container Port on the Hos

#### 3.3 Linking Containers in Docker
##### Link containers
* using the `--link` option of the `docker run` command.
	```
	$ docker run -d --name database -e MYSQL_ROOT_PASSWORD=root mysql
	$ docker run -d --link database:db --name web runseb/hostname
	$ docker run -d --link web:application --name lb nginx
	$ docker ps
	CONTAINER ID IMAGE COMMAND ... PORTS NAMES
	507edee2bbcf nginx "nginx -g 'daemon of ... 80/tcp, 443/tcp lb
	62c321acb102 runseb/hostname "python /tmp/hello ... 5000/tcp web
	cf17b64e7017 mysql "/entrypoint.sh mysq ... 3306/tcp database
	```

* As result, the application and load-balancer containers now contains environment variables:
	```
	$ docker exec -ti web env | grep DB
	DB_PORT=tcp://172.17.0.13:3306
	DB_PORT_3306_TCP=tcp://172.17.0.13:3306
	DB_PORT_3306_TCP_ADDR=172.17.0.13
	DB_PORT_3306_TCP_PORT=3306
	DB_PORT_3306_TCP_PROTO=tcp
	DB_NAME=/web/db
	DB_ENV_MYSQL_ROOT_PASSWORD=root
	DB_ENV_MYSQL_MAJOR=5.6
	DB_ENV_MYSQL_VERSION=5.6.25

	$ docker exec -ti lb env | grep APPLICATION
	APPLICATION_PORT=tcp://172.17.0.14:5000
	APPLICATION_PORT_8080_TCP=tcp://172.17.0.14:5000
	APPLICATION_PORT_8080_TCP_ADDR=172.17.0.14
	APPLICATION_PORT_8080_TCP_PORT=5000
	APPLICATION_PORT_8080_TCP_PROTO=tcp
	APPLICATION_NAME=/lb/application
	```
	
* The /etc/hosts file is also automatically updated to contain information for name resolution:
	```
	$ docker exec -ti web cat /etc/hosts
	172.17.0.14 62c321acb102
	172.17.0.13 db cf17b64e7017 database

	$ docker exec -ti lb cat /etc/hosts
	172.17.0.15 507edee2bbcf
	172.17.0.14 application 62c321acb102 web
	```
## References 
* [Docker Cookbook - by Sébastien Goasguen](https://www.oreilly.com/library/view/docker-cookbook/9781491919705/)
* [Legacy container links](https://docs.docker.com/network/links/)

