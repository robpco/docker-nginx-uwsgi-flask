# Supported tags and respective `Dockerfile` links

* [`python2.7` _(Dockerfile)_](https://github.com/robertpeteuil/docker-nginx-uwsgi-flask/blob/master/python2.7/Dockerfile)
* [`python3.5` _(Dockerfile)_](https://github.com/robertpeteuil/docker-nginx-uwsgi-flask/blob/master/python3.5/Dockerfile)
* [`python3.6` _(Dockerfile)_](https://github.com/robertpeteuil/docker-nginx-uwsgi-flask/blob/master/python3.6/Dockerfile)

**The `latest` tag is not supported - one of the tags above must be explicitly specified.**
- This is because the tags represent different variants not incremental versions.
- This eliminates importing or pulling an unexpected version

# nginx-uwsgi-flask

Docker image with **Nginx**, **uWSGI** and **Flask** in a single container that enables running Python Flask Apps on NGINX.

*NOTE: This project began as a derivative of the project at [tiangolo/UWSGI-NGINX-DOCKER](https://github.com/tiangolo/uwsgi-nginx-flask-docker).  It was created due to an urgent need for enhacements.* 

Implementing the enhancements required creating derivatives of both the [base images](https://github.com/robertpeteuil/docker-nginx-uwsgi) and the [flask images](https://github.com/robertpeteuil/docker-nginx-uwsgi-flask).

GitHub Repo: <https://github.com/robertpeteuil/docker-nginx-uwsgi-flask>

Docker Hub Images: <https://hub.docker.com/r/robpco/nginx-uwsgi-flask/>

# Overview

These **Docker** images allow the creation, and migration, of Python **Flask** Web Apps to run on **Nginx** with **uWSGI** in a single container.  This image simplifies the task of deploying a Flask-based App to an Nginx-based implementation.

These images build on [base images](https://hub.docker.com/r/robpco/nginx-uwsgi/) and adding Flask, Environment Variables and auto-generated Nginx configuration during container startup.

**GitHub repo**: <https://github.com/robertpeteuil/docker-nginx-uwsgi-flask>

**Docker Hub image**: <https://hub.docker.com/r/robpco/nginx-uwsgi-flask/>

# Enhancements

The images used in this repo includes the following enhancements:
- The addition of an alpine-linux variants
- Ability to change Nginx listen port with the LISTEN_PORT environment variable
- Nginx updated to 1.13.7 on non-alpine variants
- `supervisord` enhancements to reduce CRIT errors
  - `supervisord.conf` is explicitly referenced via the Dockerfile CMD statement
  - `supervisord.conf` includes an explicitly set user-name
- Automatic image republishing with Python image updates

# Usage

The Docker-Hub [repository](https://hub.docker.com/r/robpco/nginx-uwsgi-flask/) contains auto-generated images from this repo.  They can be referenced (or pulled) by using the image name `robpco/nginx-uwsgi-flask`, plus a tag for the python version desired (ex: `:python3.6`), and optionally appending `-alpine` to the tag (for alpine variants).

**Detailed usage documentation and examples can be found on the repo by Sebastián Ramírez [tiangolo/UWSGI-NGINX-FLASK-DOCKER](https://github.com/tiangolo/uwsgi-nginx-flask-docker).**

# Custom Environment Variables

These images support the following custom environment variables:

# QuickStart

To use this image:

- Create a project directory
- Create a `Dockerfile` containing the basics here
  - the image tag in the FROM line should match the version of Python your app is written in
  - you can optionally add an "-alpine" suffix if you want to use the alpine images.

```Dockerfile
FROM robpco/nginx-uwsgi-flask:python3.6

COPY ./app /app
```

- Create an `app` directory and enter in it
- Create a `main.py` file (it should be named like that and should be in your `app` directory) with:

```python
from flask import Flask
app = Flask(__name__)

@app.route("/")
def hello():
    return "Hello World from Flask"

if __name__ == "__main__":
    # Only for debugging while developing
    app.run(host='0.0.0.0', debug=True, port=80)
```

the main application object should be named `app` (in the code) as in this example.

- You should now have a directory structure like:

```
.
├── app
│   └── main.py
└── Dockerfile
```

* Go to the project directory (in where your `Dockerfile` is, containing your `app` directory)
* Build your Flask image:

```bash
docker build -t myimage .
```

* Run a container based on your image:

```bash
docker run -d --name mycontainer -p 80:80 myimage
```

...and you have an optimized Flask server in a Docker container.

You should be able to check it in your Docker container's URL, for example: <http://192.168.99.100/>


## QuickStart for serving both Static Content and Web API

This section explains how to configure the image to serve the contents of `/static/index.html` directly when the browser requests `/`.

This is specially helpful (and efficient) if you are building a Single-Page Application (SPA) with JavaScript (Angular, React, etc) and you want the `index.html` to be served directly, without modifications by Python or Jinja2 templates. And you want to use Flask mainly as an API / back end for your SPA front end.

---

Or you may follow the instructions to build your project from scratch (it's very similar to the procedure above):

* Go to your project directory
* Create a `Dockerfile` with:

```Dockerfile
FROM robpco/nginx-uwsgi-flask:python3.6

ENV STATIC_INDEX 1

COPY ./app /app
```

* Create an `app` directory and enter in it
* Create a `main.py` file (it should be named like that and should be in your `app` directory) with:

```python
from flask import Flask, send_file
app = Flask(__name__)


@app.route("/hello")
def hello():
    return "Hello World from Flask"


@app.route("/")
def main():
    index_path = os.path.join(app.static_folder, 'index.html')
    return send_file(index_path)


# Everything not declared before (not a Flask route / API endpoint)...
@app.route('/<path:path>')
def route_frontend(path):
    # ...could be a static file needed by the front end that
    # doesn't use the `static` path (like in `<script src="bundle.js">`)
    file_path = os.path.join(app.static_folder, path)
    if os.path.isfile(file_path):
        return send_file(file_path)
    # ...or should be handled by the SPA's "router" in front end
    else:
        index_path = os.path.join(app.static_folder, 'index.html')
        return send_file(index_path)


if __name__ == "__main__":
    # Only for debugging while developing
    app.run(host='0.0.0.0', debug=True, port=80)
```

the main application object should be named `app` (in the code) as in this example.

**Note**: The section with the `main()` function is for debugging purposes. To learn more, read the **Advanced instructions** below.

* Make sure you have an `index.html` file in `./app/static/index.html`, for example with:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Index</title>
</head>
<body>
<h1>Hello World from HTML</h1>
</body>
</html>
```

* You should now have a directory structure like:

```
.
├── app
│   ├── main.py
│   └── static
│       └── index.html
└── Dockerfile
```

* Go to the project directory (in where your `Dockerfile` is, containing your `app` directory)
* Build your Flask image:

```bash
docker build -t myimage .
```

* Run a container based on your image:

```bash
docker run -d --name mycontainer -p 80:80 myimage
```

...and you have an optimized Flask server in a Docker container. Also optimized to serve your main static `index.html` page.

* Now, when you go to your Docker container URL, for example: <http://192.168.99.100/>, you will see your `index.html`.

* You should be able to also go to, for example, <http://192.168.99.100/hello> to see a "Hello World" page served by Flask.

**Note**: As your `index.html` file will be served from `/` and from `/static/index.html`, it would be better to have absolute paths in the links to other files in the `static` directory from your `index.html` file. As in `/static/css/styles.css` instead of relative paths as in `./css/styles.css`.


## License

This project is licensed under the terms of the Apache license.
