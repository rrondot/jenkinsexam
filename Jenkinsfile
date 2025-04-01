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
                    docker rm -f jenkins
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
                    docker rm -f jenkins
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

        stage('Deployment in dev') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    deployNamespace("dev")
                }
            }
        }

        stage('Deployment in qa') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    deployNamespace("qa")
                }
            }
        }

        stage('Deployment in staging') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    deployNamespace("staging")
                }
            }
        }

        stage('Deployment in prod') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    deployNamespace("prod")
                }
            }
        }
    }
}

def deployNamespace(namespace) {
    sh '''
    rm -Rf .kube
    mkdir .kube
    ls
    cat $KUBECONFIG > .kube/config
    '''

    createPersistentVolume(namespace)
    createPersistentVolumeClaim(namespace)

    deployHelm(namespace, "castdb")
    deployHelm(namespace, "moviedb")
    deployHelm(namespace, "cast-service")
    deployHelm(namespace, "movie-service")
}

def createPersistentVolume(namespace) {
    sh """
    cat <<EOF | kubectl apply -f -
    apiVersion: v1
    kind: PersistentVolume
    metadata:
      name: cast-db-volume-${namespace}
    spec:
      capacity:
        storage: 10Gi
      accessModes:
        - ReadWriteOnce
      persistentVolumeReclaimPolicy: Retain
      hostPath:
        path: "/mnt/data/cast-db-volume-${namespace}"
    EOF
    """
}

def createPersistentVolumeClaim(namespace) {
    sh """
    cat <<EOF | kubectl apply -f -
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: cast-db-pvc-${namespace}
      namespace: ${namespace}
    spec:
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 10Gi
      volumeName: cast-db-volume-${namespace}
    EOF
    """
}

def deployHelm(namespace, service) {
    sh """
    cp ${service}/values.yaml ${service}-values.yml
