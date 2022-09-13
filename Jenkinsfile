pipeline { 
    environment { 
        ID_DOCKER = "${votre_id_dockerhub}"
        IMAGE_NAME = "nginx:1.21.1"
        IMAGE_TAG = "latest"
        STAGING = "${ID_DOCKER}-staging"
        PRODUCTION = "${ID_DOCKER}-production"
    }
    agent none
    stages {
        stage('Build image') {
            any
            steps {
                script {
                    sh 'docker build -t ${ID_DOCKER}/${IMAGE_NAME}:${IMAGE_TAG} .'
                }
            }
        }
		stage('Run container based on builded image') {
		agent any
		steps {
				script {
                    sh '''
                        echo "Cleaning Environment"
                        docker rm -f $IMAGE_NAME || echo "container does not exist"
                        docker run -d -p 80:5000 -e PORT=5000 --name ${IMAGE_NAME} ${ID_DOCKER}/${IMAGE_NAME}:${IMAGE_TAG}
                        sleep 5
                    '''
                }
            }
        stage('Test image') {
            agent any
            steps {
                script {
                    sh '''
                        echo "Testing Image..."
                        export http_proxy="${HTTP_PROXY}"
                        export https_proxy="${HTTPS_PROXY}"
                        curl -I http://docker-jenkins.web-connectivity.fr:80"
                    '''
                }
            }
        stage('Clean Container') {
            agent any
            steps {
                script {
                    sh '''
                        echo "Cleaning Container..."
                        docker stop ${IMAGE_NAME}
                        docker rm  ${IMAGE_NAME}
                    '''
                }
            }
        }
        stage ('Login and Push Image on docker hub') {
            agent any
            environment {
                DOCKERHUB_PASSWORD  = credentials("${ID}")
            }
            setps {
                script {
                    sh '''
                        export http_proxy="${HTTP_PROXY}"
                        export https_proxy="${HTTPS_PROXY}"
                        echo $DOCKERHUB_PASSWORD_PSW | docker login -u $ID_DOCKER --password-stdin
                        docker push ${ID_DOCKER}/${IMAGE_NAME}:${IMAGE_TAG}
                    '''
                }
            }
        }
        stage('Push image in staging and deploy it') {
            when {
                    expression { GIT_BRANCH == 'origin/master' }
                    }
            agent any
            environment {
                HEROKU_API_KEY = credentials('HEROKU_GEDS')
            }  
            steps {
                script {
                    sh '''
                    export http_proxy="${HTTP_PROXY}"
                    export https_proxy="${HTTP_PROXY}"
                    heroku container:login
                    heroku create $STAGING || echo "project already exist"
                    heroku container:push -a $STAGING web
                    heroku container:release -a $STAGING web
                    '''
                }
            }
        }
    }
        stage('Push image in production and deploy it') {
            when {
                expression { GIT_BRANCH == 'origin/main' }
            }
            agent any
            environment {
                HEROKU_API_KEY = credentials('HEROKU_GEDS')
                }
                steps {
                    script {
                        sh '''
                        export http_proxy="${HTTP_PROXY}"
                        export https_proxy="${HTTP_PROXY}"
                        heroku container:login
                        heroku create $PRODUCTION || echo "project already exist"
                        heroku container:push -a $PRODUCTION web
                        heroku container:release -a $PRODUCTION web
                        '''
                    }
                }
            }
        }
    }
}


