## student-list 

This repo is a simple application to list student with a webserver (PHP) and API (Flask)

![project](https://user-images.githubusercontent.com/18481009/84582395-ba230b00-adeb-11ea-9453-22ed1be7e268.jpg)



------------
## Author

First name : Konan Junior Aime Stephane

Surname : KOUADIO

For Eazytraining 17th Bootcamp

Period : January-February-March

LinkedIn: [https://www.linkedin.com/in/kouadio-konan-junior-aime-stephane/](https://www.linkedin.com/in/kouadio-konan-junior-aim%C3%A9-st%C3%A9phane/)

email : junior.kouadio21@inphb.ci

## Goals

The goal of this project is to create et deploy some microservices with the source code given to containerize a student list application of 2 modules :

- Module 1: It's a REST API written in Flask (with basic authentication needed) who send the desire list of the student based on JSON file
- Module 2 : Its's a web app written in HTML + PHP who enable end-user to get a list of students


## The process

My work was to :
1) Write the Dockerfile of api
2) Write the docker-compose
3) build one container for each module
4) make them interact with each other
5) provide a private registry


## Resolution of the problems

### 0) The files' role

In my delivery you can find three main files : ***Dockerfile*** and ***docker-compose.yml***

- docker-compose.yml:		to launch the application (API and web app)
- simple_api/student_age.py:	contains the source code of the API in python
- simple_api/Dockerfile: 	to build the API image with the source code in it
- simple_api/student_age.json: 	contains student name with age on JSON format
- index.php: 			PHP  page where end-user will be connected to interact with the service to list students with their age.

### 2) Build and test

0) Clone the code:

```
git clone https://github.com/diranetafen/student-list.git
```
1)  Change directory and edit of the Dockerfile for api with the instructions informations in `./simple_api/`

```bash
cd ./student-list/simple_api
```
 
2) Build the api container image :

```bash
docker build . -t api.student_list.img
docker images
```
> ![1-docker images](https://github.com/SHoward225/mini-projet-docker/blob/master/assets/A.png)

3) Create a bridge-type network for the two containers to be able to contact each other by their names thanks to dns functions :

```bash
docker network create student_list.network --driver=bridge
docker network ls
```
> ![2-docker network ls](https://github.com/SHoward225/mini-projet-docker/blob/master/assets/B.png)


4) Move back to the root dir of the project and run the backend api container with those arguments :

