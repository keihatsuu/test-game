pipeline {
    agent any
    enviornment
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
        //Remove if no work since no snyk
        stage('SAST')
        {
            steps
            {
                sh 'echo Running SAST scan wtih snyk...'
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
            steps
            {
                script
                {
                    echo "Pushing image ${IMAGE_NAME}:latest to Dockerhub"
                    docker.withRegestiry('https://registry.hub.docker.com',"${DOCKERHUB_CREDNETIALS}")
                    {
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