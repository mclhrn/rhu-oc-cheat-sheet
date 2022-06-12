## Summary
* A Containerfile contains instructions that specify how to construct a container image.
* Container images provided by Red Hat Container Catalog or Quay.io are a good starting point for creating custom images for a specific language or technology.
* Building an image from a Container file is a three-step process:
  1. Create a working directory.
  2. Specify the build instructions in a Containerfile file.
  3. Build the image with the podman build command.
* The Source-to-Image (S2I) process provides an alternative to Containerfiles. S2I implements a standardized container image build process for common technologies from application source code. This allows developers to focus on application development and not Containerfile development.

```
# basic containerfile

# This is a comment line
FROM ubi8/ubi:8.5
LABEL description="This is a custom httpd container image"
MAINTAINER John Doe <jdoe@xyz.com>
RUN yum install -y httpd
EXPOSE 80
ENV LogLevel "info"
ADD http://someserver.com/filename.pdf /var/www/html
COPY ./src/ /var/www/html/
USER apache
ENTRYPOINT ["/usr/sbin/httpd"]
CMD ["-D", "FOREGROUND"]

# another simple example
FROM ubi8/ubi:8.5
MAINTAINER Your Name <_youremail_>
LABEL description="A custom Apache container based on UBI 8"
RUN yum install -y httpd && \
    yum clean all
RUN echo "Hello from Containerfile" > /var/www/html/index.html
EXPOSE 80
CMD ["httpd", "-D", "FOREGROUND"]
```
