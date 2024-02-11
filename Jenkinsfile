
pipeline{
    agent any
    tools{
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                git branch: 'main', url: 'https://github.com/Boubamahir2/DevSecOps-Project.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix \
                    -Dsonar.projectKey=Netflix '''
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
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                  withDockerRegistry(credentialsId: 'docker-token', toolName: 'docker'){   
                      sh "docker build --build-arg TMDB_V3_API_KEY=61e43bb1d22eed4a6e569f505bf40cb8 -t netflix ."
                      sh "docker tag netflix boubamahir/netflix:latest "
                      sh "docker push boubamahir/netflix:latest "
                  }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image boubamahir/netflix:latest > trivyimage.txt" 
            }
        }
        stage('Remove Existing Container'){
            steps{
                script {
                    // Stop and remove the existing container if it exists
                    sh 'docker rm -f netflix || true'
                }
    }
}
        
        stage('Deploy to container'){
            steps{
                sh 'docker run -d --name netflix -p 8081:80 boubamahir/netflix:latest'
            }
        }
    }
    
    post {
      always {
        emailext attachLog: true,
          subject: "`${currentBuild.result}` ",
          body : "Project: ${env.JOB_NAME} <br/>" +
                  "Build Number: ${env.BUILD_NUMBER} <br/>" +
                  "URL: ${env.BUILD_URL} " ,
          to : "amamahir2@gmail.com",
          attachmentsPattern: "trivyfs.txt, trivyimage.txt"
      }
    }
}


