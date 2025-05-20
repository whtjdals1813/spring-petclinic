pipeline {
    agent any
    
    tools {
        maven "M3"
        jdk "JDK21"
    }
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerCredential')
        REGION = "ap-northeast-2"
        AWS_CREDENTIAL_NAME = "AWSCredentials"
    }
    
    stages {
        stage('Git Clone') {
            steps {
                echo 'Git Clone'
                git url: 'https://github.com/whtjdals1813/spring-petclinic.git',
                branch: 'main'
            }
            post {
                success {
                    echo 'Git Clone Success'
                }
                failure {
                    echo 'Git Clone Fail'
                }
            }
        }
        stage('Maven Build') {
            steps {
                echo 'Maven Build'
                sh 'mvn -Dmaven.test.failure.ignore=true clean package'
            }
            //maven build 작업
        }
        // docker Image  생성
        stage('Docker Image Build') {
            steps {
                echo 'Docker Image Build'
                dir("$env.WORKSPACE") {
                    sh '''
                       docker build -t spring-petclinic:$BUILD_NUMBER .
                       docker tag spring-petclinic:$BUILD_NUMBER joseongmin/spring-petclinic:latest
                       '''
                }
            }
        }
        // docker Image push
        stage('Docker Image Push') {
            steps {
                sh '''
                   echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
                   docker push joseongmin/spring-petclinic:latest
                   '''
            }
        }
        // Remote Docker Image
        stage('Remove Docker Image') {
            steps {
                sh '''
                   docker rmi spring-petclinic:$BUILD_NUMBER
                   docker rmi joseongmin/spring-petclinic:latest
                   '''
            }
        }
    }
}
