pipeline {
    agent any
    
    environment {
        // Define environment variables
        DOCKER_REGISTRY = 'usfomysis'
        DOCKER_IMAGE_NAME = '05-jenkins-cicd'
        DOCKER_CREDENTIALS_ID = 'dockerhub-id'
        SONARQUBE_SERVER = 'localhost:9000'
        SONARQUBE_TOKEN_ID = 'sonar-token	'
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
                withCredentials([string(credentialsId: SONARQUBE_TOKEN_ID, variable: 'SONARQUBE_TOKEN')]) {
                    sh """
                        sonar-scanner \
                        -Dsonar.projectKey="YourProjectKey" \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=${SONARQUBE_SERVER} \
                        -Dsonar.login=${SONARQUBE_TOKEN}
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
    // }
    
}