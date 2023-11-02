pipeline {

    agent any

    tools { 
        maven 'campscholar' 
    }
    environment {
        MYSQL_ROOT_LOGIN = credentials('mysql-root-login')
    }
    stages {

        stage('Build with Maven') {
            steps {
                sh 'mvn --version'
                sh 'java -version'
                sh 'mvn clean package -Dmaven.test.failure.ignore=true'
            }
        }

        stage('Packaging/Pushing imagae') {

            steps {
                withDockerRegistry(credentialsId: 'dockerhub', url: 'https://index.docker.io/v1/') {
                    sh 'docker build -t hoangvh2388/campscholar .'
                    sh 'docker push hoangvh2388/campscholar'
                }
            }
        }

        stage('Deploy MySQL to DEV') {
            steps {
                echo 'Deploying and cleaning'
                sh 'docker image pull mysql:8.0'
                sh 'docker network create dev || echo "this network exists"'
                sh 'docker container stop hoangvh238-mysql || echo "this container does not exist" '
                sh 'echo y | docker container prune '
                sh 'docker volume rm hoangvh238-mysql-data || echo "no volume"'

                sh "docker run --name hoangvh238-mysql --rm --network dev -v hoangvh238-mysql-data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_LOGIN_PSW} -e MYSQL_DATABASE=db_campscholar  -d mysql:8.0 "
                sh 'sleep 20'
                sh "docker exec -i hoangvh238-mysql mysql --user=root --password=${MYSQL_ROOT_LOGIN_PSW} < script"
            }
        }

        stage('Deploy Spring Boot to DEV') {
            steps {
                echo 'Deploying and cleaning'
                sh 'docker image pull hoangvh2388/campscholar'
                sh 'docker container stop campscholar-springboot || echo "this container does not exist" '
                sh 'docker network create dev || echo "this network exists"'
                sh 'echo y | docker container prune '

                sh 'docker container run -d --rm --name campscholar -p 8082:8080 --network dev hoangvh2388/campscholar'
            }
        }
 
    }
    post {
        // Clean after build
        always {
            cleanWs()
        }
    }
}
