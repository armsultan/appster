# NGINX Plus CICD Demo

A CICD demo for [NGINX Plus](https://www.nginx.com/products/nginx/). **Just add licenses**

# Requirements

1. A build server ([Gitlab Runner](https://docs.gitlab.com/ee/ci/runners/README.html)) with docker and [Crossplane](https://github.com/nginxinc/crossplane) installed 
2. [Gitlab repository mirroring](https://docs.gitlab.com/ee/user/project/repository/repository_mirroring.html) to [this repo](https://github.com/armsultan/nginx-plus-dockerfiles) with the [Gitlab Container Registry](https://docs.gitlab.com/ee/user/packages/container_registry/) for your project enabled
3. [Gitlab CICD]((https://docs.gitlab.com/ee/ci/quick_start/)) continuous integration service

# Instructions
 1. Place the following files in the directories of your build server
    * Retrieve your NGINX Plus Key and Certificate from the NGINX [customer portal](https://cs.nginx.com/) or from an activated evaluation, and copy the `nginx-repo.crt` and `nginx-repo.crt` files into `etc/ssl/nginx/`
 2. Automate a [CICD pipeline using gitlab](https://docs.gitlab.com/ee/ci/pipelines.html). A example gitlab CI/CD pipeline file (`.gitlab-ci.yml`) is provided.
 3. Modify the `Dockerfile` as necessary, e.g. To install addtional NGINX Plus [Dynamic modules](https://docs.nginx.com/nginx/admin-guide/dynamic-modules/dynamic-modules/). Place your own NGINX Plus configurations into `etc/nginx/`, including files in sub directories: i.e. `etc/nginx/conf.d` and `etc/nginx/stream.conf.d`

## Demos

### 1. Continuous Integration: Update our source repository and automaticly run our pipeline

1. Clone repo to local machine
2. Demonstrate a configuration change in the NGINX config or change in the Web App. E.g. Search and replace `iphone_7.png` with `iphone_x.png`. Change and revert as needed, e.g.

```bash
# This works with both GNU and BSD versions of sed:

# replace iphone 7 image to iphone x
sed -i '' 's/iphone_7.png/iphone_x.png/g' etc/nginx/html/index.html

# replace iphone x image to iphone 7
sed -i '' 's/iphone_x.png/iphone_7.png/g' etc/nginx/html/index.html
```

3. Commit and push changes to code repository:
```bash
git add .; git commit -m "changed phone image"; git push origin master
```
3. Watch the build process in realtime on [Gitlab](https://docs.gitlab.com/ee/ci/quick_start/)

### 2. Continuous Deployment: Automate the deployement of our containerized web app to a live environment using Watchtower

In this demo we can illustrate how to automate the deployement of our containerized web app to a live environment.

We can use [watchtower](https://containrrr.github.io/watchtower/) to update the running version of your containerized app 
simply by pushing our a new image to a image registry. Watchtower will pull down your new image, gracefully shut down the
existing container(s) and restart it with the same options that were used when it was deployed initially.

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
