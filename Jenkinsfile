pipeline {
    agent any
    tools {
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

        stage("Checkout from Git") {
            steps{
                git branch: 'main', url: 'https://github.com/obusorezekiel/Amazon-FE.git'
            }
        }

        // stage("Sonarqube Analysis "){
        //     steps{
        //         withSonarQubeEnv('sonar-server') {
        //             sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Amazon \
        //             -Dsonar.projectKey=Amazon'''
        //         }
        //     }
        // }
        // stage("quality gate"){
        //    steps {
        //         script {
        //             waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-Token'
        //         }
        //     }
        // }
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
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){
                       sh "docker build -t amazon ."
                       sh "docker tag amazon obusorezekiel/amazon:latest "
                       sh "docker push obusorezekiel/amazon:latest "
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image obusorezekiel/amazon:latest > trivyimage.txt"
            }
        }
        stage("deploy_docker"){
            steps{
                sh "docker stop amazon || true"  // Stop the container if it's running, ignore errors
                sh "docker rm amazon || true" 
                sh "docker run -d --name amazon -p 4000:3000 obusorezekiel/amazon:latest"
            }
        }
    }
}