<p align="center">
<img src="http://wangjiayu.net/media/upload/2020/04/16/docker-on-raspberrypi.jpg" width="800">
</p>


# Dockerizing Django App with MySQL, Gunicorn and Nginx
### Welcome to visit [my website](http://wangjiayu.net) for more tutorials


## Chapter One

### Preparation

This series of tutorial is aimed to help Docker beginners or people have experience of Docker but have no idea how to build Django project over it and deploy.. 

#### What is Docker?
Docker is a Linux-based container technology, which can package your code and the environment required by the code together, and assemble into a standard, lightweight, and safe isolated environment. Docker is designed to make it easier to create, deploy, and run applications by using containers. Containers allow a developer to package up an application with all of the parts it needs, such as libraries and other dependencies, and deploy it as one package. 

A Docker container can be seen as a computer inside your computer. The cool thing about this virtual computer is that you can send it to your friends; And when they start this computer and run your code they will get exactly the same results as you did.

Build, Ship and Run!

#### Why Docker?
Imagine you are working on an analysis in Python pandas and you send your code to a friend. Your friend runs exactly this code on exactly the same data set but gets a slightly different result. This can have various reasons such as a different operating system, a different version of python environment, et cetera. Docker is trying to solve problems like that.

#### Dev Environment
Although Windows version Docker is an option, the compatibility is not quite good in all aspects (installation is also troublesome), thus it is recommended that Linux or MacOS is used for this tutorial. Once we get the Django project done, it can be run on Windows without problem.

#### Requirement
<img src="https://img.shields.io/badge/Docker-19.03.8-blue" height="20">
<img src="https://img.shields.io/badge/Docker--compose-1.17.1-blue" height="20">
<img src="https://img.shields.io/badge/Python-3.6-yellow" height="20">
<img src="https://img.shields.io/badge/MySQL-5.7-orange" height="20">
<img src="https://img.shields.io/badge/Nginx-1.16.1-green" height="20">
<img src="https://img.shields.io/badge/Gunicorn-20.0.4-brightgreen" height="20">
<img src="https://img.shields.io/badge/Ubuntu%20Server-18.04-red" height="20">


```
$ sudo snap install docker
```

### Setup a virtual enviroment
```
$ sudo pip3 install virtualenv
$ cd ~
$ sudo mkdir -p djangodemo && cd djangodemo
$ sudo virtualenv .
$ source bin/activate
```

### Initiate a Django project
```
$ pip install django // Do not use python3 or sudo
$ django-admin startproject demo
$ cd demo

demo
├── demo
│   ├── asgi.py
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
└── manage.py

```

Now we are going to migrate data
```
$ python manage.py migrate

Running migrations:
  Applying contenttypes.0001_initial... OK
  Applying auth.0001_initial... OK
  Applying admin.0001_initial... OK
  ...
  Applying sessions.0001_initial... OK
  
$ pip freeze > requirements.txt

```

### Run Docker

Pull the hello-world image from Docker Hub and run a container:
```
$ docker run hello-world
```
```
docker : Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
1b930d010525: Pull complete
Digest: sha256:c3b4ada4687bbaa170745b3e4dd8ac3f194ca95b2d0518b417fb47e5879d9b5f
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.
...
```
Once you see "This message shows that your installation appears to be working correctly.", you are good to go.

#### Top  docker commands
```
$ docker images // List local images, you should see hello-world
$ docker ps -a // show all the running and exited containers
$ docker pull  // pull images from the docker repository (hub.docker.com)
$ docker run  // create a container from an image
$ docker exec -it <container id> bash  // access the running container
$ docker stop <container id>  // stop a running containe
$ docker kill <container id>  // kills the container by stopping its execution immediately
$ docker commit <conatainer id> <username/imagename>  // creates a new image of an edited container on the local system
$ docker login  // login to the docker hub repository
$ docker push <username/image name>  // push an image to the docker hub repository
$ docker rm <container id>  // delete a stopped container
$ docker rmi <image-id>  // delete an image from local storage
$ docker build <path to docker file>  // build an image from a specified docker file
```

### Dockerfile
Docker allows you to build an image from a formated text file, called Dockerfile. Let's make a file `Dockerfile` under the root directory of project and put info below in:

