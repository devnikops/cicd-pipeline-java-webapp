pipeline {
    agent {
        label 'QA'
    }
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    environment {
        ArtifactId = readMavenPom().getArtifactId()
        Version = readMavenPom().getVersion()
        GroupId = readMavenPom().getGroupId()
        Name = readMavenPom().getName()
    }
    stages {
        stage('Cleanup Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout from SCM') {
               steps {
                   git branch: 'main', credentialsId: 'Git-Github-token', url: 'https://github.com/Devnikops/cicd-pipeline-java-webapp.git'
               }
        }

        stage('Build Project') {
            steps {
                sh 'mvn clean install package'
            }
        }

       stage('Publish to Nexus') {
            steps { 
                script {
                    def NexusRepo = Version.endsWith("SNAPSHOT") ? "MyLab-SNAPSHOT" : "MyLab-RELEASE"
                    
                    nexusArtifactUploader artifacts: [
                        [
                            artifactId: '${ArtifactId}', classifier: '', file: 'target/${ArtifactId}-${Version}.war', type: 'war'
                        ]
                    ], 
                    credentialsId: 'nexus', 
                    groupId: '${GroupId}', 
                    nexusUrl: '35.154.158.124:8081', 
                    nexusVersion: 'nexus3', 
                    protocol: 'http', 
                    repository: '${NexusRepo}', 
                    version: '${Version}'              
                }
            }
        }
        stage('Print Environment variables') {
            steps {
                echo "Artifact ID is '${ArtifactId}'"
                echo "Group ID is '${GroupId}'"
                echo "Version is '${Version}'"
                echo "Name is '${Name}'"
            }
        }
        /*
        stage('Deploy to Docker') {
            steps {
                echo 'Deploying...'
                sshPublisher(publishers: 
                [sshPublisherDesc(
                    configName: 'ansible-controller', 
                    transfers: [
                        sshTransfer(
                            sourceFiles: 'download-deploy.yaml, hosts',
                            remoteDirectory: '/playbooks',
                            cleanRemote: false,
                            execCommand: 'cd playbooks/ && ansible-playbook download-deploy.yaml -i hosts', 
                            execTimeout: 120000, 
                        )
                    ], 
                    usePromotionTimestamp: false, 
                    useWorkspaceInPromotion: false, 
                    verbose: false)
                ])
            }
        }*/
    }
    post {
        failure {
            emailext(
                to: 'devopswithnik@gmail.com',
                subject: "Build Failed: ${currentBuild.fullDisplayName}",
                body: "This build has failed.",
                attachLog: true
            )
        }
    }
}