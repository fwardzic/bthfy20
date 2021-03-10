## LAB 00 - creating container with 2048 game

In this excercise you will create container which will contain web-server with the 2048 game.

First login to play-with-docker page as outlined in lab-access instruction.

Create one instance by clicking on " + ADD NEW INSTANCE " button.

Once created you should see following screen:


### Task 1: Repo cloning
First clone git repository with the 2048 game:

`git clone https://github.com/gabrielecirulli/2048`

> All credits goes to Gabriele Cirulli - author of the 2048 game.

### Task 1: Create dockerfile

1. create "dockerfile":

`touch dockerfile`

2. Using play-with-docker editor

<img src="https://raw.githubusercontent.com/fwardzic/bthfy20/master/docs/images/pwd-editor.png">

paste following content:
~~~~
FROM alpine:latest
MAINTAINER fwardzic
RUN apk --update add nginx
ADD 2048/ /usr/share/nginx/html
EXPOSE 80
RUN mkdir -p /run/nginx
CMD ["nginx", "-g", "daemon off;"]
~~~~

3. once done, please save.

4. Make sure that content has been added to the file:

`cat dockerfile`

### Task 2: Build container image

1. Build container using following command:

`docker build -t <your_docker_hub_user_id>/bth-2048:v1 .`

> Don't forget about "." (dot) at the end !

2. Run container 

`docker run -P -d <your_docker_hub_user_id>/bth-2048:v1`

### Task 3: Fix issue with the application


1. Open web page by clicking on the port link above terminal in Play-with-docker, 

<img src="https://raw.githubusercontent.com/fwardzic/bthfy20/master/docs/images/pwd-service.png">

or following command or in terminal of docker host using following command:

`curl 127.0.0.1:port`

2. Check container ID:

`docker ps`

~~~~
[node1] (local) root@192.168.0.8 ~
$ docker ps 
CONTAINER ID        IMAGE                  COMMAND                  CREATED             STATUS              PORTS                   NAMES
916d67bd3c6c        fwardzic/bth-2048:v1   "nginx -g 'daemon of…"   9 minutes ago       Up 9 minutes        0.0.0.0:32768->80/tcp   intelligent_saha
~~~~

3. copy nginx file from docker container to the host (replace container ID with yours)

`docker cp 916d67bd3c6c:/etc/nginx/conf.d/default.conf ./default.conf`

4. Using editor, replace string "return 404;" with the following "root /usr/share/nginx/html" in the `location /` section:

```
<snip>
location / {       
                root /usr/share/nginx/html;
        }
<snip>
```

5. Copy back updated file back to the container:

`docker cp ./default.conf 916d67bd3c6c:/etc/nginx/conf.d/default.conf`

6. Commit changes to container:

`docker commit 916d67bd3c6c <your_docker_hub_user_id>/bth-2048:v2`

Above command creates new image with additional layer. 

7. Stop old version and run new version of container based on the new image:

`docker stop 916d67bd3c6c`

`docker run -d -P <your_docker_hub_user_id>/bth-2048:v2`