pipeline {
    agent any
    
    tools {
        maven "M3"
        jdk "JDK17"
    }
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerCredential')
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
        stage('SSH Publish') {
            steps {
                echo 'ssh Puplish'
                sshPublisher(publishers: [sshPublisherDesc(configName: 'target', 
                transfers: [sshTransfer(cleanRemote: false, excludes: '', 
                execCommand: '''
                docker rm -f $(docker ps -aq)
                docker rmi $(docker images -q)
                docker run -d -p 8080:8080 --name spring-petclinic joseongmin/spring-petclinic:latest
                ''',
                execTimeout: 120000, 
                flatten: false, 
                makeEmptyDirs: false, 
                noDefaultExcludes: false, 
                patternSeparator: '[, ]+', 
                remoteDirectory: '', 
                remoteDirectorySDF: false, 
                removePrefix: 'target', 
                sourceFiles: 'target/*.jar')], 
                usePromotionTimestamp: false, 
                useWorkspaceInPromotion: false, verbose: false)])

            }
        }
    }
}