```
$ touch Dockerfile
$ vim Dockerfile
```

```dockerfile
# pull Linux environment with python 3.6
FROM python:3.6
# set python environment variable
ENV PYTHONUNBUFFERED 1
# create code folder as set it as workspace
RUN mkdir /code
WORKDIR /code
# upgrade pip
RUN pip install pip -U
# copy requirements.txt into the code folder in container
ADD requirements.txt /code/
# pip install required packages
RUN pip install -r requirements.txt
# copy current folder into the code folder in container
ADD . /code/
```

Now your project folder should like this
```
├── db.sqlite3
├── demo
│   ├── asgi.py
│   ├── __init__.py
│   ├── __pycache__
│   │   ├── __init__.cpython-36.pyc
│   │   ├── settings.cpython-36.pyc
│   │   └── urls.cpython-36.pyc
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
├── Dockerfile
├── manage.py
└── requirements.txt
```

### Docker-compose
In production environment, it is usually not possible to put all the components of a project into the same container. We usually put each independent application (MySQL, Django etc) into a separate container, which is convenient for reuse. Therefore, there may be multiple containers running on the same server, docker-compose then can be adopted for defining and running multi-container Docker applications. With Compose, you use a YAML file to configure your application’s services. Then, with a single command, you create and start all the services from your configuration. Let's first make sure docker-compose is successfully installed:
```
$ docker-compose -v
docker-compose version 1.25.5, build unknown
```

#### docker-compose.yml
Let's create a docker-compose.yml under project root folder
```yml
version: "3"
services:
  app:
    restart: always
    build: .  # point means current folder
    command: "python3 manage.py runserver 0.0.0.0:8000"
    volumes:
      - .:/code
    ports:
      - "8000:8000"
```

Now let's test! 
```
$ docker-compose up
```
You should be able to see this:
```
Creating network "demo_default" with the default driver
Building app
Step 1/8 : FROM python:3.6
3.6: Pulling from library/python
f15005b0235f: Pull complete
41ebfd3d2fd0: Pull complete
b998346ba308: Pull complete
f01ec562c947: Pull complete
2447a2c11907: Pull complete
da3a3213d74f: Pull complete
f651e23d51f3: Pull complete
469e71af0b61: Pull complete
b860a1a9fd72: Pull complete
Digest: sha256:53f66116f7122fe6045f68e14ee5bd82847d5a6f60f659bfc18ac13caf729fc9
Status: Downloaded newer image for python:3.6
 ---> c4f7d42f7b89
Step 2/8 : ENV PYTHONUNBUFFERED 1
 ---> Running in feb6b81a37fd
Removing intermediate container feb6b81a37fd
 ---> 7b477f7f9afb
Step 3/8 : RUN mkdir /code
 ---> Running in 51751a0e29ee
Removing intermediate container 51751a0e29ee
 ---> e9a3c66f35a3
Step 4/8 : WORKDIR /code
 ---> Running in 6897358aa401
Removing intermediate container 6897358aa401
 ---> 0a3790d054c3
Step 5/8 : RUN pip install pip -U
 ---> Running in 996fd7b19aee
Requirement already up-to-date: pip in /usr/local/lib/python3.6/site-packages (20.0.2)
Removing intermediate container 996fd7b19aee
 ---> f136bdd7dba6
Step 6/8 : ADD requirements.txt /code/
 ---> b36f035d1189
Step 7/8 : RUN pip install -r requirements.txt
 ---> Running in 545ff3d2e662
Collecting asgiref==3.2.7
  Downloading asgiref-3.2.7-py2.py3-none-any.whl (19 kB)
Collecting Django==3.0.5
  Downloading Django-3.0.5-py3-none-any.whl (7.5 MB)
Collecting pytz==2019.3
  Downloading pytz-2019.3-py2.py3-none-any.whl (509 kB)
Collecting sqlparse==0.3.1
  Downloading sqlparse-0.3.1-py2.py3-none-any.whl (40 kB)
Installing collected packages: asgiref, sqlparse, pytz, Django
Successfully installed Django-3.0.5 asgiref-3.2.7 pytz-2019.3 sqlparse-0.3.1
Removing intermediate container 545ff3d2e662
 ---> 7bb65960ffd8
Step 8/8 : ADD . /code/
 ---> acb253211c8f
Successfully built acb253211c8f
Successfully tagged demo_app:latest
WARNING: Image for service app was built because it did not already exist. To rebuild this image you must use `docker-compose build` or `docker-compose up --build`.
Creating demo_app_1 ... done
Attaching to demo_app_1
app_1  | Watching for file changes with StatReloader
app_1  | Performing system checks...
app_1  | 
app_1  | System check identified no issues (0 silenced).
app_1  | April 16, 2020 - 05:40:47
app_1  | Django version 3.0.5, using settings 'demo.settings'
app_1  | Starting development server at http://0.0.0.0:8000/
app_1  | Quit the server with CONTROL-C.
app_1  | [16/Apr/2020 05:41:46] "GET / HTTP/1.1" 200 16351
app_1  | [16/Apr/2020 05:41:46] "GET /static/admin/fonts/Roboto-Bold-webfont.woff HTTP/1.1" 200 86184
app_1  | [16/Apr/2020 05:41:46] "GET /static/admin/fonts/Roboto-Regular-webfont.woff HTTP/1.1" 200 85876
app_1  | [16/Apr/2020 05:41:46] "GET /static/admin/fonts/Roboto-Light-webfont.woff HTTP/1.1" 200 85692
app_1  | Not Found: /favicon.ico
app_1  | [16/Apr/2020 05:41:46] "GET /favicon.ico HTTP/1.1" 404 1970
```
<p align="center">
<img src="http://wangjiayu.net/media/upload/2020/04/16/peek-2020-04-16-00-39.gif" width="700">
</p>
<p align="center">
<img src="http://wangjiayu.net/media/upload/2020/04/16/peek-2020-04-16-00-40.gif" width="700">
</p>

