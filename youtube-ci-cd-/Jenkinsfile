pipeline {
    agent any
    tools {

        nodejs 'NodeJs'
    }
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        DOCKERHUB_CREDENTIALS = credentials('Docker_hub')
        GITHUB_REPO_URL = 'https://github.com/curlsysolange/youtube-ci-cd-.git'
    }
    
    stages {
        stage('Clean workspace') {
            steps {
                cleanWs()
            }
        }
    } 
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/curlsysolange/youtube-ci-cd-.git'
            }
        }
        
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh "$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Youtube -Dsonar.projectKey=Youtube"
                }
            }
        } 

        stage('NPM') {
            steps {
                sh 'npm install'
            }
        }
        

        stage('Quality Gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-Token'
                }
            }
        }
        
        stage('Trivy File Scan') {
            steps {
                script {
                    sh '/usr/local/bin/trivy fs . > trivy_result.txt'
                }
            }
        }

        stage('OWASP FILE SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit --nvdApiKey 4bdf4acc-8eae-45c1-bfc4-844d549be812', odcInstallation: 'DP'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('Login to DockerHUB') {
            steps {
                script {
                    sh "echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin"
                    echo 'Login Succeeded'
                }
            }
        }
        stage('Docker Build') {
            steps {
                script {
                    sh 'docker build -t curlsysolange/youtube:latest .'
                    echo "Image Build Successfully"
                }
            }
        }
     
        stage('Docker Push') {
            steps {
                script {
                    sh 'docker push curlsysolange/youtube:latest'
                    echo "Push Image to Registry"
                }
            }
        } 
             
        stage('Trivy image Scan') {
            steps {
                script {
                    sh 'trivy fs . > trivy_result.txt'  // Assuming Trivy is properly installed
                }
            }
        }
        
             stage('Containerization Deployment') {
            steps {
                script {
                    def containername = 'youtube'
                    def isRunning = sh(script: "docker ps -a | grep ${containername}", returnStatus: true)
                    if (isRunning == 0) {
                        sh "docker stop ${containername}"
                        sh "docker rm ${containername}"
                    }
                    sh "docker run -d -p 3000:3000 --name ${containername} curlsysolange/youtube:latest"
                }
            }
        }
      /*  
      stage('Deploy to Kubernetes') {
            steps {
                withKubeConfig([credentialsId: 'kubeconfig']) {
                    sh 'kubectl apply -f deployment.yml'
                    sh 'kubectl apply -f service.yml'
                }
            }
      }
      */
        stage('cluster Scan') {
            steps {
                withKubeConfig([credentialsId: 'kubeconfig']) {
                    // Run Trivy vulnerability scan on your container images
                   sh 'kubectl get pods --all-namespaces -o=jsonpath=\"ef84f56fc686 \" | tr \' \' \'\\n\' | sort -u | xargs -n1 trivy image --format=json'
                }
                // Publish the results as an artifact
            }
        }
}