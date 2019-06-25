pipeline {
    agent {
        node {
            label 'jenkins-jnlp-with-docker'
        }
    }
    stages {
        stage('Build Jar') {
            steps {
                cleanWs()
                checkout scm

                sh '/opt/apache-maven-3.6.0/bin/mvn clean package -DskipTests'
            }
        }
        stage('Build Image') {
            steps {
                script {
                    app = docker.build("nashpaz1/containertest")
                }
            }
        }
        stage('Push Image') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
                        app.push("${BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
    }

}