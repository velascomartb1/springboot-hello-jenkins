pipeline {
    agent any
    
    tools {
        maven 'Maven-3.9'
    }
    
    environment {
        
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "host.docker.internal:8081"
        NEXUS_REPOSITORY = "maven-releases"
        NEXUS_CREDENTIAL_ID = "nexus-credentials"
        
        
        ARTIFACT_GROUP = "com.example"
        ARTIFACT_ID = "springboot-hello"
        ARTIFACT_VERSION = "1.0.0"
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo '=== Checking out code from GitHub ==='
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                echo '=== Building Spring Boot Application ==='
                script {
                    if (isUnix()) {
                        sh 'mvn clean compile'
                    } else {
                        bat 'mvn clean compile'
                    }
                }
            }
        }
        
        stage('Test') {
            steps {
                echo '=== Running Tests ==='
                script {
                    if (isUnix()) {
                        sh 'mvn test'
                    } else {
                        bat 'mvn test'
                    }
                }
            }
        }
        
        stage('Package') {
            steps {
                echo '=== Packaging Application ==='
                script {
                    if (isUnix()) {
                        sh 'mvn package -DskipTests'
                    } else {
                        bat 'mvn package -DskipTests'
                    }
                }
            }
        }
        
        stage('Publish to Nexus') {
            steps {
                echo '=== Publishing Artifact to Nexus ==='
                script {
                    
                    def pom = readMavenPom file: 'pom.xml'
                    
                    
                    def jarFile = findFiles(glob: "target/*.jar")[0]
                    
                    echo "Artifact: ${jarFile.name}"
                    echo "Group: ${pom.groupId}"
                    echo "Artifact ID: ${pom.artifactId}"
                    echo "Version: ${pom.version}"
                    
                    
                    configFileProvider([configFile(fileId: 'maven-settings', variable: 'MAVEN_SETTINGS')]) {
                        if (isUnix()) {
                            sh "mvn deploy -s $MAVEN_SETTINGS -DskipTests"
                        } else {
                            bat "mvn deploy -s %MAVEN_SETTINGS% -DskipTests"
                        }
                    }
                }
            }
        }
        
        stage('Archive Artifacts') {
            steps {
                echo '=== Archiving Build Artifacts ==='
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }
    }
    
    post {
        success {
            echo '=== Pipeline Completed Successfully! ==='
            echo "Artifact published to Nexus at: ${NEXUS_PROTOCOL}://${NEXUS_URL}/repository/${NEXUS_REPOSITORY}/"
        }
        failure {
            echo '=== Pipeline Failed! ==='
        }
        always {
            echo '=== Cleaning up workspace ==='
            cleanWs()
        }
    }
}