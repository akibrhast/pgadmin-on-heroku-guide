# pgadmin-on-heroku-guide

The following are a list of instruction on how to run a pgadmin instance on heroku to browse a PostgreSQL database. The PostgreSQL database in this example is also running on another instance of heroku. The instruction assumes you have the following things already installed and configured:
1. Heroku CLI
2. Git
3. Docker

It has been compiled using the various steps from the following documentation/links:
1. [Official pgAdmin Container Deployment](https://www.pgadmin.org/docs/pgadmin4/latest/container_deployment.html) 
2. [Official Container Registry & Runtime (Docker Deploys)](https://devcenter.heroku.com/articles/container-registry-and-runtime#pushing-an-existing-image)
3. [Root Password Inside a Docker Container](https://stackoverflow.com/questions/28721699/root-password-inside-a-docker-container)
4. [Docker Commit Refference](https://docs.docker.com/engine/reference/commandline/commit/)

To start with we are going to need to obtain the official pgAdmin container image from docker hub, the official image for pgAdmin can be found [here](#https://hub.docker.com/r/dpage/pgadmin4). We do that by running :
```
docker pull dpage/pgadmin4
```

Next we are going to create a container from the image, run the following command in your terminal:
```
docker run --name pgAdminTestContainer -p 80:80 -e 'PGADMIN_DEFAULT_EMAIL=user@domain.com' -e PGADMIN_DEFAULT_PASSWORD=SuperSecret' -d dpage/pgadmin4
```
Dont worry about setting a proper email and password at the momment as those will be configured later via heroku config vars.

After running the above command navigate to `localhost:80` on your browser and log in using the credentials you have set above:

>*username*: user@domain.com

>*password*: SuperSecret

Upon navigating to `localhost:80`, and if everything has been correctly set up you will see this:
![pgAdmin Login Page](https://github.com/akibrhast/pgadmin-on-heroku-guide/blob/master/pgAdmin_login_page.png?raw=true)
and upon successfull login you will see this:
![pgAdmin Landing Page](https://github.com/akibrhast/pgadmin-on-heroku-guide/blob/master/pgAdmin_initial_landing_page.png?raw=true)

The next thing we need to do is access the running container as root to modify some files in it. We can do that by running the following command:
```
docker exec -u 0 -it <container_name> /bin/sh
```
If you have been following the instructions word for word here you should run:
```
docker exec -u 0 -it  pgAdminTestContainer /bin/sh
```

You will be logged in as the pgadmin user. We are then going to set/change the password for the pgadmin user as well as the root user. Run `passwd` and set up a new password. Then run the command `su root` to switch to root user followed by the `passwd` command again to set up a new password for root. 


Now while you are still a root user navigate to the root directory. Here you will see a file called `entrypoint.sh`. Give yourself write permission to the file by running:
```
chmod r+w entypoint.sh
```
and then run:
```
vi entrypoint.sh
```
to edit the file.

Inside the `entrypoint.sh` file scroll all the way to the bottom of the file using arrow keys. Until you see `${PGADMIN_LISTEN_PORT:-80}`

Replace `80` with $PORT like so : `${PGADMIN_LISTEN_PORT:-$PORT}`

Save and quit from `entrypoint.sh`, and exit your container

Now we are going to make an image from the running container using the following command: 
```
docker commit <container_name>
```
example if you have been following word-for-word:
```
docker commit pgAdminTestContainer
```

You can confirm that a new image has been created by running:
```
docker image ls -a
```
You will see a new image with REPOSITORY as `<none>` and TAG as `<none></none>`

The next few steps involve setting things up for heroku.
Create an empty app on heroku and name it anything, for example *redfoo*
- Navigate to settings on the newly created app and add the following configvars
![Heroku Configs](https://raw.githubusercontent.com/akibrhast/pgadmin-on-heroku-guide/master/Heroku_Config_Vars.png)

where:
>PGADMIN_DEFAULT_EMAIL : your_email

>PGADMIN_DEFAULT_PASSWORD: your_password

>PGADMIN_LISTEN_ADDRESS: 0.0.0.0

Next we will follow some of the instuction from this [heroku guide](https://devcenter.heroku.com/articles/container-registry-and-runtime#pushing-an-existing-image) to push the image to heroku

First run:
```
docker tag <image_id> registry.heroku.com/<app>/<process-type>
```
where `<image_id>` is the id of your newly created image, `<app>` would be called `redfoo`(if that is the name of your heroku app) and `<process-type>` would be web, example: 
```
docker tag cc792ced279b registry.heroku.com/redfoo/web
```
The follow up with this:
```
docker push registry.heroku.com/<app>/<process-type>
```
example: 
```
docker push registry.heroku.com/redfoo/web
```
Finally run:

```
heroku container:release web
```
