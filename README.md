Project for a Dockerized Microservics

I am going to use Spring Netflix OSS to build microservices API for a very basic Shopping cart application

Goals: -
1. Docker for deployment
2. Spring Boot Cloud Config - deployed as docker container
3. Spring Netflix Eureka Server - deployed as docker container
4. Spring Eureka Client - deployed as docker container
5. Spring Feign Client
6. Spring Hysterix circuit breaker
7. Spring Hysterix Dashboard
8. Spring ZuulProxy
9. Spring Turbine
10. mysql - deployed as docker container and populated via script on start up
11. Stretch Goal - Async API aggregator written in RxJava
12. Stretch Goal - Jenkins
13. Spring sleuth  And ELK
14. Distributed caching


<div>
I have written <b>install.sh</b> which does the job of CI of building and packaging the Spring boot application and put them in a directory 
from where they can be mounted to docker volumes.
</div>

<b>install.sh</b> will also take care of bringing all containers using docker-compose. 
So clone the project, create a volume directory in your system and modify the install.sh and docker-compose.yml accordingly. Finally run .install.sh.
TODO - modify the install.sh so that it creates the working directory

Under the hood this is what happens -
1. Builds all microservices, eureka server, config
2. Bring up mysql
3. Populate mysql with DDL and DML if not done already
4. Bring up config server
5. Bring up Eureka Server
6. Bring up microservices - Eureka clients. 
7. Add them into one network so that they can communicate

Working Endpoints so far :-
<div>
<ul>
<li>Eureka - http://localhost:1111/eureka</li>
<li>Customer - localhost:localhost:2222/customer </li>
<li>Inventory - http://localhost:3333/inventory </li>
<li>Invoice - http://localhost:4444/invoice</li>
<li>Config - http://localhost:5555/customer-service/dev</li>
<li>
</ul>
</div>

-------------------------------------------------------------------------------------------------------------

This old doc is here for doc purpose only

Old ->
To start the Mysql container: <BR>
<b><code>docker run --name docker-mysql -e MYSQL_ROOT_PASSWORD=test -P -d mysql</code></b>

To populate the data , go to sql command prompt with 
sudo  docker run -it --link docker-mysql:mysql --rm mysql sh -c 'exec mysql -h"$MYSQL_PORT_3306_TCP_ADDR" -P"$MYSQL_PORT_3306_TCP_PORT" -uroot -p"$MYSQL_ENV_MYSQL_ROOT_PASSWORD"'

Run init.sql


Then Create a network , this should also be ideally part of docker compose

<div>
<b><code>sudo docker network create microservicesnet</b></code>
</div>
Connect mysql to it -

<div><b><code>sudo docker network connect microservicesnet docker-mysql</b></code></div>


Commands ->

As of now I go to each inside each microservice and generate its jar
example,
cd customers
To build and package a jar to target folder :<BR>
<b><code>mvn clean package</code></b>


Copy all microservices jar to the place which is shared with the volume - Ideally this should be done througha CI JOb (TODO) <br>
<code>cp customer-0.0.1-SNAPSHOT.jar.original /home/hitesh/jarloc</code>



once you have all your jars in jarloc (Jar location), you can bring all the micro-services as well as the discovery server up with

<div><b><code>sudo docker-compose up -d</code></b></div>


The Java 8 base image used to build the microservice containers is also checked in dockerhub and can be seprately downloaded as
<div><b><code>docker pull hiteshjoshi1/microservice-docker-cart-example</code></b></div

The base image can be built using

<div><b><code>docker build -t microservice/baseserviceimg .</code></b></div>



___________________________________________________________________________________________________________________________

NOT required-

Building individual containers without docker compose


To Build the image from Docker File - Custom image as specified in the Dockerfile ---> (Note the .) <br>

1. Build(Or Rebuilding) Service Discovery Customer, Inventory , Invoice from the docker file .
<div>
<ul>
<li><b><code>docker build -t microservice/customer . </code></b>
<li><b><code>docker build -t microservice/inventory . </code></b>
<li><b><code>docker build -t microservice/invoice . </code></b>
</ul>
</div>
Building a container happens once, once it is built you can run it from the same folder and give it a name.

To Run the  Custom Image, notice the linkage to the mysql container for microservices--><br>
<ul>
<li><b><code>docker run --name docker-discovery  -P -d microservice/serviceDiscovery</code></b>
<li><b><code>docker run --name docker-customer --link docker-mysql:mysql -P -d microservice/customer</code></b>
<li><b><code>docker run --name docker-inventory --link docker-mysql:mysql -P -d microservice/inventory</code></b>
<li><b><code>docker run --name docker-invoice --link docker-mysql:mysql -P -d microservice/invoice</code></b>
</ul>


In this example, all my microservices connect to the same DB, but this can be easily changed above.


__________________________________________________________________________________________________________________________________

<b>References and Citations - </b> <br>
https://alexandreesl.com/2016/01/08/docker-using-containers-to-implement-a-microservices-architecture/ <br>
https://www.3pillarglobal.com/insights/building-a-microservice-architecture-with-spring-boot-and-docker-part-iii <br>
https://github.com/kbastani/spring-cloud-microservice-example</br>
https://github.com/sqshq/PiggyMetrics</br>




