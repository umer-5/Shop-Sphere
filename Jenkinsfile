pipeline {
    agent any

    environment {
        COMPOSE_PROJECT_NAME = 'shop-sphere-jenkins'
        COMPOSE_FILE = 'docker-compose.yaml'
        TEST_IMAGE = 'shop-sphere-tests-image'
        CONTAINER_NAME = 'shop-sphere-tests-container'
    }

    stages {
        stage('Checkout App Repo') {
            steps {
                git branch: 'main', credentialsId: 'github-credentials', url: 'https://github.com/umer-5/Shop-Sphere.git'
            }
        }

        stage('Pre-Cleanup') {
            steps {
                echo 'Cleaning up old containers and volumes (if any)...'
                sh '''
                    docker-compose -p $COMPOSE_PROJECT_NAME -f $COMPOSE_FILE down --volumes || true
                    docker system prune -af || true
                    docker volume prune -f || true
                '''
            }
        }

        stage('Build App Containers') {
            steps {
                echo 'Building Docker containers for the app...'
                sh 'docker-compose -p $COMPOSE_PROJECT_NAME -f $COMPOSE_FILE build --no-cache'
            }
        }

        stage('Verify Frontend Build Output') {
            steps {
                echo 'Checking frontend output directory...'
                sh 'ls -lah my-project/dist || echo "Dist folder not found!"'
            }
        }

        stage('Deploy App') {
            steps {
                echo 'Deploying application containers...'
                sh 'docker-compose -p $COMPOSE_PROJECT_NAME -f $COMPOSE_FILE up -d'
            }
        }

        stage('Clone Selenium Tests') {
            steps {
                git url: 'https://github.com/umer-5/Shop-Sphere-Tests.git', branch: 'main'
            }
        }

        stage('Build Selenium Test Image') {
            steps {
                script {
                    docker.build(env.TEST_IMAGE, '.')
                }
            }
        }

        stage('Run Selenium Tests') {
            steps {
                script {
                    sh """
                        docker run --rm --name ${env.CONTAINER_NAME} \
                          ${env.TEST_IMAGE}
                    """
                }
            }
        }

        stage('Test Cleanup') {
            steps {
                script {
                    sh "docker rm -f ${env.CONTAINER_NAME} || true"
                }
            }
        }
    }

    post {
        always {
            echo '🧪 Pipeline execution completed.'
        }
        success {
            echo '✅ All stages completed successfully.'
        }
        failure {
            echo '❌ Pipeline failed at one or more stages.'
        }
    }
}
