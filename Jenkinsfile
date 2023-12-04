pipeline{
    agent any
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages{
        stage ('clean Workspace'){
            steps{
                cleanWs()
            }
        }
        stage ('checkout scm') {
            steps {
                git 'https://github.com/waelmiladi21/mongo-demo.git'
            }
        }
        stage ('maven compile') {
            steps {
                sh 'mvn clean compile'
            }
        }
        stage ('maven Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=student \
                    -Dsonar.java.binaries=. \
                    -Dsonar.projectKey=student '''
                }
            }
        }
        stage("quality gate"){
            steps {
                script {
                  waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token' 
                }
           }
        }
        stage ('Build jar file'){
            steps{
                sh 'mvn clean package -DskipTests=true'   
            }
        }
        stage("OWASP Dependency Check"){
            steps{
                dependencyCheck additionalArguments: '--scan ./ --format XML ', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage ('Build and push to docker hub'){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                        sh 'docker build -t drugman21/student .'
                        //sh 'docker build -t waelmiladi21/mongo-demo .'
                        //sh "docker tag student drugman21/student:latest"
                        sh "docker push drugman21/student:latest"
                   }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image drugman21/student:latest > trivy.txt"
            }
        }
        stage ('Deploy to container'){
            steps{
                sh 'docker run -d --name student -p 8080:8080 drugman21/student:latest'
            }
        }
   }
}
