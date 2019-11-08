pipeline {
    agent {
        label 'build' // Use build node
        }
    triggers {
        pollSCM('') // Enabling being build on Push
    }
    environment {
        REGISTRY = "repo.t3st.org:48083/appster-${BRANCH_NAME}"
        REGISTRY_URL = "https://repo.t3st.org:48083"
        REGISTRY_CREDENTIALS = 'jenkins_nexus'
        DOCKER_IMAGE = ''
        DOCKER_IMAGE_TAG = "$REGISTRY:$BUILD_NUMBER"
    }
    stages {
            stage('Build') {
                steps {
                    sh 'cp /etc/ssl/nginx/nginx-repo.key $WORKSPACE/etc/ssl/nginx'
                    sh 'cp /etc/ssl/nginx/nginx-repo.crt $WORKSPACE/etc/ssl/nginx'
                    script {
                        DOCKER_IMAGE = docker.build REGISTRY + ":$BUILD_NUMBER"
                    }
                    sh 'docker images'
                    // sh 'docker save $DOCKER_IMAGE_TAG | gzip > appster-$BRANCH_NAME-$BUILD_NUMBER.tgz'
                    // sh 'docker save $DOCKER_IMAGE_TAG > appster-$BRANCH_NAME-$BUILD_NUMBER.tar'

                }
            }
            stage('Nginx syntax configuration check') {
                 steps {
                    sh 'whoami'
                    sh 'echo $WORKSPACE'
                    sh 'ls -la'
                    sh 'docker run --rm -t -a stdout --name my-nginx --sysctl net.ipv4.ip_nonlocal_bind=1 -v $WORKSPACE/etc/nginx:/etc/nginx/ $DOCKER_IMAGE_TAG nginx -t'
                 }
            }
            stage('Deploy Image') {
                steps{
                    script {
                        // Use private repo
                        docker.withRegistry( REGISTRY_URL, REGISTRY_CREDENTIALS ) {
                            DOCKER_IMAGE.push()
                        }
                    }
                }
            }
            stage('Publish to prod') {
                steps {
                    // sh 'scp -P 2222 appster-$BRANCH_NAME-$BUILD_NUMBER.tgz corgi@142.93.91.120:/home/corgi'
                    // sh "ssh -t corgi@142.93.91.120 -p 2222 'gunzip -c appster-$BRANCH_NAME-$BUILD_NUMBER.tgz | docker load'"
                    // sh 'scp -P 2222 appster-$BRANCH_NAME-$BUILD_NUMBER.tar corgi@142.93.91.120:/home/corgi'
                    // sh "ssh -t corgi@142.93.91.120 -p 2222 'docker load < appster-$BRANCH_NAME-$BUILD_NUMBER.tar'"
                    // sh "ssh -t corgi@142.93.91.120 -p 2222 'docker stop $(docker ps -aq)'"
                    //sh "ssh -t corgi@142.93.91.120 -p 2222 'docker run -d -p 80:80 -p 443:443 -p 8080:8080 $DOCKER_IMAGE_TAG'"
                    sh "ssh -t root@104.248.185.191 '/home/corgi/./update-nginx-config.sh'"

                }
            }
            // parallel {
            //     stage('Unit tests') {
            //         agent any
            //         steps {
            //             script {
            //                 docker_image.inside("--entrypoint='/start.sh'") {
            //                     sh 'cd /var/www/app && vendor/bin/phpunit --testsuite=Unittest'
            //                 }
            //             }
            //         }
            //     }
            //     stage('Health check') {
            //         agent any
            //         steps {
            //             script {
            //                 docker_image.inside("--entrypoint='/start.sh'") {
            //                     timeout(time: 1, unit: 'MINUTES') {
            //                         retry(5) {
            //                             sleep 5
            //                             sh "curl -sS http://localhost/info | grep 'My API'"
            //                         }
            //                     }
            //                 }
            //             }
            //         }
            //     }
            // }
        // stage('Deploy NGINX to POC Server') {
        //     steps {
        //         sshagent(['NGINX_VM1_Pass']) {
        //             sh "scp -r nginx/ root@192.168.20.40:~"
        //             sh "ssh -tt -o StrictHostKeyChecking=no -l root 192.168.20.40 whoami"
        //             sh "ssh -tt -o StrictHostKeyChecking=no -l root 192.168.20.40 'sudo cp -R ~/nginx/ /etc/'"
        //             sh "ssh -tt -o StrictHostKeyChecking=no -l root 192.168.20.40 'sudo /usr/sbin/nginx -t && sudo nginx -s reload'"
        //         }
        //     }
        // }
    }
    post {
        cleanup {
            echo 'Cleanup'
            // Remove the image built in this build
            sh 'docker rmi $DOCKER_IMAGE_TAG'
            //clean up any resources — images, containers, volumes, and networks — that are dangling (not associated with a container) and any stopped containers and all unused images (not just dangling images)
            sh 'docker system prune -a -f'
            // Delete all others:
            //clean up any resources — images, containers, volumes, and networks — that are dangling (not associated with a container) and any stopped containers and all unused images (not just dangling images)
            //sh 'docker system prune -a -f'
            // docker stop $(docker ps -q)
            // docker rm $(docker ps -a -q)
            // docker rmi $(docker images -q -f dangling=true)
            sh 'docker images'
            deleteDir()
        }
    }
}