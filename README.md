# NGINX Plus CICD Demo

![gitlab CICD](extra/gitlab_cicd.png)

A CICD demo for [NGINX Plus](https://www.nginx.com/products/nginx/). **Just add [licenses](https://www.nginx.com/free-trial-request/)**

#### Requirements

1. Continuous Integration: Setup a [Gitlab CICD]((https://docs.gitlab.com/ee/ci/quick_start/)) continuous integration service
2. A Linux build server ([Gitlab Runner](https://docs.gitlab.com/ee/ci/runners/README.html)) with docker and [Crossplane](https://github.com/nginxinc/crossplane) installed
3. Docker images for NGINX Plus and [Crossplane](https://github.com/nginxinc/crossplane): Setup [Gitlab repository mirroring](https://docs.gitlab.com/ee/user/project/repository/repository_mirroring.html) to my [nginx-plus-dockerfiles](https://github.com/armsultan/nginx-plus-dockerfiles) repo, with the [Gitlab Container Registry](https://docs.gitlab.com/ee/user/packages/container_registry/) for your project enabled
4. *Optional:* Continuous Deployment: One linux docker host. See [`Setup Production Environment.md`](setup_production_environment.md)

#### Other setup Instructions:
 1. Place the following files in the directories of your build server
    * Retrieve your NGINX Plus Key and Certificate from the NGINX [customer portal](https://cs.nginx.com/) or from an activated evaluation, and copy the `nginx-repo.crt` and `nginx-repo.crt` files into `etc/ssl/nginx/`
 2. Automate a [CICD pipeline using gitlab](https://docs.gitlab.com/ee/ci/pipelines.html). A example gitlab CI/CD pipeline file (`.gitlab-ci.yml`) is provided.
 3. Optional: Modify the `Dockerfile` as necessary, e.g. To install addtional NGINX Plus [Dynamic modules](https://docs.nginx.com/nginx/admin-guide/dynamic-modules/dynamic-modules/). Place your own NGINX Plus configurations into `etc/nginx/`, including files in sub directories: i.e. `etc/nginx/conf.d` and `etc/nginx/stream.conf.d`

## Demos

### 1. Continuous Integration

#### Update our source repository and automaticly run our pipeline

1. Clone repo to local machine
2. Demonstrate a configuration change in the NGINX config or change in the Web App: 

##### Example 1. Update / revert phone image

1. Search and replace `iphone_7.png` with `iphone_x.png`. Change and revert as needed:

```bash
# This works with both GNU and BSD versions of sed:

# replace iphone 7 image to iphone x
sed -i '' 's/iphone_7.png/iphone_x.png/g' etc/nginx/html/index.html

# replace iphone x image to iphone 7
sed -i '' 's/iphone_x.png/iphone_7.png/g' etc/nginx/html/index.html
```

2. Commit and push changes to code repository:

```bash
git add .; git commit -m "changed phone image"; git push origin master
```

3. Watch the build process in realtime on [Gitlab](https://docs.gitlab.com/ee/ci/quick_start/)

##### Example 2. Update / revert background image

1. Search and replace `#ffb300` with `#512DA8`. Change and revert as needed:

```bash
# This works with both GNU and BSD versions of sed:

# Flip background colors - yellow to purple
sed -i '' 's/-45deg, #ffb300/-45deg, #512DA8/g' etc/nginx/html/css/bootstrap.min.css

# Flip background colors - purple to yellow
sed -i '' 's/-45deg, #512DA8/-45deg, #ffb300/g' etc/nginx/html/css/bootstrap.min.css
```

2. Commit and push changes to code repository:

```bash
git add .; git commit -m "changed background image"; git push origin master
```

3. Watch the build process in realtime on [Gitlab](https://docs.gitlab.com/ee/ci/quick_start/)

![appster iphone7](extra/appster_iphone7.png)
![appster iphonex](extra/appster_iphonex.png)

### 2. Continuous Deployment

#### Automate the deployement of our containerized web app to a live environment using Watchtower

In this demo we can illustrate how to automate the deployement of our containerized web app to a live environment.

We will use [watchtower](https://containrrr.github.io/watchtower/) to update the running version of your containerized app 
simply by pushing our a new image to a image registry. Watchtower will pull down your new image, gracefully shut down the
existing container(s) and restart it with the same options that were used when it was deployed initially.

1. First, follow the setup instructions as outlined in [`Setup Production Environment.md`](setup_production_environment.md)

2. After you push new changes to your source code repository, a Pipeline will kick off and upon a successful build,
   push the new docker image to the image repository.

3. After 30s have elapsed and there is a new image of our web app available with the tag `latest`, watchtower running on the
   production server will update the running version of our containerized app by:
   1. Pull down your new image
   2. Gracefully shutting down the existing container(s)
   3. Restart the container(s) with the same options that were used when it was deployed initially

4. You should be able to see the new changes on port `9090` or `9091`

Note: Running a load balacer in front of our containers, with active health checks (such as [NGINX Plus](https://www.nginx.com/products/nginx/), hint hint) will ensure availablity and minimize downtime