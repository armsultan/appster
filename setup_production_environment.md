# Setup production environment

Our "Production" servers will be hosted as containers using `docker-compose`, and we will use the `watchtower` to automate
the updating and deployement of our containerized web app

#### Requirements:

 * Linux host
 * [`docker-ce`](https://docs.docker.com/install/linux/docker-ce/)
 * [`docker-compose`](https://docs.docker.com/compose/install/)


### Install docker and `docker-compose`

1. Install [`docker-ce`](https://docs.docker.com/install/linux/docker-ce/)
2. Install [`docker-compose`](https://docs.docker.com/compose/install/)

### Create a Credentials File for access to Private Docker Registry

 * source: https://mesosphere.github.io/marathon/docs/native-docker-private-registry.html

1. Log in to your private registry manually. This will create a `~/.docker` directory and a `~/.docker/config.json` file.

```bash
docker login some.docker.host.comUsername: fooPassword:Email: foo@bar.com
```

2. Check that you have the `~/.docker/config.json` file

```bash
ls ~/.dockerconfig.json
```

3. Your `config.json` file should look like the example below, where value of `auth` is a **based64-encoded** `username:password` string.
   You can generate it using: `echo -n 'username:password' | base64`

```json
{
  "auths": {
      "https://registry.gitlab.com": {
          "auth": "xxxxxxxxxxxxx",
          "email": "armand@email.com"
      }
  }
}
```

### Confifgure docker-compose to deploy two web servers and Watchtower

Using `docker-compose` we will deploy two instances of our NGINX Plus webserver hosting our sample app, `appster`, 
along with a [Watchtower](https://containrrr.github.io/watchtower/usage-overview/) container


`watchtower` is configured to check the image registery every 30s and when we update the web content, rebuild our docker 
container and push the image to our private repository, watchtower will update the running version of your containerized 
app by pulling down your new image, gracefully shut down your existing container and restart it with the same options
that were used when it was deployed initially.

Since we are pulling images from private Docker registry, we have to supply registry authentication credentials with the environment
variables `REPO_USER` and `REPO_PASS` **or** mount the host's docker config file into the container (at the root 
of the container filesystem `/`). Since we are going to use `docker-compose` we will mount volumes:

1. Create a `docker-compose.yml` file to run two instances of our web app on port `9090` and `9091`, and a `watchtower`
   container to keep our containerized app up to date by checking the images registry every 30s:

I have placed the [docker-compose.yml](extra/docker-compose.yml) in `/var/www/appster`:

```yaml
version: "3"
services:
  watchtower:
    image: containrrr/watchtower
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /root/.docker/config.json:/config.json
    command: --interval 30
  web1:
    image: registry.gitlab.com/armsultan/appster:latest
    ports:
      - '9090:80'
  web2:
    image: registry.gitlab.com/armsultan/appster:latest
    ports:
      - '9091:80'
```

### Optional: Setup startup script and cronjob

A startup script and cronjob will force the webapp to start on system reboot

 1. Create a startup script. I have placed the [start.sh](extra/start.sh) in `/var/www/appster`:

```bash
#! /usr/bin/env bash

# To Run on boot add the following cron job (crontab -e):
# Start appster on boot
# @reboot ( sleep 60 ; sh /var/www/appster/start.sh )

DIRECTORY=`dirname $0`
docker && docker-compose -f $DIRECTORY/docker-compose.yml up -d --remove-orphans
```

 1. Make the script executable for the user you wish to run your docker containers as:

```bash
chmod +x /var/www/appster/start.sh
```

 2. Install the following [cronjob](extra/cron) by running crontab -e under the using you wish to run your docker containers as

```cron
# Start appster on boot
@reboot ( sleep 60 ; sh /var/www/appster/start.sh)
```

The docker containers will start aftern system reboot...

--------------------------------------------------------------------------------

## Troubleshooting

### Stop and remove all docker containers and images

```bash
# List all containers (only IDs)
docker ps -aq

# Stop all running containers
docker stop $(docker ps -aq)

# Remove all containers
docker rm $(docker ps -aq)

# Remove all images
docker rmi $(docker images -q)