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
        /* stage ('start sonar container') {
            steps {
                sh 'docker start sonar'
            }
        } */
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
        /* stage("Deploy with compose"){
            steps{
                sh "docker-compose up -d"
            }
        } */
        stage('Deploy with k8s') {
            steps {
                script {
                    withKubeConfig([credentialsId: 'mykubeconfig']) {
                        sh 'minikube status'
                        sh 'kubectl apply -f k8s/'
                        //sh 'kubectl apply -f k8s/mongo-deployment.yml'
                        //sh 'kubectl apply -f k8s/spring-deployment.yml'   
                    }
                }
            }
        }
        
        //stage ('Deploy to container'){
            //steps{
                //sh 'docker run -d --name deploytest -p 9090:9090 drugman21/student:latest'
            //}
        //}
        
        
   }
   post {
     always {
        emailext attachLog: true,
            subject: "'${currentBuild.result}'",
            body: "Project: ${env.JOB_NAME}<br/>" +
                "Build Number: ${env.BUILD_NUMBER}<br/>" +
                "URL: ${env.BUILD_URL}<br/>",
            to: 'miladi.wael21@gmail.com',
            attachmentsPattern: 'trivy.txt'
        }
    }
}
