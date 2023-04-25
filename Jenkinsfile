pipeline {
    agent {
      label 'worker-node'
    }
    
    environment {
        // Define your environment variables here
        NEXUS_URL = "https://nexus.pwasil.pl/repository/maven-snapshots-custom/"
        NEXUS_CREDENTIALS_ID = "nexus-credentials"
        DOCKERHUB_REGISTRY = "https://index.docker.io/v1/"
        DOCKERHUB_CREDENTIALS_ID = "dockerhub-credentials"
    }
    
    stages {
        stage('Build and Test') {
            steps {
                sh './gradlew clean bootJar test'
            }
            post {
                success {
                    junit 'build/test-results/**/*.xml'
                    archiveArtifacts 'build/libs/*.jar'
                }
            }
        }
        
        stage('Upload Artifacts to Nexus') {
            steps {
                script {
                    nexusArtifactUploader(
                        nexusVersion: 'nexus3',
                        protocol: 'http',
                        nexusUrl: env.NEXUS_URL,
                        groupId: 'com.example',
                        version: '1.0.0-SNAPSHOT',
                        repository: 'snapshots',
                        credentialsId: env.NEXUS_CREDENTIALS_ID,
                        artifacts: [
                            [
                                artifactId: 'petclinic',
                                type: 'jar',
                                classifier: '',
                                file: 'build/libs/petclinic-3.0.0-SNAPSHOT.jar'
                            ]
                        ]
                    )
                }
            }
        }
        
        stage('Build and Push Docker Image') {
            steps {
                script {
                    docker.withRegistry(env.DOCKERHUB_REGISTRY, env.DOCKERHUB_CREDENTIALS_ID) {
                        def imageTag = "petclinic:1.0.0-SNAPSHOT"
                        def dockerfile = 'Dockerfile'
                        def context = '.'
                        
                        docker.build(imageTag, "-f ${dockerfile} ${context}")
                        docker.push(imageTag)
                    }
                }
            }
        }
    }
}
