pipeline {
    agent any
    environment
    {
        //Docker hub credentials ID stored in Jenkins
        DOCKERHUB_CREDENTIALS ='cybr-3120'
        IMAGE_NAME ='keihatsu/test-game'
    }
    
    stages
    {
        stage('Cloning Git')
        {
            steps
            {
                checkout scm
            }
        }
        stage('SAST-TEST')
        {
            agent { 
                label 'hello-world-soto'
            }
            steps
            {
                script
                {
                    snykSecurity(
                        snykInstallation: 'Snyk-Installations',
                        snykTokenId: 'Snyk-API-Token',
                        severity: 'critical'
                    )
                }
                sh 'echo Running SAST scan wtih snyk...'
            }    
        }
        stage('SonarQube Analysis') {
            agent {
                label 'hello-world-soto'
            }
            steps {
                script {
                    def scannerHome = tool 'SonarQube-Scanner'
                    withSonarQubeEnv('SonarQube-Installations') {
                        sh "${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=gameapp \
                            -Dsonar.sources=."
                    }
                }
            }
        }
        stage('BUILD-AND-TAG')
        {
            agent { 
                label 'hello-world-soto'
            }
            steps {
                script
                {
                    // Build Docker image using Jenkins Pipeline API
                    echo "Building Docker image ${IMAGE_NAME}..."
                    app = docker.build("${IMAGE_NAME}")
                    app.tag("latest")
                }
            }
        }
        stage('POST-TO-DOCKERHUB')
        {
            agent { label 'hello-world-soto' }
            steps {
                script {
                    echo "Pushing image ${IMAGE_NAME}:latest to Docker Hub..."
                    docker.withRegistry('https://registry.hub.docker.com', "${DOCKERHUB_CREDENTIALS}") {
                        app.push("latest")
                    }
                }
            }
        }
        //REMOVE IF NO WORK CUZ NO SNYK
        stage('DAST')
        {
            steps
            {
                sh 'echo Running DAST scan...'
            }
        }
        stage('DEPLOYMENT')
        {
            agent { label 'hello-world-soto' }
            steps
            {
                echo 'Starting deployment using docker-compose...'
                script
                {
                    dir("${WORKSPACE}")
                    {
                        sh '''
                            docker-compose down
                            docker-compose up -d
                            docker ps
                        '''
                    }
                }
                echo 'Deployment completed succcessfully'
            }
        }
    }
}