Now open your crowser, type `127.0.0.1:8000` and see if you can see the expected startup page, with its little rocket and message.
<p align="center">
<img src="http://wangjiayu.net/media/upload/2020/04/16/screenshot_7.png" width="700">
</p>

OK~ `Ctrl + C` to stop the running app, however the container is still there, just not in running, to delete the container, use:
```
$ docker-compose down
```
To run the container in the background:
```
$ docker-compose up -d
```
To rebuild the image:
```
$ docker-compose build
```
To start and stop existed container:
```
$ docker-compose start
$ docker-compose stop
```

## Chapter Two

In last [Chapter](http://wangjiayu.net/article/article-detail/31/) we successfully built a Django project in docker container, but the database we used was SQLite, which is not recommended in production. To be clear, I don't agree with the saying that "you can’t use SQLite in production", data of this website is stored in SQLite, and it's quite stable, the data reading speed or efficiency is never a problem, SQLite is not directly comparable to client/server SQL database engines such as MySQL, Oracle, PostgreSQL, or SQL Server since SQLite competes with fopen(). In this chapter, we are going to talk about setting up Django on MySQL in Docker containers.

If you have MySQL installed on host machine and running on 3306 port, make sure to stop the service first:
```
$ sudo /etc/init.d/mysql stop
// you can later run 
$ sudo /etc/init.d/mysql start
// to restart the service
```

### Docker-compose
The idea of object-oriented programming tended to separate modules with independent functions to facilitate reuse and maintenance. Container is the same. Although it is theoretically possible to cram all components into the same container, it is better to keep the modules in separate containers as long as the communication between them is well maintained. That is to say, we are going to see two containers in this Chapter:
`app`: container for Django
`db`: container for MySQL

First, we edit the `docker-compose.yml` created in previous chapter.
```yml
version: "3"
services:
  app:
    restart: always
    build: .
    command: bash -c "python3 manage.py migrate && python3 manage.py runserver 0.0.0.0:8000"
    volumes:
      - .:/code
    ports:
      - "8000:8000"
    depends_on:
      - db
  db:
    image: mysql:5.7
    volumes:
      - "./mysql:/var/lib/mysql"
    ports:
      - "3306:3306"
    restart: always
    environment:
      - MYSQL_ROOT_PASSWORD=P4S5WoRd
      - MYSQL_DATABASE=dockerDemo
```
depends_on does not wait for `db` to be “ready” before starting web only until they have been started. 

Why do we use volumes here? Can we just keep data in a container and keep it isolated? Keeping the data in the container is theoretically possible, but there is a fatal problem because the data is linked to the life cycle of the container, imagine what if  the container is deleted one day, the data in it will follow the wind and pass away with it, you'll then become the legendary figure in your company. Be aware that the life cycle of the container may be very short, and the delete instruction is also quite smooth (just typing `docker-compose down` and it's gone). Thus we map the data to the localhost machine, even if the container is deleted, the data is still lying safely on your server. In other words, container is suitable for running stateless applications; when it comes to stateful things like data, you must think about it carefully.

