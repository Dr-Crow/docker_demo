# Docker-Compose and Full Stack Application Deployment
When deploying our full stack application you might have a couple different components like the database, backend and
frontend. Running each of these services in containers is a great way to deploy your application, but how do we get
the containers to talk together other? Plus is there away we can manage all 3 of them at once?  

Thats where Docker-Compose comes in. Docker-Compose allows you to deploy multiple containers in one go. This makes managing
your application alot easier. Plus Docker-Compose also has some additional features to make your life easier.

Lets take a look at an example of a Docker-Compose file that deploys a Wordpress site and a MySQL database.

```
version: '3.3'
services:
   db:
     image: mysql:5.7
     volumes:
       - /Users/jim/demo/docker_demo/docker_compose_full_stack/db_data:/var/lib/mysql
     restart: always
     environment:
       MYSQL_ROOT_PASSWORD: somewordpress
       MYSQL_DATABASE: wordpress
       MYSQL_USER: wordpress
       MYSQL_PASSWORD: wordpress

   wordpress:
     depends_on:
       - db
     image: wordpress:5.5
     ports:
       - "8090:80"
     restart: always
     environment:
       WORDPRESS_DB_HOST: db:3306
       WORDPRESS_DB_USER: wordpress
       WORDPRESS_DB_PASSWORD: wordpress
       WORDPRESS_DB_NAME: wordpress
``` 

Now this might seem like a lot so lets break it down and look at one service(container) first:

```
   db:
     image: mysql:5.7
     volumes:
       - /Users/jim/demo/docker_demo/docker_compose_full_stack/db_data:/var/lib/mysql
     restart: always
     environment:
       MYSQL_ROOT_PASSWORD: somewordpress
       MYSQL_DATABASE: wordpress
       MYSQL_USER: wordpress
       MYSQL_PASSWORD: wordpress
```

Here we can see a couple things. First this is our database service. In this example we are using `mysql` but other database
image exist. So look around to find the one that fits your needs.

Second we might see a familiar term, `volume`. In our last tutorial we mount the `httpd`'s default location for HTML pages
so we could edit them on the fly. Here we have a database which we want data to persist through out our deployments. Here
we are telling Docker to mount out local computer path, `/Users/jim/demo/docker_demo/docker_compose_full_stack/db_data`, to 
the containers file path of `/var/lib/mysql`. `/var/lib/mysql` just happens to be where `mysql` stores is data for its 
databases. With this volume mount we are able to persist data through out multiple restart of our application.

The next line we see is `restart: always`. Docker-Compose has a keyword called `restart`. Restart controls in what conditions
the container should restart if something happens. In this instance we are telling Docker to restart the container always.
This will help ensure the database is always up since your wordpress site depends on it.

Next we have a block called `enviroment`. Environment controls what environment variables are set inside the container.
Environment variables are usually how we pass in passwords, usernames and other configuration data. Be careful about pushing
passwords up to GitHub where things are public. There are alternative ways to deal with environment variables in Docker-Compose 
but that is out of scope for this tutorial. 

Lets take a look at our second service:

```
   wordpress:
     depends_on:
       - db
     image: wordpress:5.5
     ports:
       - "8090:80"
     restart: always
     environment:
       WORDPRESS_DB_HOST: db:3306
       WORDPRESS_DB_USER: wordpress
       WORDPRESS_DB_PASSWORD: wordpress
       WORDPRESS_DB_NAME: wordpress
```

Something new we see is the `depends_on` line. This is telling Docker that this container needs to have the `db` container
up before starting this container. Be careful as this does not mean the database is ready. A lot of times a database container
might be up but not ready for connections. In this example you might see the Wordpress server trying to connect to `mysql` but 
the connect is refused. This is because the `mysql` container is still being setup.

Next you will see the `ports` line. This should be familiar since we used similar syntax in our apache webserver container.
We are letting Docker to map the internal port `80` to our computers port of `8090`. Port `8090` is the port we will be able
to access our server on.

Lastly we have the environment block. It is mostly the same as the block we saw in our `mysql` section. But one important part
is the `WORDPRESS_DB_HOST: db:3306`. This line is telling Wordpress the IP and port of our database. The interesting part
is the IP section just says `db` and not an typically IP. This is because Docker-Compose has create a network for our 
containers to talk to each other. `db` refers to the name of our database service which will point to our `mysql` instance.
This is super helpful since we do not expose our database to the entire network.


Now that we covered what the `docker-compose.yml` file looks like, lets talk about how to run it. Running Docker-Compose is pretty simple, 
the command is:

```commandline
docker-compose up
```

This will spin up both containers in the foreground. Just like running containers in the foreground, it is not that helpful. 
Thus we can use this command to run our containers in the background: 

```commandline
docker-compose up -d
```

Please note when running these commands, you need to be in the same directory as your `docker-compose.yml` file.


To bring down your Docker-Compose use the following command:

```commandline
docker-compose down
```

Docker-Compose can get more complex but hopefully this will get you started!