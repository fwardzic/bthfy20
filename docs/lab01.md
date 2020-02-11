## LAB 01 - creating container with 2048 game

In this excercise you will create container which will contain web-server with the 2048 game.

First clone git repository with the 2048 game:

`git clone https://github.com/gabrielecirulli/2048`

All credits goes to Gabriele Cirulli - author of the 2048 game.

then create "dockerfile":

`touch dockerfile`

Using PWD editor or vi in terminal, paste following content:

~~~~
FROM alpine:latest
MAINTAINER fwardzic
RUN apk --update add nginx
ADD 2048/ /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
~~~~

Build container using following command:

`docker build -t <your_docker_hub_user_id>/bth-2048:v1 .`

Run container 

`docker run -P -d <your_docker_hub_user_id>/bth-2048:v1`

check web what's displayed on the webpage

1. first find our host port to which our container redirected traffic

`docker ps`

2. Open web page in docker host using following command or by clicking on the port link abouve terminal in PWD.

`curl 127.0.0.1:port`

3. Once you found that you get 404 no response, then you need to modify nginx configuration file:

`/etc/nginx/conf.d/default.conf`

4. (option 1) You can copy it from running container:

`docker cp <image_ID>:/<path> <host_path>`

5. (option 2) You can modify file inside container and commit container with the new layer.

while having your container running, enter into it:

first check container ID:

`docker ps`

`docker exec –it db2c356a51c1 /bin/sh`

edit file:

`vi /etc/nginx/conf.d/default.conf`

replace existing statement with the following in the `location /` section:

```
<snip>
location / {       
                root /usr/share/nginx/html;
        }
<snip>
```
Commit container:

`docker commit db2c356a51c1 2048:v2`

6. Stop old version and run new version of container:

`docker stop db2c356a51c1`

`docker run -d -P <your_docker_hub_user_id>/bth-2048:v2`