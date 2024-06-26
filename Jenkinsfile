pipeline{
    agent any
    tools{
        jdk 'JAVA_HOME'
        maven 'MAVEN_HOME'
    }
       environment {
         SCANNER_HOME=tool 'sonar-server'
     }
    stages{
        stage('SCM checkout'){
            steps{
                sh 'echo cloning the repo into the slave machine'
                git branch: 'main', url: 'https://github.com/devopsaws355/github.git'
            }
        }
        stage('compile source code'){
            steps{
                
                sh '''echo compiling the source code
                mvn compile'''
            }
        }
        stage('test compile source code'){
            steps{
                sh '''echo testing the compiled source code using suitable unit testing framework
                mvn test''' 
            }
        }
        stage('build project'){
            steps{
                sh '''echo cleaing the maven projct by deleting any existing target directory and build the project
                mvn clean package
                ls -al target'''
            }
        }
        stage('scan file system'){
            steps{
                sh '''echo scanning the files in the cloned git repository using trivy
                trivy fs --format table -o trivy-fs-report.html .'''
            }
        }
         stage("Sonarqube Analysis"){
           steps{
                withSonarQubeEnv('sonar-server') {
                     sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=javaproject \
                     -Dsonar.projectKey=javaproject \
                     -Dsonar.exclusions=**/*.java
                    '''
                 }
         }
         }
        // stage("Quality Gate"){
        //     steps {
        //         script {
        //             waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token' 
        //         }
        //     } 
        // }
        // stage('OWASP DP SCAN') {
        //     steps {
        //         dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'owasp-dp-check'
        //         dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
        //     }
        // }
        // stage('deploy the artifact to nexus'){
        //     steps{
        //         withMaven(globalMavenSettingsConfig: '', jdk: 'java17', maven: 'maven3', mavenSettingsConfig: 'nexus', traceability: true) {
        //             sh '''echo deploying the build artifact to the nexus repository with version 0.0.${BUILD_NUMBER}
        //             mvn deploy'''
        //         }
        //     }
        // }
        stage('Jfrog Artifact Upload') {
            steps {
              rtUpload (
                serverId: 'jfrog-artifactory',
                spec: '''{
                      "files": [
                        {
                          "pattern": "*.jar",
                           "target": "local-snapshots"
                        }
                    ]
                }'''
              )
          }
        }
            stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage('Build Docker Image') {
            steps {
                script{
                    sh 'docker build -t sandya890/board_game .'
                }
            }
        }
        stage('Containerize And Test') {
            steps {
                script{
                    sh 'docker run -d --name board_game_app sandya890/board_game && sleep 10 && docker stop board_game_app'
                }
            }
        }
        stage('Push Image To Dockerhub') {
            steps {
                script{
                    withCredentials([string(credentialsId: 'docker-cred', variable: 'docker-cred')]) {
                    sh 'docker login -u sandya890 --password ${docker-cred}' }
                    sh 'docker push sandya890/board_game:latest'
                }
            }
        }    
         stage("TRIVY Image Scan"){
            steps{
                sh "trivy image sandya890/board_game:latest > trivyimage.txt" 
            }
        }
    //     stage('Deploy to Kubernetes'){
    //         steps{
    //             script{
    //                 dir('Kubernetes') {
    //                     withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
    //                             sh 'kubectl apply -f deployment.yml'
    //                             sh 'kubectl apply -f service.yml'
    //                             sh 'kubectl get svc'
    //                             sh 'kubectl get all'
    //                     }   
    //                 }
    //             }
    //         }
        
       
      
       
    // }
}
}
