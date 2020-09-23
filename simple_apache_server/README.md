# Your First Container
For this example we are going to be running a basic apache(httpd) webserver. Additionally, we are going to be showing
off how we can add local files into the Docker container. 

To build the container you can run:

```commandline
docker build -t webserver:test .
```

In that command we are calling `build` and passing in the `-t` flag which means tag. We are saying build this image
and tag is as `webserver:test`.

After the image is built we can check to see if the image exist by issuing:

```commandline
docker images
```

This command will show us all the current images we have on our machine. Not only will you see our `webserver:test` image
but you will see `httpd:2.4` which was the image we had in our `FROM` statement in our Dockerfile. The `FROM` statement
will pull down any images we currently do not have on our local system.


To run the container we can issue this command:

```commandline
docker run -p 8080:80 webserver:test
```

In this command we see that we have a flag, `-p`. This flag is telling docker to map the containers port of `80` to our 
computers port of `8080`. We need to pass in the port flag to allow us to locally access the webserver. If we do not pass
in the flag, we will not be able to access the server.


To run the container in the background we need to add another flag to the container:

```commandline
docker run -d -p 8080:80 webserver:test
```

The flag, `-d`, allows you to run your container in the background. This is helpful since when running the a container
in the foreground if you try to type or try to exit the tab it will kill your container.

Another useful thing about running your container in the background is being able to exec into the container to troubleshoot
or debug things. To exec into the container we first need to check for our running containers. We do do that will the 
following command:

```commandline
docker ps
```

This command will show us all running containers. If you want to see all containers, even ones that are stopped/dead, use
`docker ps --all`.

Using these commands we can see some container running. For example here is my output:

```
CONTAINER ID        IMAGE               COMMAND              CREATED             STATUS              PORTS                  NAMES
e3012f879ced        test:html           "httpd-foreground"   6 seconds ago       Up 3 seconds        0.0.0.0:8080->80/tcp   musing_lichterman
```

You can see this is the container I just launched. Knowing that I can take the container ID and issues the following 
command:

```commandline
docker exec -it e3012f879ced /bin/bash
```

Breaking down this command, we are pass in some flags that allow us to have an interactive terminal with out container.
Lastly, we are passing in the command `/bin/bash`, which launches a shell on our container. Once you issue this command 
you should be inside your container. You can poke around the files and do some basic troubleshooting.

Please note that command `/bin/bash` might not work on all containers due to the operating system they are based on. Alpine
containers do not have `bash` by default. They have a terminal called `ash`. 



Lastly, lets touch some docker volumes. Lets say we want out HTML pages accessible to use easily on our host machine. You always
could exec into your containers to change files but once that container goes down those changes will be gone. So using a docker
volume will help with persistent storage. 

If we take a look at our simple webserver Dockerfile it looks like this:

```Dockerfile
# Pulling from the httpd image and pulling the specific tag of 2.4
FROM httpd:2.4

# Copying a file on my local machine into the file system of the docker container
COPY index.html /usr/local/apache2/htdocs/
```

Now we can see we are copying our `index.html` into our docker container at this file path, `/usr/local/apache2/htdocs/`. 
How did I know this was the folder path to put HTML files? By checking out the DockerHub page, https://hub.docker.com/_/httpd.
DockerHub pages usually contain great pieces of information. 

So we know where we want to to mount our volume inside at, `/usr/local/apache2/htdocs/` , because that is where the HTML files live.
Now that we know that we can make a bind mount to say, "hey this folder on my computer should be connected to this spot in the container".

We can do this by issues this command when running the container:

```commandline
docker run -d -p 8080:80 -v PATH_TO_YOUR_FOLDER_ON_YOUR_COMPUTER:/usr/local/apache2/htdocs/ webserver:test
```

In my case `PATH_TO_YOUR_FOLDER_ON_YOUR_COMPUTER` is `/Users/jim/demo/docker_demo/simple_apache_server/pages`. So the command
looks something like:

```commandline
docker run -d -p 8080:80 -v /Users/jim/demo/docker_demo/simple_apache_server/pages:/usr/local/apache2/htdocs/ webserver:test
```

Now your webserver should be running in the background and you can now edit your pages on your local computer. Which will
be displayed on your webserver!

To kill your webserver find its container ID, by issuing `docker ps`. With the ID then issue the following command:

```commandline
docker kill YOUR_CONTAINER_ID
```
