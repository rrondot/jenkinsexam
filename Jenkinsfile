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

        stage('Deployment') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    def namespaces = ["dev", "qa", "staging", "prod"]
                    for (namespace in namespaces) {
                        deployNamespace(namespace)
                    }
                }
            }
        }
    }
}

def deployNamespace(namespace) {
    script {
        sh """
        rm -Rf .kube
        mkdir .kube
        ls
        cat $KUBECONFIG > .kube/config
        """

        createPersistentVolume(namespace)
        createPersistentVolumeClaim(namespace)

        deployHelm(namespace, "castdb")
        deployHelm(namespace, "moviedb")
        deployHelm(namespace, "cast-service")
        deployHelm(namespace, "movie-service")
    }
}

def createPersistentVolume(namespace) {
    script {
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
}

def createPersistentVolumeClaim(namespace) {
    script {
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
}

def deployHelm(namespace, service) {
    script {
        sh """
        cp ${service}/values.yaml ${service}-values.yml
        cat ${service}-values.yml
        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" ${service}-values.yml
        helm upgrade --install ${service} ${service} --values=${service}-values.yml --namespace ${namespace}
        """
    }
}
