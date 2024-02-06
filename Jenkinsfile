pipeline {

    agent any

    tools { 
        maven 'maven-tool' 
    }
    environment {
        MYSQL_ROOT_LOGIN = credentials('mysql-account')
    }
    stages {

        stage('Build with Maven') {
            steps {
                sh 'mvn --version'
                sh 'java -version'
                sh 'mvn clean package -Dmaven.test.failure.ignore=true'
                sh 'docker --version'
            }
        }

        stage('Packaging/Pushing imagae') {

            steps {
                withDockerRegistry(credentialsId: 'docker-account') {
                    sh 'echo 123'
                    sh 'docker build -t quickybooster/springboot .'
                    sh 'docker push quickybooster/springboot'
                }
            }
        }

        stage('Deploy MySQL to DEV') {
            steps {
                echo 'Deploying and cleaning'
                sh 'docker image pull mysql:8.0'
                sh 'docker network create dev || echo "this network exists"'
                sh 'docker container stop booster-mysql || echo "this container does not exist" '
                sh 'echo y | docker container prune '
                sh 'docker volume rm booster-mysql-data || echo "no volume"'

                sh "docker run --name booster-mysql --rm --network dev -v booster-mysql-data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_LOGIN_PSW} -e MYSQL_DATABASE=db_example  -d mysql:8.0 "
                sh 'sleep 20'
                sh "docker exec -i booster-mysql mysql --user=root --password=${MYSQL_ROOT_LOGIN_PSW} < script"
            }
        }

        stage('Deploy Spring Boot to DEV') {
            steps {
                echo 'Deploying and cleaning'
                sh 'docker image pull quickybooster/springboot'
                sh 'docker container stop booster-springboot || echo "this container does not exist" '
                sh 'docker network create dev || echo "this network exists"'
                sh 'echo y | docker container prune '

                sh 'docker container run -d --rm --name booster-springboot -p 8081:8080 --network dev quickybooster/springboot'
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
