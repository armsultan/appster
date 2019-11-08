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

1. Clone repo to local machine 
2. Demonstrate a configuration change in the NGINX config or change in the Web App. E.g. Search and replace `iphone_7.png` with `iphone_x.png`. Change and revert as needed
3. Commit and push changes to code repository:
```bash
git commit .; git commit -m "change description"; git push origin master
```
3. Watch the build process in realtime on [Gitlab](https://docs.gitlab.com/ee/ci/quick_start/)

## TODO:
 * Extend the CICD pipeline to deploy NGINX i.e. "Production"
