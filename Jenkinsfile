pipeline {
environment { // Declaration of environment variables
DOCKER_ID = "rrondot" // replace this with your docker-id
DOCKER_IMAGE = "cast-service"
DOCKER_IMAGE2 = "movie-service"
DOCKER_TAG = "v.${BUILD_ID}.0" // we will tag our images with the current build in order to increment the value by 1 with each new build
}
agent any // Jenkins will be able to select all available agents
stages {
        stage(' Docker Build Cast'){ // docker build image stage
            steps {
                script {
                sh '''
                 docker rm -f jenkins
                 docker build -t $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG . -f app/cast-service/Dockerfile
                sleep 6
                '''
                }
            }
        }
       stage(' Docker Build Movie'){ // docker build image stage
            steps {
                script {
                sh '''
                 docker rm -f jenkins
                 docker build -t $DOCKER_ID/$DOCKER_IMAGE2:$DOCKER_TAG . -f app/movie-service/Dockerfile
                sleep 6
                '''
                }
            }
        }

        stage('Docker Push'){ //we pass the built image to our docker hub account
            environment
            {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS") // we retrieve  docker password from secret text called docker_hub_pass saved on jenkins
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

stage('Deploiement en dev'){
        environment
        {
        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
        }
            steps {
    script {
        sh '''
        rm -Rf .kube
        mkdir .kube
        ls
        cat $KUBECONFIG > .kube/config
        '''

        // Deploy castdb
        sh '''
        cp castdb/values.yaml castdb-values.yml
        cat castdb-values.yml
        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" castdb-values.yml
        helm upgrade --install castdb castdb --values=castdb-values.yml --namespace dev
        '''

        // Deploy moviedb
        sh '''
        cp moviedb/values.yaml moviedb-values.yml
        cat moviedb-values.yml
        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" moviedb-values.yml
        helm upgrade --install moviedb moviedb --values=moviedb-values.yml --namespace dev
        '''

        // Deploy cast-service
        sh '''
        cp cast-service/values.yaml cast-service-values.yml
        cat cast-service-values.yml
        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" cast-service-values.yml
        helm upgrade --install cast-service cast-service --values=cast-service-values.yml --namespace dev
        '''

        // Deploy movie-service
        sh '''
        cp movie-service/values.yaml movie-service-values.yml
        cat movie-service-values.yml
        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" movie-service-values.yml
        helm upgrade --install movie-service movie-service --values=movie-service-values.yml --namespace dev
        '''
    }
}



}
stage('Deploiement en qa'){
        environment
        {
        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
        }
            steps {
    script {
        sh '''
        rm -Rf .kube
        mkdir .kube
        ls
        cat $KUBECONFIG > .kube/config
        '''

        // Deploy castdb
        sh '''
        cp castdb/values.yaml castdb-values.yml
        cat castdb-values.yml
        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" castdb-values.yml
        helm upgrade --install castdb castdb --values=castdb-values.yml --namespace qa
        '''

        // Deploy moviedb
        sh '''
        cp moviedb/values.yaml moviedb-values.yml
        cat moviedb-values.yml
        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" moviedb-values.yml
        helm upgrade --install moviedb moviedb --values=moviedb-values.yml --namespace qa
        '''

        // Deploy cast-service
        sh '''
        cp cast-service/values.yaml cast-service-values.yml
        cat cast-service-values.yml
        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" cast-service-values.yml
        helm upgrade --install cast-service cast-service --values=cast-service-values.yml --namespace qa
        '''

        // Deploy movie-service
        sh '''
        cp movie-service/values.yaml movie-service-values.yml
        cat movie-service-values.yml
        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" movie-service-values.yml
        helm upgrade --install movie-service movie-service --values=movie-service-values.yml --namespace qa
        '''
    }
}



}
stage('Deploiement en staging'){
        environment
        {
        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
        }
            steps {
    script {
        sh '''
        rm -Rf .kube
        mkdir .kube
        ls
        cat $KUBECONFIG > .kube/config
        '''

        // Deploy castdb
        sh '''
        cp castdb/values.yaml castdb-values.yml
        cat castdb-values.yml
        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" castdb-values.yml
        helm upgrade --install castdb castdb --values=castdb-values.yml --namespace staging
        '''

        // Deploy moviedb
        sh '''
        cp moviedb/values.yaml moviedb-values.yml
        cat moviedb-values.yml
        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" moviedb-values.yml
        helm upgrade --install moviedb moviedb --values=moviedb-values.yml --namespace staging
        '''

        // Deploy cast-service
        sh '''
        cp cast-service/values.yaml cast-service-values.yml
        cat cast-service-values.yml
        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" cast-service-values.yml
        helm upgrade --install cast-service cast-service --values=cast-service-values.yml --namespace staging
        '''

        // Deploy movie-service
        sh '''
        cp movie-service/values.yaml movie-service-values.yml
        cat movie-service-values.yml
        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" movie-service-values.yml
        helm upgrade --install movie-service movie-service --values=movie-service-values.yml --namespace staging
        '''
    }
}
}
stage('Deploiement en prod'){
        environment
        {
        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
        }
            steps {
    script {
        sh '''
        rm -Rf .kube
        mkdir .kube
        ls
        cat $KUBECONFIG > .kube/config
        '''

        // Deploy castdb
        sh '''
        cp castdb/values.yaml castdb-values.yml
        cat castdb-values.yml
        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" castdb-values.yml
        helm upgrade --install castdb castdb --values=castdb-values.yml --namespace prod
        '''

        // Deploy moviedb
        sh '''
        cp moviedb/values.yaml moviedb-values.yml
        cat moviedb-values.yml
        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" moviedb-values.yml
        helm upgrade --install moviedb moviedb --values=moviedb-values.yml --namespace prod
        '''

        // Deploy cast-service
        sh '''
        cp cast-service/values.yaml cast-service-values.yml
        cat cast-service-values.yml
        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" cast-service-values.yml
        helm upgrade --install cast-service cast-service --values=cast-service-values.yml --namespace prod
        '''

        // Deploy movie-service
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
}