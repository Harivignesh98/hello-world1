pipeline {
    agent {label 'test'}
    options {
        buildDiscarder (logRotator(numToKeepStr: '1'))
        timeout (unit: 'MINUTES', time: 5)
    }
    stages {
        stage('checkout scm') {
            steps {
                git branch: 'master', credentialsId:'', url: 'https://github.com/jay19888/hello-world-1.git'
            }
        }
        stage ('code analysis'){
            steps {
                withSonarQubeEnv('sonarqube'){
                    sh "mvn clean verify sonar:sonar -Dsonar.projectKey=devops_project"
                }

            }
        }
        stage ('build the binaries'){
            steps {
                sh "mvn clean install"
            }
        }
        stage ('artifact the builds'){
            steps{
                rtUpload (
                    serverId: 'artifact',
                    spec: """{
                        "files": [
                            {
                                "pattern": "webapp/target/*.war",
                                "target": "devops_project/snapshot-1.0-${BUILD_NUMBER}/"
                            }
                        ]
                    }"""
                )
            }
        }
        stage ('deploy'){
            steps {
                deploy adapters: [tomcat9(credentialsId: 'tomcat_server', path: '', url: 'http://13.126.241.51:8080/')], contextPath: null, war: 'webapp/target/*.war'
            }
        }
        stage ('deploy_ansible'){
            steps {
                sshPublisher(publishers: [sshPublisherDesc(configName: 'ansible', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: 'cd ansible', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: 'ansible/', remoteDirectorySDF: false, removePrefix: 'webapp/target/', sourceFiles: 'webapp/target/*.war')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false), sshPublisherDesc(configName: 'ansible', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: '', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: 'ansible/', remoteDirectorySDF: false, removePrefix: '', sourceFiles: 'Dockerfile')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false), sshPublisherDesc(configName: 'ansible', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: '''cd ansible
                docker build -t demo .
                docker tag demo jayn21/demo:latest
                docker login -u jayn21 -p d0cker123
                docker push jayn21/demo:latest
                docker rmi demo jayn21/demo
                ansible-playbook deploy.yml
                ''', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: 'ansible/', remoteDirectorySDF: false, removePrefix: '', sourceFiles: 'deploy.yml')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
            }
        }
    }
}
