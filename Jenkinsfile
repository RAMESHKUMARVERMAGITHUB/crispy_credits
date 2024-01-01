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
                git branch: 'main', url: 'https://github.com/RAMESHKUMARVERMAGITHUB/crispy_credits.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=crispy_credits \
                    -Dsonar.projectKey=crispy_credits'''
                }
            }
        }
        stage("quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar'
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
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){
                       sh "docker build -t rameshkumarverma/crispy_credits:latest ."
                       // sh "docker tag uber rameshkumarverma/crispy_credits:latest "
                       sh "docker push rameshkumarverma/crispy_credits:latest"
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image rameshkumarverma/crispy_credits:latest > trivyimage.txt"
            }
        }
        // stage("deploy_docker"){
        //     steps{
        //         sh "docker run -d --name uber -p 3000:3000 rameshkumarverma/crispy_credits:latest"
        //     }
        // }
        stage("Deploy"){
            steps {
                echo "Deploying the container"
                sh "docker-compose down && docker-compose up -d"
                
            }
        }
        // stage('Deploy to kubernets'){
        //     steps{
        //         script{
        //             // dir('K8S') {
        //                 withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
        //                         sh 'kubectl apply -f deployment-service.yml'
        //                         // sh 'kubectl apply -f service.yml'
        //                 }
        //             // }
        //         }
        //     }
        // }
    }
}