In our case, you'll later find a folder called "mysql" created under the project root folder, that's where we are going to store the data thru MySQL container, this has nothing do to with your local MySQL, even you do not have it installed on host machine. `- "./mysql:/var/lib/mysql"`means we expose a local folder `./mysql` to MySQL docker container for data storage, just like how `ports` works.

### Dockerfile
Now we edit the Dockerfile.
```
FROM python:3.6
ENV PYTHONUNBUFFERED 1

# Add these two lines
RUN apt-get update
RUN apt-get install python3-dev default-libmysqlclient-dev -y

RUN mkdir /code
WORKDIR /code
RUN pip install pip -U
ADD requirements.txt /code/
RUN pip install -r requirements.txt
ADD . /code/
```

Activate your local virtual environment, 
```
$ source bin/activate
$ cd demo
$ pip install mysqlclient
$ pip freeze > requirements.txt
```
You'll see `mysqlclient==1.4.6` or whatever version of `mysqlclient` was just added.

### Django Settings
Now let's modify database connection information in Django settings.py
```python
ALLOWED_HOSTS = ['*']
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'dockerDemo',
        'USER': 'root',
        'PASSWORD': 'P4S5WoRd',
        'HOST': 'db', # Not localhost
        'PORT': '3306',
        'OPTIONS': {'charset': 'utf8mb4'},
    }
}
```

### Go For Launch
```
$ sudo docker-compose build
$ sudo docker-compose up
```
<p align="center">
<img src="http://wangjiayu.net/media/upload/2020/04/16/peek-2020-04-16-21-03.gif" width="700">
</p>

Believe or not, you can even edit your code without turn it off, docker and django will hot watch any of your update and automatically apply it on your browser, it is cool, isn't it?
*Develop in contain environment is not recommended.*


## Chapter Three

