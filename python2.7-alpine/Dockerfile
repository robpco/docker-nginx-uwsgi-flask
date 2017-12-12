FROM robpco/nginx-uwsgi:python2.7-alpine

RUN pip install --no-cache-dir --disable-pip-version-check flask

# Enable unlimited filesize uploads (restore nginx default by setting to 1m)
ENV NGINX_MAX_UPLOAD 0

# Specify uwsgi.ini file to use
ENV UWSGI_INI /app/uwsgi.ini

# Enable serving static files directly by Nginx
# Top-level URL where requests for static files originate
ENV STATIC_URL /static
# Container location of Static Files (absolute path)
ENV STATIC_PATH /app/static
# Specify "ENV STATIC_INDEX 1" to serve '/' directly from /static/index.html (or the static URL configured)
ENV STATIC_INDEX 0

# entrypoint.sh generates Nginx config based on ENV values at container start
COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]

# copy application-code to image
COPY ./app /app
WORKDIR /app

CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]
