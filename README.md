Docker Django
-------------


Stack and version numbers used:

| Name           | Version |
|----------------|---------|
| Docker Compose | 1.12.0  |
| Nginx          | 1.11    |
| Postgresql     | 9.5.5   |
| Python         | 2.7.x   |
| uWSGI          | 2.0.12  |

## Folder structure

```
$ tree -L 1 --dirsfirst
.
├── config              # files needed for configuration
├── services            # services that support the webapp
├── webapp              # actual webapp
├── circle.yml          # circle ci setup file
├── docker-compose.yml  # docker-compose setup with container orchestration instructions
├── LICENSE             # license for this project
└── README.md           # this file
```

## Setting up

### Docker
See installation instructions at: [docker documentation](https://docs.docker.com)

```bash
$ curl -sSL https://get.docker.com/ubuntu/ | sudo sh
```

### Docker Compose
Install [docker compose](https://github.com/docker/compose), see installation
instructions at [https://docs.docker.com/compose/install/](https://docs.docker.com/compose/install/):

```bash
# Install with PyPi
$ pip install docker-compose

# or install via curl
curl -L https://github.com/docker/compose/releases/download/1.7.1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

Check the [github project](https://github.com/docker/docker-compose/releases) for new releases

### Django
Copy the `mysite` folder to the `webapp` folder. 

Edit `config/environment/development.env` file and add the name of your
project at `DJANGO_PROJECT_NAME`.


Edit the `settings.py` file with the correct database credentials and static root:

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': os.environ.get('POSTGRES_NAME'),
        'USER': os.environ.get('POSTGRES_USER'),
        'PASSWORD': os.environ.get('POSTGRES_PASSWORD'),
        'HOST': os.environ.get('POSTGRES_HOST'),
        'PORT': os.environ.get('POSTGRES_PORT'),
    }
}

STATIC_ROOT = '/srv/static-files'
```

### Environment variables
The file `config/environment/development.env` contains the environment
variables needed in the containers. You can edit this as you see fit, and at
the moment these are the defaults that this project uses. However when you
intend to use this, keep in mind that you should keep this file out of version
control as it can hold sensitive information regarding your project. The file
itself will contain some commentary on how a variable will be used in the
container.

## Fire it up
Start the container by issuing one of the following commands:
```bash
$ docker-compose up             # run in foreground
$ docker-compose up -d          # run in background
```

## Other commands
Build images:
```bash
$ docker-compose build
$ docker-compose build --no-cache       # build without cache
```

See processes:
```bash
$ docker-compose ps                 # docker-compose processes
$ docker ps -a                      # docker processes (sometimes needed)
$ docker stats [container_name]     # see live docker container metrics
```

Run commands in container:
```bash
# Name of service is the name you gave it in the docker-compose.yml
$ docker-compose run [service_name] /bin/bash
$ docker-compose run [service_name] python manage.py shell
$ docker-compose run [service_name] env                         # env vars
```

See logs
```bash
$ docker-compose logs -f webapp
$ docker-compose logs -f webserver
```

Print log directory 
```bash
$ docker inspect --format='{{.LogPath}}' containername
```

Remove all docker containers:
```bash
docker rm $(docker ps -a -q)
```

Remove all docker images:
```bash
docker rmi $(docker images -q)
```

### Some commands for managing the webapp
To initiate a command in an existing running container use the `docker exec`
command.

```bash
# Restart uwsgi in a running container.
$ docker exec [container-name] touch /etc/uwsgi/reload-uwsgi.ini

# Create migration file for an app
$ docker exec -it [container-name] python /srv/mysite/manage.py makemigrations scheduler

# Migrate
$ docker exec -it [container-name] python /srv/mysite/manage.py migrate

# Get sql contents of a migration
$ docker exec -it [container-name] python /srv/mysite/manage.py sqlmigrate [appname] 0001

# Get to interactive console
$ docker exec -it [container-name] python /srv/mysite/manage.py shell

# Testing
docker exec [container-name] python /srv/mysite/manage.py test
```

## Troubleshooting
Q: I get the following error message when using the docker command:

```
FATA[0000] Get http:///var/run/docker.sock/v1.16/containers/json: dial unix /var/run/docker.sock: permission denied. Are you trying to connect to a TLS-enabled daemon without TLS? 

```

A: Add yourself (user) to the docker group, remember to re-log after!

```bash
$ usermod -a -G docker <your_username>
$ service docker restart
```

Q: Changes in my code are not being updated despite using volumes.

A: Remember to restart uWSGI for the changes to take effect.

```bash
# Find container_name by using docker-compose ps
$ docker exec [container_name] touch /etc/uwsgi/reload-uwsgi.ini

# Example
$ docker exec docker_webapp_1 touch /etc/uwsgi/reload-uwsgi.ini
```
