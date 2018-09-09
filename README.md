# Supported tags and respective `Dockerfile` links

- `2.7`, `py2.7`, `python2.7` [_(python2.7/Dockerfile)_](https://github.com/robpco/docker-nginx-uwsgi-flask/blob/master/python2.7/Dockerfile)
- `3.6`, `py3.6`, `python3.6` [_(python3.6/Dockerfile)_](https://github.com/robpco/docker-nginx-uwsgi-flask/blob/master/python3.6/Dockerfile)

**You must explicitly use one of the tags above.**  The `latest` tag is not assigned since each tag represents a different variant, not an incremental version.

## NGINX-UWSGI-FLASK

Docker image with **Nginx**, **uWSGI** and **Flask** in a single container that enables running Python Flask Apps on NGINX.

**GitHub repo**: <https://github.com/robpco/docker-nginx-uwsgi-flask>

**Docker Hub image**: <https://hub.docker.com/r/robpco/nginx-uwsgi-flask/>

## Overview

This **Docker** image enables Python **Flask** Apps to run on **Nginx** using **uWSGI**.  It simplifies the task of migrating pure Flask Web Apps to Nginx-based Web Apps, which desirable for production deployment scenarios.

This image builds on the [nginx-uwsgi](https://hub.docker.com/r/robpco/nginx-uwsgi/) base image and adds Flask support and additional Environment Variables to enable customization.

This repo auto-generates images to [Docker-Hub](https://hub.docker.com/r/robpco/nginx-uwsgi-flask/).  It includes variants for each supported Python version (2.7, 3.6).

## Usage

Basic usage information is provided below.  If using the documentation on the original repo, remember to reference this image `robpco/nginx-uwsgi-flask`.

To use this image as a base for a **Flask Web-App**:

1. Create a `Dockerfile` that references this image:tag
2. Build an Image from that `Dockerfile`
3. Run the Image and Testing the App

### STEP 1 - Create a `Dockerfile`

- In this example, we use the `FROM` line to specify this image and the `python3.6` variant
- We copy our python scripts, in a sub-directory on the local computer called `app`, to a folder in the container called `/app`.

```Dockerfile
FROM robpco/nginx-uwsgi-flask:python3.6

COPY ./app /app
```

### STEP 2 - Build an image from the `Dockerfile`

- Next we use the `docker build` command to create the image, name it `myapp`

``` shell
docker build -t myapp .
```

### STEP 3 - Run the image and viewing the output

- Now, we can run the image and use a few extra parameters to make things easier
  - The `-p` parameter maps the localhost's port 8080 to port 80 of the image
  - The `-d` parameter `detaches` our terminal session from the image
  - The `--rm` parameter automatically removes the image when it's stopped.
  - Finally, we specify the name of the image: `myapp`
- After running the command, we can open up a web-browser and type in `http://localhost:8080` and interact with our Python Flask application

``` shell
docker run --rm -p 8080:80 -d myapp
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

- 2017-12-11: Added multiple tags per variant: `py3.6` is the same as `python3.6`, and so forth...
- 2017-11-29: Added ability to change port Nginx listens on with new environment variable `LISTEN_PORT`.
  - Thanks to github user [tmshn](https://github.com/tmshn)
- 2017-11-29: Automatic image re-build when Python updates
- 2017-11-28: Updated Nginx version installed
- 2018-05-04: Updated for new base images

## CHANGELOG

- 2017-12-15: Fix to avoid duplicate listen entries in nginx.conf
- 2017-11-30: limit build failures caused by GPG key validation failing
- 2017-11-28: Fixed console errors from supervisor process:
  - Added explicit path reference to `supervisord.conf` in Dockerfile `CMD` statement
  - Added explicitly set username in `supervisord.conf`
