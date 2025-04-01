pipeline {
    environment {
        DOCKER_ID = "rrondot"
        DOCKER_IMAGE = "cast-service"
        DOCKER_IMAGE2 = "movie-service"
        DOCKER_TAG = "v.${BUILD_ID}.0"
    }
    agent any
    stages {
      
        stage('Docker Build Cast') {
            steps {
                script {
                    sh '''
                    docker rm -f jenkins || true
                    docker build -t $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG . -f app/cast-service/Dockerfile
                    sleep 6
                    '''
                }
            }
        }
        stage('Docker Build Movie') {
            steps {
                script {
                    sh '''
                    docker rm -f jenkins || true
                    docker build -t $DOCKER_ID/$DOCKER_IMAGE2:$DOCKER_TAG . -f app/movie-service/Dockerfile
                    sleep 6
                    '''
                }
            }
        }
        stage('Docker Push') {
            environment {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS")
            }
            steps {
                script {
                    sh '''
                    docker login -u $DOCKER_ID -p $DOCKER_PASS
                    docker push $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
                    docker push $DOCKER_ID/$DOCKER_IMAGE2:$DOCKER_TAG
                    '''
                }
            }
        }
        stage('Deploiement en dev') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    ls
                    cat $KUBECONFIG > .kube/config
                    '''
                    sh '''
                    cp castdb/values.yaml castdb-values.yml
                    cat castdb-values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" castdb-values.yml
                    helm upgrade --install castdb castdb --values=castdb-values.yml --namespace dev
                    '''
                    sh '''
                    cp moviedb/values.yaml moviedb-values.yml
                    cat moviedb-values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" moviedb-values.yml
                    helm upgrade --install moviedb moviedb --values=moviedb-values.yml --namespace dev
                    '''
                    sh '''
                    cp cast-service/values.yaml cast-service-values.yml
                    cat cast-service-values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" cast-service-values.yml
                    helm upgrade --install cast-service cast-service --values=cast-service-values.yml --namespace dev
                    '''
                    sh '''
                    cp movie-service/values.yaml movie-service-values.yml
                    cat movie-service-values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" movie-service-values.yml
                    helm upgrade --install movie-service movie-service --values=movie-service-values.yml --namespace dev
                    '''
                }
            }
        }
        stage('Deploiement en qa') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    ls
                    cat $KUBECONFIG > .kube/config
                    '''
                    sh '''
                    cp castdb/values.yaml castdb-values.yml
                    cat castdb-values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" castdb-values.yml
                    helm upgrade --install castdb castdb --values=castdb-values.yml --namespace qa
                    '''
                    sh '''
                    cp moviedb/values.yaml moviedb-values.yml
                    cat moviedb-values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" moviedb-values.yml
                    helm upgrade --install moviedb moviedb --values=moviedb-values.yml --namespace qa
                    '''
                    sh '''
                    cp cast-service/values.yaml cast-service-values.yml
                    cat cast-service-values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" cast-service-values.yml
                    helm upgrade --install cast-service cast-service --values=cast-service-values.yml --namespace qa
                    '''
                    sh '''
                    cp movie-service/values.yaml movie-service-values.yml
                    cat movie-service-values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" movie-service-values.yml
                    helm upgrade --install movie-service movie-service --values=movie-service-values.yml --namespace qa
                    '''
                }
            }
        }
        stage('Deploiement en staging') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    ls
                    cat $KUBECONFIG > .kube/config
                    '''
                    sh '''
                    cp castdb/values.yaml castdb-values.yml
                    cat castdb-values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" castdb-values.yml
                    helm upgrade --install castdb castdb --values=castdb-values.yml --namespace staging
                    '''
                    sh '''
                    cp moviedb/values.yaml moviedb-values.yml
                    cat moviedb-values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" moviedb-values.yml
                    helm upgrade --install moviedb moviedb --values=moviedb-values.yml --namespace staging
                    '''
                    sh '''
                    cp cast-service/values.yaml cast-service-values.yml
                    cat cast-service-values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" cast-service-values.yml
                    helm upgrade --install cast-service cast-service --values=cast-service-values.yml --namespace staging
                    '''
                    sh '''
                    cp movie-service/values.yaml movie-service-values.yml
                    cat movie-service-values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" movie-service-values.yml
                    helm upgrade --install movie-service movie-service --values=movie-service-values.yml --namespace staging
                    '''
                }
            }
        }
        stage('Deploiement en prod') {
    when {
        expression {
            env.GIT_BRANCH.trim().contains('master')
        }
    }
    environment {
        KUBECONFIG = credentials("config")
    }
    steps {
        script {
            // Debug branch right before the deployment
            echo "Checking branch before prod deployment: ${env.GIT_BRANCH}"
            input message: 'Deploy to production?', ok: 'Deploy'
            sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
            '''
            sh '''
                cp castdb/values.yaml castdb-values.yml
                cat castdb-values.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" castdb-values.yml
                helm upgrade --install castdb castdb --values=castdb-values.yml --namespace prod
            '''
            sh '''
                cp moviedb/values.yaml moviedb-values.yml
                cat moviedb-values.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" moviedb-values.yml
                helm upgrade --install moviedb moviedb --values=moviedb-values.yml --namespace prod
            '''
            sh '''
                cp cast-service/values.yaml cast-service-values.yml
                cat cast-service-values.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" cast-service-values.yml
                helm upgrade --install cast-service cast-service --values=cast-service-values.yml --namespace prod
            '''
            sh '''
                cp movie-service/values.yaml movie-service-values.yml
                cat movie-service-values.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" movie-service-values.yml
                helm upgrade --install movie-service movie-service --values=movie-service-values.yml --namespace prod
            '''
        }
    }
}
    }
    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}