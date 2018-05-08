#!/usr/bin/env groovy

node {
    stage('checkout') {
        checkout scm
    }

    

        stage('clean') {
            sh "chmod +x mvnw"
            sh "./mvnw clean"
        }

        stage('backend tests') {
            try {
                sh "./mvnw test"
            } catch(err) {
                throw err
            } finally {
                junit '**/target/surefire-reports/TEST-*.xml'
            }
        }

        stage('package and deploy') {
            sh "./mvnw com.heroku.sdk:heroku-maven-plugin:1.1.1:deploy -DskipTests -Pprod -Dheroku.appName="
            archiveArtifacts artifacts: '**/target/*.war', fingerprint: true
        }

        stage('quality analysis') {
            withSonarQubeEnv('Sonar') {
                sh "./mvnw sonar:sonar"
            }
        }
    }

    def dockerImage
    stage('build docker') {
        sh "cp -R src/main/docker target/"
        sh "cp target/*.war target/docker/"
        dockerImage = docker.build('docker-login/devoxx', 'target/docker')
    }

    stage('publish docker') {
        docker.withRegistry('https://registry.hub.docker.com', 'docker-login') {
            dockerImage.push 'latest'
        }
    }
}
