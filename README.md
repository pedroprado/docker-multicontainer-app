5.MULTI-CONTAINER APPLICATION

5.1.Dockering

This app contains three services: one react app, two node apps.

For dockering this app we need to set and Dockerfile do each service, and connect these containers using a docker compose file.

The structure pointed in the image above shows us that, in the docker-compose file, we need the following constraints:
```
React app should connect to the Server ;
Server should connect to Redis and Postgres;
Worker should connect to Redis;
```

The docker compose file should reflect the constraints above;

5.2.The role of nginx 
Nginx have ONE JOB: to route the incoming requests from the browser to the React Server or to the Express Server (nginx looks to the path request). Thus, we need a service for nginx too.

Requests that start with “/” go to React Server. Requests that start with “/api” go to Express Server. Nginx “chops off” the “/api” when a request that start with that comes in (it comes in “/api/values/all”, it  comes out from nginx “/values/all”).

Nginx default.conf file
For nginx to work with the proposed architecture, we must create a Service of nginx in the docker-compose file. For this nginx service to work, we have to config it properly, using the default.conf file (this file is used to create a custom image of nginx).

Web-socket problem
Every time a React app starts in DEVELOPMENT MODE, it wants to keep a active connection with the Nginx Server.
This can be made adding the configuration needed to the default.conf file.

```
  location /sockjs-node{
        proxy_pass http://client;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "Upgrade";
    }
```
 

5.3.Continuous integration flow
A multi-container application must not have a phase where ElasticBeanstalk builds the production image (it is to slow!).


In a multi-container app it is recommended to have the CI server (Travis, in this case) to build the production image. The app server (ElasticBeanstalk) should only “pull” this images, and not build it!

5.4.Multicontainer application deploy to Amazon Elastic Bean Stalk
When we have a single container application (with ONE dockerfile), Amazon EB is capable of looking to a single Dockerfile and build and run the image.

That does not happen when we have a multicontainer application. That’s why we have to build  the images before, and push the built images to EB. EB is not capable of reading multiple dockerfiles (that exist in a multicontainer application).

We need to tell EB where to pull the images and how to connect them. For that we use Dockerrun.aws.json file.

Task definitions in Amazon ECS (Elastic Container Service)
Amazon EB does not really know how to work with containers. EB delegates the hosting services to ECS.
ECS works with “Task Definition”, which are files that tell ECS how to run containers.
Documentation: https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_definitions.html


Dockerrun.aws.json file should look like this:
```json{
    "AWSEBDockerrunVersion": 2,
    "containerDefinitions":[  
        {
            "name": "client",    //name shown in EB dashboards
            "image": "phsprado/complex-client",  //image name from docker hub
            "hostname": "client",    //the host (service) created
            "essential": false
        },
        {
            "name": "nginx",
            "image": "phsprado/complex-nginx",
            "essential": true,   //at least one container must be essencial
            //essencial flag means that "if this container stops, the rest should stop too"
            "portMappings": [
                {
                    "hostPort": 80,  // port of the machine
                    "containerPort": 80  //port of the container
                }
            ],
            "links": ["client", "server"]  //this flag has the same purpose as "depends_on" in the docker-compose file
        },
        {
            "name": "server",
            "image": "phsprado/complex-server",
            "hostname": "api",
            "essential": false  
        },
        {
            "name": "worker",
            "image": "phsprado/complex-worker",
            "hostname": "worker",
            "essential": false
        }
    ]
}
```

Note: Json does not permit comment. The comments must be removed for it to be a valid json.


