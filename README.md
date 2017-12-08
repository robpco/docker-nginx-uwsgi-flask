# Supported tags and respective `Dockerfile` links

* [`python2.7` _(Dockerfile)_](https://github.com/robertpeteuil/docker-nginx-uwsgi-flask/blob/master/python2.7/Dockerfile)
* [`python3.5` _(Dockerfile)_](https://github.com/robertpeteuil/docker-nginx-uwsgi-flask/blob/master/python3.5/Dockerfile)
* [`python3.6` _(Dockerfile)_](https://github.com/robertpeteuil/docker-nginx-uwsgi-flask/blob/master/python3.6/Dockerfile)

**The `latest` tag is not supported - one of the tags above must be explicitly specified.**
- This is because the tags represent different variants not incremental versions.
- This eliminates importing or pulling an unexpected version

# NGINX-UWSGI-FLASK

Docker image with **Nginx**, **uWSGI** and **Flask** in a single container that enables running Python Flask Apps on NGINX.

*NOTE: This project is a derivative of the project at [tiangolo/UWSGI-NGINX-DOCKER](https://github.com/tiangolo/uwsgi-nginx-flask-docker), and was created ato address an urgent need for fixes and enhancements.*

Implementing the enhancements required creating derivatives of both the [base images](https://github.com/robertpeteuil/docker-nginx-uwsgi) and the [flask images](https://github.com/robertpeteuil/docker-nginx-uwsgi-flask).

**GitHub repo**: <https://github.com/robertpeteuil/docker-nginx-uwsgi-flask>

**Docker Hub image**: <https://hub.docker.com/r/robpco/nginx-uwsgi-flask/>

# Overview

This **Docker** image enables the creation of Python **Flask** Apps that run on **Nginx** by using **uWSGI**.  It also simplifies the task of migrating a Flask-only App to run on Nginx, which is much more scalable for production deployments.

This image adds Flask support, Environment Variables and the ability to generated configuration details during startup to the underlying base image [nginx-uwsgi](https://hub.docker.com/r/robpco/nginx-uwsgi/).

The Docker-Hub [repository](https://hub.docker.com/r/robpco/nginx-uwsgi-flask/) contains auto-generated images from this repo: one for each Python version, and each one of those has a normal and Alpine-based image.  These images can be referenced, or pulled, by using the image name with the tags listed above.

# Enhancements

The image used in this repo includes the following enhancements (over previous repos):
- Adds image variants built on alpine-linux
- Ability to change Nginx listen port with a new LISTEN_PORT environment variable
- Updated built-in Nginx to 1.13.7 on non-alpine variants
- Reduces CRIT errors from `supervisord`
  - `supervisord.conf` is explicitly referenced via the Dockerfile CMD statement
  - `supervisord.conf` includes an explicitly set user-name
- Docker-Hub Image is Automatically re-built when Python updates
- Manually Building the image is protected against failures from key-server outages

# Usage

Basic usage information is provided below.  For full documentation with examples please visit the original repo: [tiangolo/UWSGI-NGINX-DOCKER](https://github.com/tiangolo/uwsgi-nginx-docker).  If you reference the documentation on the original repo, remember to use this image `robpco/nginx-uwsgi-flask` to get the enhancements described above.

**This image is normally as a base image for a project**

This involves three high-level steps:
1. Creating a `Dockerfile` that references this image:tag
2. Building an Image from that `Dockerfile`
3. Running the Image and Testing the App

**STEP 1 - Create a `Dockerfile`**
- In this example, we use the `FROM` line to specify this image and the `python3.6-alpine` variant
- We copy our python scripts, in a sub-directory on the local computer called `app`, to a folder in the container called `/app`.

```Dockerfile
FROM robpco/nginx-uwsgi-flask:python3.6-alpine

COPY ./app /app
```


**STEP 2 - Build an image from the `Dockerfile`**
- Next we use the `docker build` command to create the image, name it `myapp` and tag it as version `1`
```
docker build -t myapp:1 .
```


**STEP 3 - Run the image and viewing the output**
- Now, we can run the image and use a few extra parameters to make things easier
  - The `-p` parameter maps the localhost's port 8080 to port 80 of the image
  - The `-d` parameter `detaches` our terminal session from the image
  - The `--rm` parameter automatically removes the image when it's stopped.
  - Finally, we specify the name `myapp` and tag `1` of the image
- After running the command, we can open up out web-browser and type in `http://localhost:8080` and interact with our Python Flask application

```
docker run --rm -p 8080:80 -d myapp:1
```


## Custom Environment Variables

This image supports the following custom environment variables:

- **UWSGI_INI** - the path and file of the configuration info
  - default: `/app/uwsgi.ini`
- **NGINX_MAX_UPLOAD** - the maximum file upload size allowed by Nginx
  - 0 = unlimited (image default)
  - 1m = normal Nginx default
- **LISTEN_PORT** - custom port that Nginx should listen on
  - 80 = Nginx default

The variables that begin with `STATIC_` allow configuring Nginx to relay "static content" directly without going through uWSGI or Flask.  This is advantageous for basic HTML pages, css and js files, that don't need their output adjusted by your Flask App.

- **STATIC_INDEX** - serve '/' directly from `/app/static/index.html`
  - 0 = disabled (default)
  - 1 = enabled - the file `index.html` located in the `/app/static` directory (in the container) will be forwarded to any requests to the root of your server (`/`) will
- **STATIC_URL** - external URL where requests for static files originate
- **STATIC_PATH** - container location of static files (absolute path)


### Setting Environment Variables

Environment variables can be set in multiple ways.  The following examples, demonstrate setting the `LISTEN_PORT` environment variable via three different methods.  These methods can be applied to any of the Environment Variables.

**Setting in a `Dockerfile`**

```dockerfile
# ... (snip) ...
ENV LISTEN_PORT 8080
# ... (snip) ...
```


**Setting during [`docker run`](https://docs.docker.com/engine/reference/commandline/run/#options) with the `-e` option**

```shell
docker run -e LISTEN_PORT=8080 -p 8080:8080 myimage
```


**Setting in `docker-compose` file using the `environment:` keyword in a `docker-compose` file**

```yml
version: '2.2'
services:
  web:
    image: myapp
  environment:
    LISTEN_PORT: 8080
```


## UPDATES
- 2017-11-29: Added ability to change port Nginx listens on with new environment variable `LISTEN_PORT`.
  - Thanks to github user [tmshn](https://github.com/tmshn)
- 2017-11-29: Alpine variants added
  - Thanks to github user [ProgEsteves](https://github.com/ProgEsteves)
- 2017-11-29: Automatic image re-build when Python updates
- 2017-11-28: Updated Nginx version installed on non-Alpine images


## FIXES
- 2017:11-30: Alpine images - eliminated uWSGI random build failures
- 2017-11-30: Non-Alpine images - limit build failures caused by GPG key validation failing
- 2017-11-29: Alpine required additional changes:
  - Replace default `/etc/nginx/nginx.conf` with an alternate version
  - Create `/run/nginx` directory to stop immediate Nginx crash
- 2017-11-28: Fixed console errors from supervisor process:
  - Added explicit path reference to `supervisord.conf` in Dockerfile `CMD` statement
  - Added explicitly set username in `supervisord.conf`
