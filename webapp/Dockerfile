FROM python:2.7

# Add code
ADD ./mysite /srv/mysite

# Install application requirements
ADD ./mysite/requirements.txt /srv/mysite/
RUN pip install -r /srv/mysite/requirements.txt

# Split Dockerfile here

# Add start script
ADD ./config/start.sh /

# Add uWSGI config
ADD ./config/django-uwsgi.ini /etc/uwsgi/django-uwsgi.ini

# Add database check script
ADD ./config/database-check.py /srv/config/database-check.py

# Create django user, will own the Django app
RUN adduser --no-create-home --disabled-login --group --system django
RUN chown -R django:django /srv/mysite

# Execute start script
CMD ["./start.sh"]