```bash
cd ..
docker run --rm -d --name=api.student_list --network=student_list.network -v ./simple_api/:/data/ api.student_list.img
docker ps
```
> ![3-docker ps](https://github.com/SHoward225/mini-projet-docker/blob/master/assets/C.png)

As you can see, the api backend container is listening to the 5000 port.
This internal port can be reached by another container from the same network so I chose not to expose it.

I also had to mount the `./simple_api/` local directory in the `/data/` internal container directory so the api can use the `student_age.json` list 


> ![4-./simple_api/:/data/](https://github.com/SHoward225/mini-projet-docker/blob/master/assets/D.png)


5) Update the `index.php` file :

You need to update the following line before running the website container to make ***api_ip_or_name*** and ***port*** fit your deployment
   ` $url = 'http://<api_ip_or_name:port>/pozos/api/v1.0/get_student_ages';`

Thanks to our bridge-type network's dns functions, we can easyly use the api container name with the port we saw just before to adapt our website

```bash
sed -i s\<api_ip_or_name:port>\api.student_list:5000\g ./website/index.php
```
> ![5-api.student_list:5000](https://github.com/SHoward225/mini-projet-docker/blob/master/assets/E.png)


6) Run the frontend webapp container :

Username and password are provided in the source code `.simple_api/student_age.py`

> ![6-id/passwd](https://user-images.githubusercontent.com/101605739/224590363-0fdd56ae-9fb9-45e7-8912-64a6789faa9e.png)

```bash
docker run --rm -d --name=webapp.student_list -p 8081:80 --network=student_list.network -v ./website/:/var/www/html -e USERNAME=toto -e PASSWORD=python php:apache
docker ps
```
> ![7-docker ps](https://user-images.githubusercontent.com/101605739/224591443-344fd2cd-ddbc-4780-bbc5-7cc0bdac156f.jpg)


7) Test the api through the frontend :

7a) Using command line :

The next command will ask the frontend container to request the backend api and show you the output back.
The goal is to test both if the api works and if frontend can get the student list from it.

```bash
docker exec webapp.student_list curl -u toto:python -X GET http://api.student_list:5000/pozos/api/v1.0/get_student_ages
```
> ![8-docker exec](https://user-images.githubusercontent.com/101605739/224593842-23c7f3a5-e5bc-4840-a6af-2eda0f622710.png)


7b) Using a web browser `IP:80` :

- If you're running the app into a remote server or a virtual machine (e.g provisionned by eazytraining's vagrant file), please find your ip address typing `hostname -I`
> ![9-hostname -I](https://user-images.githubusercontent.com/101605739/224594393-841a5544-7914-4b4f-91fd-90ce23200156.jpg)

- If you are working on PlayWithDocker, just `open the 80 port` on the gui
- If not, type `localhost:80`

Click the button

> ![10-check webpage](https://user-images.githubusercontent.com/101605739/224594989-0cb5bcb7-d033-4969-a12e-0b2aa9953a97.jpg)


7) Clean the workspace :

Thanks to the `--rm` argument we used while starting our containers, they will be removed as they stop.
Remove the network previously created.


```bash
docker stop api.student_list
docker stop webapp.student_list
docker network rm student_list.network
docker network ls
docker ps
```
> ![11-clean-up](https://user-images.githubusercontent.com/101605739/224595124-3ea15f42-e6d5-462a-92a0-52af7c73c17a.jpg)


## 3) Infrastructure


## 4) Docker-registry and Deployment

As the tests passed we can now 'composerize' our infrastructure by putting the `docker run` parameters in ***infrastructure as code*** format into a `docker-compose.yml` file.

1) Run the application (api + webapp) :

As we've already created the application image, now you just have to run :

```bash
docker-compose up -d
```

Docker-compose permits to chose which container must start first.
The api container will be first as I specified that the webapp `depends_on:` it.
> ![12-depends on](https://user-images.githubusercontent.com/101605739/224595564-e010cc3f-700b-4b3e-9251-904dafbe4067.png)

And the application works :
> ![13-check app](https://github.com/Abdel-had/mini-projet-docker/assets/101605739/2002c41f-6590-4491-b571-65de7ba1457e)


2) Create a registry and its frontend

I used `registry:2` image for the registry, and `joxit/docker-registry-ui:static` for its frontend gui and passed some environment variables :

> ![14-gui registry env var](https://user-images.githubusercontent.com/101605739/224596117-76cda01c-f2f6-4a18-862f-95d56449f98a.png)


E.g we'll be able to delete images from the registry via the gui.

```bash
docker-compose -f docker-compose.registry.yml up -d
```
> ![15-check gui reg](https://github.com/Abdel-had/mini-projet-docker/assets/101605739/99f182f1-bc73-458c-8b29-207a681a31fd)


3) Push an image on the registry and test the gui

You have to rename it before (`:latest` is optional) :

> NB: for this exercise, I have left the credentials in the **.yml** file.

```bash
docker login
docker image tag api.student_list.img:latest localhost:5000/pozos/api.student_list.img:latest
docker images
docker image push localhost:5000/pozos/api.student_list.img:latest
```

> ![16-push image to registry](https://github.com/Abdel-had/mini-projet-docker/assets/101605739/4dbff256-72ae-4c6e-b076-65e705828f28)

> ![17-full reg](https://github.com/Abdel-had/mini-projet-docker/assets/101605739/fbc9cd2b-ec4b-4211-ba26-1a454936b204)

> ![18-full reg details](https://github.com/Abdel-had/mini-projet-docker/assets/101605739/3c89e9cc-f1f4-42f8-bea4-4e3ec1672cbe)


------------


# This concludes my Docker mini-project run report.

Throughout this project, I had the opportunity to create a custom Docker image, configure networks and volumes, and deploy applications using docker-compose. Overall, this project has been a rewarding experience that has allowed me to strengthen my technical skills and gain a better understanding of microservices principles. I am now better equipped to tackle similar projects in the future and contribute to improving containerization and deployment processes within my team and organization.

![octocat](https://myoctocat.com/assets/images/base-octocat.svg) 


