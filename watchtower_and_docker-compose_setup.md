## Setup watchtower and docker-compose

Run the watchtower container with the following command:

#### Setup: Using a Private Docker Registry

 * source: https://mesosphere.github.io/marathon/docs/native-docker-private-registry.html 

##### Create a Credentials File
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

4. Enable [Watchtower](https://containrrr.github.io/watchtower/usage-overview/) to automaticly update docker image.
   Since we are pulling images from private Docker registry, supply registry authentication credentials with the environment
   variables `REPO_USER` and `REPO_PASS` **or** by mounting the host's docker config file into the container (at the root 
   of the container filesystem `/`). Since we are going to use `docker-compose` see next step..

#### 2. Use Docker-compose to deploy two web servers and Watchtower

On a Docker host, we will deploy two instances of our NGINX Plus webserver hosting our sample app, `appster`.

`watchtower` is configured to check the image registery every 30s and when we update the web content, rebuild our docker 
container and push the image to our private repository, watchtower will update the running version of your containerized 
app by pulling down your new image, gracefully shut down your existing container and restart it with the same options
that were used when it was deployed initially.

1. Create a `docker-compose.yml` file to run two instances of our web app on port `9090` and `9091`, alongside the `watchtower`
   container to keep our ontainerized app up to date:

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