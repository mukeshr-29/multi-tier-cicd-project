pipeline{
    agent any
    tools{
        jdk 'jdk17'
        maven 'maven3'
    }
    environment{
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages{
        stage('git checkout'){
            steps{
                git branch: 'main', url: 'https://github.com/mukeshr-29/multi-tier-cicd-project.git'
            }
        }
        stage('code compilation'){
            steps{
                sh 'mvn compile'
            }
        }
        stage('code testing'){
            steps{
                sh 'mvn test -DskipTest=true'
            }
        }
        stage('trivy file scan'){
            steps{
                sh 'trivy fs --format table -o fs-report.html .'
            }
        }
        stage('sonarqybe analysis'){
            steps{
                script{
                    withSonarQubeEnv('sonar-server'){
                        sh '''
                            $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Multitier -Dsonar.projectKey=Multitier \
                                -Dsonar.java.binaries=target
                        '''
                    }
                }
            }
        }
        stage('code build'){
            steps{
                sh 'mvn package -DskipTest=true'
            }
        }
        stage('publish to nexus'){
            steps{
                withMaven(globalMavenSettingsConfig: 'settings.xml', jdk: '', maven: 'maven3', mavenSettingsConfig: '', traceability: true){
                    sh 'mvn deploy -DskipTest=true'
                }
            }
        }
        stage('Docker image build'){
            steps{
                sh 'docker build -t mukeshr29/bankapp:latest .'
            }
        }
        stage('trivy img scan'){
            steps{
                sh 'trivy image --format table -o img-report.html mukeshr29/bankapp:latest'
            }
        }
        stage('docker image push'){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){
                        sh 'docker push mukeshr29/bankapp:latest'
                    }
                }
            }
        }
        stage('deploy to k8s'){
            steps{
                script{
                    withKubeConfig(caCertificate: '', clusterName: 'my-app', contextName: '', credentialsId: 'k8s', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://15CA6F2B70BA3FB50BAEBCBEAC40F1F8.gr7.us-east-1.eks.amazonaws.com'){
                        sh 'kubectl apply -f dev-sev.yml'
                        sleep 30
                    }
                }
            }
        }
        stage('verify deployment'){
            steps{
                script{
                    withKubeConfig(caCertificate: '', clusterName: 'my-app', contextName: '', credentialsId: 'k8s', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://15CA6F2B70BA3FB50BAEBCBEAC40F1F8.gr7.us-east-1.eks.amazonaws.com'){
                        sh 'kubectl get pods'
                        sh 'kubectl get svc'
                    }
                }
            }
        }
    }
}