In last [Chapter](http://wangjiayu.net/article/article-detail/34/) we successfully added MySQL to Django and docker container, but the code was still running locally and in development environment which is not stable for production use, in this chapter, we are going to wrap up our Django project and deploy on server with Docker + Django + MySQL + Nginx + Gunicorn.

### Local setting

#### Install gunicorn
Activate your local virtual environment, 
```
$ source bin/activate
$ cd demo
$ pip install gunicorn
$ pip freeze > requirements.txt
```
You'll see `gunicorn==19.9.0` or whatever version of `gunicorn` was just added.

####  Docker-compose
Before deploying on server, let's do it locally. Edit the `docker-compose.yml` we previously worked on.
```yml
version: "3"

services:
  app:
    restart: always
    build: .
    command: bash -c "python3 manage.py collectstatic --no-input && python3 manage.py migrate && gunicorn --timeout=30 --workers=4 --bind :8000 demo.wsgi:application"
    volumes:
      - .:/code
      - static-volume:/code/collected_static
    expose:
      - "8000"
    depends_on:
      - db
    networks:
      - web_network
      - db_network
  db:
    image: mysql:5.7
    volumes:
      - "./mysql:/var/lib/mysql"
    ports:
      - "3306:3306"
    restart: always
    environment:
      - MYSQL_ROOT_PASSWORD=mypassword
      - MYSQL_DATABASE=dockerDemo
    networks:
      - db_network
  nginx:
    restart: always
    image: nginx:latest
    ports:
      - "8000:8000"
    volumes:
      - static-volume:/code/collected_static
      - ./config/nginx:/etc/nginx/conf.d
    depends_on:
      - app
    networks:
      - web_network

networks:
  web_network:
    driver: bridge
  db_network:
    driver: bridge

volumes:
  static-volume:
```
Looks a little bit complicated isn't it? Let's take a closer look at it. 
- Here we have three containers defined - `app`, `db` and `nginx`. The containers communicate between each other through defined ports;
- Two networks are defined - `web_network` and `db_network`. Only containers within the same network can communicate with each other. Networks are isolated to thers, and they cannot communicate each other even if sharing the same port; 
- One data volume is defined, called `static-volume`. The data volume is very suitable for multiple containers to share data, as you have found out both `app` and `nginx` are using it.
- Both `expose` and `ports` can expose the ports of the container. The difference is that `expose` only exposes to other containers, while `ports` exposes to other containers and the host machine..

#### Network
Docker allows users to define the working network for each container, only contains with in the same network can communicate. In our case, the `nginx` container is in the `web_network` network, and the `db` container is in the `db_network` network, so these two cannot communicate, and in fact they do not need to communicate as well. The `app` container is in both `web_network` and `db_network` network, which works as a bridge to connecting these three containers.

Defining the network can isolate the network environment of the container, and it is convenient for O & M personnel to see the logical relationship of the network at a glance.

#### Volume
The volume that we have seen in previous chapters for mapping host and container directories is actually a type of the mount, the newly created static-volume in this case is called a volume. 
```
static-volume: / code / collected_static. 
```
Pathlink after the colon is directory in the container, the `static-volume` before colon is not the host directory, but the name of the volumn. Essentially, the volumn also implements the mapping dictory between host machine and the container, but the volumn is managed by Docker, and you don’t even need to know the specific location where the volume is stored on the host machine.

Compared with mounting, the advantage of volumn is that it is managed by Docker, there is no mounting problem potentially causing by insufficient permissions, and there is no need to specify different paths on different servers; the disadvantage is that it is not suitable for mapping single configuration files.

Like the mounting, the life cycle of volumn is separated from the container, and the volumn still exist after the container is deleted. You can continue to use it next time building an image.

The volumn has a very important feature:
- when the container is started, if an empty volume is mounted to a non-empty directory in the container, the files in this directory will be copied to the volumn; 
- If you mount a non-empty volumn to a directory in the container, the data in the volumn will be displayed in the directory in the container; 
- if there is data in the directory in the original container, then the original data will be hidden.

In other words, as long as the volume is initialized, the original collected_static directory of the container will be hidden and no longer used, and the newly added files only exist in the volume, not in the container.

In addition, the permanent storage of static static files (and media files) in Django project can be achieved by mounting or volume.

#### Local nginx setting
Let's modify the local Nginx configuration file (not on server, create a file with `.conf` extension, name whatever you want or even just use the `default` file), we are going to map the local nginx to the `nginx` container, expose port 8000 of host machine to port 8000 of the nginx container.
```nginx
upstream app {
  ip_hash;
  server app:8000;
}

server {
  listen 8000;
  server_name localhost;

  location /static/ {
    autoindex on;
    alias /code/collected_static/;
  }

  location / {
    proxy_pass http://app/; // Reverse proxy
  }
}
```

#### Modify Django settings.py
```python
STATIC_ROOT = os.path.join(BASE_DIR, 'collected_static')
STATIC_URL = '/static/'
```

#### Go for launch
```
$ sudo docker-compose up
```
Visit `127.0.0.1:8000` on your browser, ta-da! A little rocket!

### Deploy on server

Follow what we have done locally, setup the docker, docker-compose and python3 on server, clone the Django project to server with Git, modify the settings.py, /etc/nginx/whatEverName.conf (create this file or use the `default` one), and load requirements.txt, docker-compose.yml and Dockerfile onto server in the same directery structure. Here we need to modify something in `docker-compose.yml` because the default http port is 80:
```
version: "3"

services:
  app:
    ...
    command: bash -c "... demo.wsgi:application"
    ...
  db:
    ...
  nginx:
    ...
    ports:
      - "80:8000"
    ...

networks:
  ...

volumes:
  ...
```

/etc/nginx/whatEverName.conf
```nginx
upstream domain_name {
  ip_hash;
  server app:8000;
}

server {
  ...

  location / {
    proxy_pass http://domain_name/;
  }
}
```
Last but not least, modify the Django settings.py
```python
DEBUG=False
```

```
$ sudo docker-compose build
$ sudo docker-compose up
```
Your Django project is deployed on the internet!


