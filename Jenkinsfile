//variables
def network='jenkins-${BUILD_NUMBER}'
def seleniumHub='selenium-hub-${BUILD_NUMBER}'
def chrome='chrome-${BUILD_NUMBER}'
def firefox='firefox-${BUILD_NUMBER}'
def containertest='conatinertest-${BUILD_NUMBER}'

pipeline {
    agent {
        node {
            label 'jenkins-jnlp-with-docker'
        }
    }
    stages {
        stage('Build Jar') {
            steps {
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


        stage('Setting Up Selenium Grid') {
            steps{
                sh "docker network create ${network}"
                sh "docker run -d -p 4444:4444 --name ${seleniumHub} --network ${network} selenium/hub"
                sh "docker run -d -e HUB_PORT_4444_TCP_ADDR=${seleniumHub} -e HUB_PORT_4444_TCP_PORT=4444 --network ${network} --name ${chrome} selenium/node-chrome"
                sh "docker run -d -e HUB_PORT_4444_TCP_ADDR=${seleniumHub} -e HUB_PORT_4444_TCP_PORT=4444 --network ${network} --name ${firefox} selenium/node-firefox"
            }
        }
        stage('Run Test') {
            steps{
                parallel(
                        "search-module":{
                            // a directory 'search' is created for container test-output
                            sh "docker run --rm -e SELENIUM_HUB=${seleniumHub} -e BROWSER=firefox -e MODULE=search-module.xml -v ${WORKSPACE}/search:/usr/share/tag/test-output --network ${network} vinsdocker/containertest"
                            //archive all the files under 'search' directory
                            archiveArtifacts artifacts: 'search/**', fingerprint: true
                        },
                        "order-module":{
                            // a directory 'order' is created for container test-output
                            sh "docker run --rm -e SELENIUM_HUB=${seleniumHub} -e BROWSER=chrome -e MODULE=order-module.xml -v ${WORKSPACE}/order:/usr/share/tag/test-output  --network ${network} vinsdocker/containertest"
                            //archive all the files under 'order' directory
                            archiveArtifacts artifacts: 'order/**', fingerprint: true
                        }
                )
            }
        }
        stage('Tearing Down Selenium Grid') {
            steps {
                //remove all the containers and volumes
                sh "docker rm -vf ${chrome}"
                sh "docker rm -vf ${firefox}"
                sh "docker rm -vf ${seleniumHub}"
                sh "docker network rm ${network}"
            }
        }
    }

}