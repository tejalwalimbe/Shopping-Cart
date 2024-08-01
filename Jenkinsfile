pipeline {
    agent any
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'master', changelog: false, poll: false, url: 'https://github.com/Sunilmargale/Shopping-Cart.git'
            }
        }
        
        stage('COMPILE') {
            steps {
                sh "mvn clean compile -DskipTests=true"
            }
        }
        
        stage('OWASP Scan') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ ', odcInstallation: 'DP'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('Sonarqube') {
            steps {
                withSonarQubeEnv('sonar-scanner') {
                    sh ''' 
                    $SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.url=http://13.126.29.152:9000/ \
                    -Dsonar.login=squ_0c5308610f3737566f8e837d617bbae614387f7f \
                    -Dsonar.projectName=Shopping-Cart \
                    -Dsonar.java.binaries=. \
                    -Dsonar.projectKey=Shopping-Cart 
                    '''
                }
            }
        }
        
        stage('Build') {
            steps {
                sh "mvn clean package -DskipTests=true"
            }
        }
        
        stage('Docker Build & Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker build -t shopping-cart -f docker/Dockerfile ."
                        sh "docker run -dp --name shopping 8070:8070 shopping-cart"
                        sh "docker tag shopping-cart sunilmargale/shopping-cart:latest"
                        sh "docker push sunilmargale/shopping-cart:latest"
                    }
                }
            }
        }
    }
}
