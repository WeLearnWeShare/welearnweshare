pipeline{

    agent any

    stages{
        stage('Clone'){
            steps{
                git credentialsId: 'git-credentials', url: 'https://github.com/WeLearnWeShare/welearnweshare.git'
            }
        }
        stage ('Build'){
            steps {
                script {
                def mavenHome = tool name: "Maven-3.9.6", type: "maven"
                def mavenCMD = "${mavenHome}/bin/mvn"
                sh "${mavenCMD} clean package"
                }
            }
        }
        stage('SonarQube analysis') {
            steps {
                script {
                    withSonarQubeEnv('SonarQube 8.9') {
                    def mavenHome = tool name: "Maven-3.9.6", type: "maven"
                    def mavenCMD = "${mavenHome}/bin/mvn"
                    sh "${mavenCMD} sonar:sonar"
                    }
                }
            }
        }
        stage('Upload to Nexus') {
            steps {
                nexusArtifactUploader(
                    artifacts: [[
                        artifactId: 'welearnweshare',
                        classifier: '',
                        file: 'target/welearnweshare.war',
                        type: 'war'
                    ]],
                    credentialsId: 'nexus-credentials',
                    groupId: 'in.welearnweshare',
                    nexusUrl: '13.233.194.92:8081',
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    repository: 'welearnweshare-snapshot',
                    version: '1.0-SNAPSHOT'
                )
            }
        }
        stage ('Deploy') {
            steps {
                sshagent(['tomcat-ssh-agent']) {
                sh 'scp -o StrictHostKeyChecking=no target/welearnweshare.war ec2-user@52.66.208.180:/tmp/'
                sh 'ssh -o StrictHostKeyChecking=no ec2-user@52.66.208.180 "sudo mv /tmp/welearnweshare.war /home/ec2-user/apache-tomcat-9.0.86/webapps/"'
                }
            }
        }
    }
}



