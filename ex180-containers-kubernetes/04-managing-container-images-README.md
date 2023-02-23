## Summary
* The Red Hat Container Catalog provides tested and certified images at registry.redhat.io.
* Podman can interact with remote container registries to search, pull, and push container images.
* Image tags are a mechanism to support multiple releases of a container image.
* Podman provides commands to manage container images both in local storage and as compressed files.
* Use the podman commit command to create an image from a container.

```
# save to tar
podman save [-o FILE_NAME] IMAGE_NAME[:TAG]
podman save -o mysql.tar registry.redhat.io/rhel8/mysql-80

# restore from tar
podman load [-i FILE_NAME]

# delete an image
podman rmi [OPTIONS] IMAGE [IMAGE...]

# delete all imges
podman rmi -a

# tag image
podman tag mysql-custom devops/mysql

# push to registry
podman push quay.io/bitnami/nginx

# run html container
podman run -d --name official-httpd -p 8180:80 quay.io/redhattraining/httpd-parent

# podman open shell to running container
podman exec -it official-httpd /bin/bash

# modify html in container
echo "Whatever" > /var/www/html/whatever.html

# test change
curl 127.0.0.1:8180/whatever.html

# diff the change
podman diff official-httpd

# stop container
podman stop official-httpd

# commit changes to image
podman commit  -a 'My Name' official-httpd do180-custom-httpd

# tag changes to my quay
podman tag do180-custom-httpd quay.io/mhearne/do180-custom-httpd:v1.0

# push image to user quay
podman push quay.io/mhearne/do180-custom-httpd:v1.0

# pull image to confirm
podman pull -q quay.io/mhearne/do180-custom-httpd:v1.0
```

