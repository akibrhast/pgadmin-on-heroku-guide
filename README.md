# pgadmin-on-heroku-guide
1. `docker pull dpage/pgadmin4`
2. `docker run -p 80:80 -e 'PGADMIN_DEFAULT_EMAIL=user@domain.com' -e 'PGADMIN_DEFAULT_PASSWORD=SuperSecret' -d dpage/pgadmin4`
3. `docker exec -u 0 -it <container_name> /bin/sh`, from [stackoverflow](https://stackoverflow.com/questions/28721699/root-password-inside-a-docker-container)
4. Set password for root user and pgadmin user
5. type `passwd` to change user password for the pgadmin user
6. Then switch to the root user by typing `su root`
7. Confirm you are root user by typing `whoami`
8. Change the password of root user by typing `passwd`
9. `cd` to root directory as root user and then run `chmod r+w entypoint.sh`
10. Then run `vi entrypoint.sh`
11. Scroll down all the way to bottom of the file using your arrow keys until you see `${PGADMIN_LISTEN_PORT:-80}`
12. Replace 80 with $PORT like so : `${PGADMIN_LISTEN_PORT:-$PORT}`
13. Save and quit from entrypoint.sh
14. Exit your container
15. Now run `docker commit <container_name>` [Docker Commit Refference](https://docs.docker.com/engine/reference/commandline/commit/)
16. This will create a new image from the <container_name>
17. Run `docker image ls -a` to confirm and see that new image has been created
18. Create an empty app on heroku and name it anything, for example 'redfoo'
    - Navigate to settings on the newly created app and add the following configvars
    - ![Heroku Configs](https://raw.githubusercontent.com/akibrhast/pgadmin-on-heroku-guide/master/Heroku_Config_Vars.png)
19. Next we will follow some of the instuction from this [heroku guide](https://devcenter.heroku.com/articles/container-registry-and-runtime#pushing-an-existing-image) to push the image to heroku
20. Run `docker tag <image_id> registry.heroku.com/<app>/<process-type>`, where <image_id> is the id of your newly created image, <app> would be called redfoo and <process-type> would be web, example,  `docker tag cc792ced279b registry.heroku.com/redfoo/web`
21. Run `docker push registry.heroku.com/<app>/<process-type>`, example ,`docker push registry.heroku.com/redfoo/web`
22. Then run `heroku container:release web`
