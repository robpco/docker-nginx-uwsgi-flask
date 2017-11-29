## Supported tags and respective `Dockerfile` links

* [`python2.7` _(Dockerfile)_](https://github.com/robertpeteuil/nginx-uwsgi-flask-docker/blob/master/python2.7/Dockerfile)
* [`python3.5` _(Dockerfile)_](https://github.com/robertpeteuil/nginx-uwsgi-flask-docker/blob/master/python3.5/Dockerfile)
* [`python3.6` _(Dockerfile)_](https://github.com/robertpeteuil/nginx-uwsgi-flask-docker/blob/master/python3.6/Dockerfile)
* [`python2.7-index` _(Dockerfile)_](https://github.com/robertpeteuil/nginx-uwsgi-flask-docker/blob/master/python2.7-index/Dockerfile)
* [`python3.5-index` _(Dockerfile)_](https://github.com/robertpeteuil/nginx-uwsgi-flask-docker/blob/master/python3.5/Dockerfile)
* [`python3.6-index` _(Dockerfile)_](https://github.com/robertpeteuil/nginx-uwsgi-flask-docker/blob/master/python3.6-index/Dockerfile)

## TODO: ADD ATTRIBUTION & LINK TO ORIGINAL REPO - PLACE MINIMUM INFO HERE

# nginx-uwsgi-flask

Single container **Docker** image with **Nginx** and **uWSGI** for **Flask** web applications written in Python.

## Description

This [**Docker**](https://www.docker.com/) image allows you to create [**Flask**](http://flask.pocoo.org/) web applications in [**Python**](https://www.python.org/) that run with [**uWSGI**](https://uwsgi-docs.readthedocs.org/en/latest/) and [**Nginx**](http://nginx.org/en/) in a single container.

uWSGI with Nginx is one of the best ways to deploy a Python web application.

**GitHub repo**:

**Docker Hub image**:

## General Instructions

You don't have to clone this repo, you should be able to use this image as a base image for your project with something in your `Dockerfile` like:

```Dockerfile
FROM tiangolo/uwsgi-nginx-flask:python3.6

COPY ./app /app
```

This Docker image is based on [**robpco/nginx-uwsgi**](https://hub.docker.com/r/robpco/nginx-uwsgi/). That Docker image has uWSGI and Nginx installed in the same container and was made to be the base of this image.

## QuickStart

**Note**: You can download the **example-flask-python3.6.zip** project example and use it as the template for your project from the section **Examples** above.

---

Or you may follow the instructions to build your project from scratch:

* Go to your project directory
* Create a `Dockerfile` with:

```Dockerfile
FROM robpco/nginx-uwsgi-flask:python3.6

COPY ./app /app
```

* Create an `app` directory and enter in it
* Create a `main.py` file (it should be named like that and should be in your `app` directory) with:

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

**Note**: The section with the `main()` function is for debugging purposes. To learn more, read the **Advanced instructions** below.

* You should now have a directory structure like:

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
