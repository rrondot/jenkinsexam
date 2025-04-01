pipeline {
    environment {
        DOCKER_ID = "rrondot"
        DOCKER_IMAGE = "cast-service"
        DOCKER_IMAGE2 = "movie-service"
        DOCKER_TAG = "v.${BUILD_ID}.0"
    }

    agent any

    stages {
        stage('Docker Build & Push') {
            steps {
                script {
                    sh '''
                    docker build -t $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG . -f app/cast-service/Dockerfile
                    docker build -t $DOCKER_ID/$DOCKER_IMAGE2:$DOCKER_TAG . -f app/movie-service/Dockerfile
                    docker login -u $DOCKER_ID -p $(cat /run/secrets/DOCKER_HUB_PASS)
                    docker push $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
                    docker push $DOCKER_ID/$DOCKER_IMAGE2:$DOCKER_TAG
                    '''
                }
            }
        }

        stage('Deploy to dev') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    cat $KUBECONFIG > .kube/config
                    '''

                    // Create PersistentVolume and PersistentVolumeClaim for dev
                    sh '''
                    cat <<EOF | kubectl apply -f -
                    apiVersion: v1
                    kind: PersistentVolume
                    metadata:
                      name: cast-db-volume-dev
                    spec:
                      capacity:
                        storage: 10Gi
                      accessModes:
                        - ReadWriteOnce
                      persistentVolumeReclaimPolicy: Retain
                      hostPath:
                        path: "/mnt/data/cast-db-volume-dev"
                    EOF

                    cat <<EOF | kubectl apply -f -
                    apiVersion: v1
                    kind: PersistentVolumeClaim
                    metadata:
                      name: cast-db-pvc-dev
                      namespace: dev
                    spec:
                      accessModes:
                        - ReadWriteOnce
                      resources:
                        requests:
                          storage: 10Gi
                      volumeName: cast-db-volume-dev
                    EOF
                    '''

                    // Deploy services in dev
                    deployHelm("dev")
                }
            }
        }

        stage('Deploy to qa') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    cat $KUBECONFIG > .kube/config
                    '''

                    // Create PersistentVolume and PersistentVolumeClaim for qa
                    sh '''
                    cat <<EOF | kubectl apply -f -
                    apiVersion: v1
                    kind: PersistentVolume
                    metadata:
                      name: cast-db-volume-qa
                    spec:
                      capacity:
                        storage: 10Gi
                      accessModes:
                        - ReadWriteOnce
                      persistentVolumeReclaimPolicy: Retain
                      hostPath:
                        path: "/mnt/data/cast-db-volume-qa"
                    EOF

                    cat <<EOF | kubectl apply -f -
                    apiVersion: v1
                    kind: PersistentVolumeClaim
                    metadata:
                      name: cast-db-pvc-qa
                      namespace: qa
                    spec:
                      accessModes:
                        - ReadWriteOnce
                      resources:
                        requests:
                          storage: 10Gi
                      volumeName: cast-db-volume-qa
                    EOF
                    '''

                    // Deploy services in qa
                    deployHelm("qa")
                }
            }
        }

        stage('Deploy to staging') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    cat $KUBECONFIG > .kube/config
                    '''

                    // Create PersistentVolume and PersistentVolumeClaim for staging
                    sh '''
                    cat <<EOF | kubectl apply -f -
                    apiVersion: v1
                    kind: PersistentVolume
                    metadata:
                      name: cast-db-volume-staging
                    spec:
                      capacity:
                        storage: 10Gi
                      accessModes:
                        - ReadWriteOnce
                      persistentVolumeReclaimPolicy: Retain
                      hostPath:
                        path: "/mnt/data/cast-db-volume-staging"
                    EOF

                    cat <<EOF | kubectl apply -f -
                    apiVersion: v1
                    kind: PersistentVolumeClaim
                    metadata:
                      name: cast-db-pvc-staging
                      namespace: staging
                    spec:
                      accessModes:
                        - ReadWriteOnce
                      resources:
                        requests:
                          storage: 10Gi
                      volumeName: cast-db-volume-staging
                    EOF
                    '''

                    // Deploy services in staging
                    deployHelm("staging")
                }
            }
        }

        stage('Deploy to prod') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    cat $KUBECONFIG > .kube/config
                    '''

                    // Create PersistentVolume and PersistentVolumeClaim for prod
                    sh '''
                    cat <<EOF | kubectl apply -f -
                    apiVersion: v1
                    kind: PersistentVolume
                    metadata:
                      name: cast-db-volume-prod
                    spec:
                      capacity:
                        storage: 10Gi
                      accessModes:
                        - ReadWriteOnce
                      persistentVolumeReclaimPolicy: Retain
                      hostPath:
                        path: "/mnt/data/cast-db-volume-prod"
                    EOF

                    cat <<EOF | kubectl apply -f -
                    apiVersion: v1
                    kind: PersistentVolumeClaim
                    metadata:
                      name: cast-db-pvc-prod
                      namespace: prod
                    spec:
                      accessModes:
                        - ReadWriteOnce
                      resources:
                        requests:
                          storage: 10Gi
                      volumeName: cast-db-volume-prod
                    EOF
                    '''

                    // Deploy services in prod
                    deployHelm("prod")
                }
            }
        }
    }
}

// Function to deploy all services in a given namespace
def deployHelm(namespace) {
    sh """
    cp castdb/values.yaml castdb-values.yml
    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" castdb-values.yml
    sed -i "s+cast-db-volume+cast-db-volume-${namespace}+g" castdb-values.yml
    helm upgrade --install castdb castdb --values=castdb-values.yml --namespace ${namespace}

    cp moviedb/values.yaml moviedb-values.yml
    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" moviedb-values.yml
    helm upgrade --install moviedb moviedb --values=moviedb-values.yml --namespace ${namespace}

    cp cast-service/values.yaml cast-service-values.yml
    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" cast-service-values.yml
    helm upgrade --install cast-service cast-service --values=cast-service-values.yml --namespace ${namespace}

    cp movie-service/values.yaml movie-service-values.yml
    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" movie-service-values.yml
    helm upgrade --install movie-service movie-service --values=movie-service-values.yml --namespace ${namespace}
    """
}
