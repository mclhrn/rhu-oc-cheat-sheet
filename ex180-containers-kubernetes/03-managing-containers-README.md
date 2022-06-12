## Summary
* Podman has subcommands to: create a new container (run), delete a container (rm), list
containers (ps), stop a container (stop), and start a process in a container (exec).
* Default container storage is ephemeral, meaning its contents are not present after the container
is removed.
* Containers can use a folder from the host file system to work with persistent data.
* Podman mounts volumes in a container with the -v option in the podman run command.
* The podman exec command starts an additional process inside a running container.
* Podman maps local ports to container ports by using the -p option in the run subcommand.

```
# bind mount a volume into the container
podman run -v /home/student/dbfiles:/var/lib/mysql rhmap47/mysql

# add selinux context to host dir
sudo semanage fcontext -a -t container_file_t '/home/student/local/mysql(/.*)?'

# apply the seLinux policy
sudo restorecon -R /home/student/local/mysql

# verify context
ls -ldZ /home/student/local/mysql

# change ownership of dir
podman unshare chown 27:27 /home/student/local/mysql

# create a container specifying the mount point
podman run \
 --name persist-db \
 -d \
 -v /home/student/local/mysql:/var/lib/mysql/data \
 -e MYSQL_USER=user1 
 -e MYSQL_PASSWORD=mypa55 \
 -e MYSQL_DATABASE=items \
 -e MYSQL_ROOT_PASSWORD=r00tpa55 registry.redhat.io/rhel8/mysql-80:1

# format running containers
podman ps --format="{{.ID}} {{.Names}} {{.Status}}"

# port forward to expose external access
podman run -d --name apache1 -p 8080:8080 registry.redhat.io/rhel8/httpd-24

# port forward to expose external access bound to ip
podman run -d --name apache2 -p 127.0.0.1:8081:8080 registry.redhat.io/rhel8/httpd-24

# leave blank to get port auto assigned
podman run -d --name apache3 -p 127.0.0.1::8080 registry.redhat.io/rhel8/httpd-24

# view assigned port
podman port apache3

# load sql data
mysql -uuser1 -h 127.0.0.1 -pmypa55 -P13306 items < ./db.sql

# output sql data 
mysql -uuser1 -h 127.0.0.1 -pmypa55 -P13306 items -e "SELECT * FROM Item"

# graceful stop
podman stop mysql-1

# save list of running containers
podman ps -a > /tmp/my-containers

# access the bash shell inside the container
podman exec -it mysql-2 /bin/bash

# inside conatiner, connect to sql
mysql -uroot
use items;
show tables;
SELECT * FROM Item;

# using port forwarding, insert a new row into the Item table
mysql -uuser1 -h 127.0.0.1 -pmypa55 -P13306 items
insert into Item (description, done) values ('Finished lab', 1);

# remove container
podman rm mysql-1 
```


