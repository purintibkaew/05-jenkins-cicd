pipeline {
    agent any
    tools { 
        nodejs "NodeJS 16" 
        // sonarQubeScanner 'sonar' 

        // hudson.plugins.sonar.SonarRunnerInstallation 'sonar'
    }
    environment {
        // Define environment variables
        DOCKER_REGISTRY = 'ufomysis'
        DOCKER_IMAGE_NAME = '05-jenkins-cicd'
        DOCKER_CREDENTIALS_ID = 'dockerhub-id'
        SONARQUBE_SERVER = 'localhost:9000'
        SONARQUBE_TOKEN_ID = 'sonar-token'
        WORKSPACE = '/home/jenkins/agent/workspace/05-jenkins-cicd'
        // KUBECONFIG_CREDENTIALS_ID = 'your_kubeconfig_credentials_id'
    }
    
    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Unit Tests') {
            steps {
                script {
                    sh 'npm install'
                    sh 'npm run test:ci'
                    junit '**/coverage/junit.xml'
                }
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv(credentialsId: 'sonar-token', installationName: 'sonar-server') {
                    sh """
                        sonar-scanner \
                        -Dsonar.projectKey="05-jenkins-cicd" \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=${SONARQUBE_SERVER} \
                        -Dsonar.projectBaseDir=${WORKSPACE}
                    """
                }
            }
        }
        
        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: '--project "05-jenkins-cicd" --scan ./ --format "HTML"',
                               odcInstallation: 'default'
            }
            post {
                always {
                    dependencyCheckPublisher pattern: '**/dependency-check-report.html'
                }
            }
        }
        
        // stage('Build and Push Docker Image') {
        //     steps {
        //         script {
        //             withCredentials([usernamePassword(credentialsId: DOCKER_CREDENTIALS_ID, usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
        //                 sh 'echo $DOCKER_PASSWORD | docker login $DOCKER_REGISTRY -u $DOCKER_USERNAME --password-stdin'
        //                 sh 'docker build -t $DOCKER_REGISTRY/$DOCKER_IMAGE_NAME .'
        //                 sh 'docker push $DOCKER_REGISTRY/$DOCKER_IMAGE_NAME'
        //             }
        //         }
        //     }
        // }
        
    //     stage('Deploy to Kubernetes') {
    //         steps {
    //             script {
    //                 withCredentials([file(credentialsId: KUBECONFIG_CREDENTIALS_ID, variable: 'KUBECONFIG')]) {
    //                     sh 'kubectl apply -f your_kubernetes_deployment_manifest.yaml'
    //                     sh 'kubectl rollout status deployment/your-deployment-name'
    //                 }
    //             }
    //         }
    //     }
    }
    
}

