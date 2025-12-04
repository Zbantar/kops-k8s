pipeline {
    agent any
    parameters{
            string(name: 'IMAGE_TAG', defaultValue: 'latest', description: 'Docker tag')
    }
    tools {
        
        maven "maven3"
    }
    environment{
        SCANNER_HOME = tool "sonar-scanner"
    }
    stages {
        stage('git clone') {
            steps {
                echo 'clonning from the git repo'
                git branch: 'main', credentialsId: 'git-cred', url: 'https://github.com/Zbantar/Multi-Tier-BankApp-CI.git'
            }
        }
       stage('compilling') {
            steps {
              echo 'compiling the code'
              sh "mvn compile"
            }
        }
       stage('maven test') {
            steps {
               echo 'running test cases'
               sh "mvn test -DskipTests=true"
            }
        }
        stage('trivy scan') {
            steps {
              echo 'scanning the file system with trivy'
              sh "trivy fs --output table -o fs.html ."
               
            }
        }
        stage('codeQuality with sonarqube') {
            steps {
               withSonarQubeEnv('sonar-server') { 
               sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=bankapp -Dsonar.projectKey=bankapp \
                  -Dsonar.java.binaries=target'''
               }
            }
        }
     /*   stage('Qualitygate') {
            steps {
                timeout(2) {
                waitForQualityGate abortPipeline: true, credentialsId: 'sonar-cred'
              }
           }
        }
    */    
      stage('nexus') {
            steps {
               withMaven(globalMavenSettingsConfig: 'global-maven-settings', maven: 'maven3', traceability: true) {
                  sh "mvn deploy -DskipTests=true"
              }
            }
        }
/*    
       stage('docker build and tag') {
            steps {
               script{
                  withDockerRegistry(credentialsId: 'docker', url: 'https://index.docker.io/v1/') {
                  sh "docker build -t zbantar/myapp:${params.IMAGE_TAG} ."
                 }
              }
           }
       }
       
        stage('image scan with trivy') {
            steps {
               echo 'scanning the image with trivy...'
                 sh "trivy image --format table -o dimage.html  zbantar/bankapp:${params.IMAGE_TAG}"
            }
        }
           
         stage('docker push') {
            steps {
               script{
                  withDockerRegistry(credentialsId: 'docker', url: 'https://index.docker.io/v1/') {
                       echo 'pushing the image to the docker registry' 
                          sh "docker push  zbantar/myapp:${params.IMAGE_TAG}"
                 } 
               }
            }
        }
          stage('Update Yaml manifest in other Repo') {
            steps {
             script {
                withCredentials([gitUsernamePassword(credentialsId: 'git-cred', gitToolName: 'Default')]) {

                sh '''
                    echo "Cleaning old repo..."
                    rm -rf Multi-Tier-BankApp-CD
                    
                    echo 'Cloning deployment repo...'
                    git clone https://github.com/Zbantar/Multi-Tier-BankApp-CD.git
                '''

                sh '''
                    echo 'Navigating into repo...'
                    cd Multi-Tier-BankApp-CD

                    echo 'Current files in bankapp folder:'
                    ls -l bankapp
                '''

                sh '''
                    echo 'Updating YAML manifest file...'
                    cd Multi-Tier-BankApp-CD

                    repo_dir=\$(pwd)

                    # Update the image tag in bankapp-ds.yml
                    sed -i 's|image: .*|image: zbantar/myapp:${IMAGE_TAG}|g' bankapp/bankapp-ds.yml
                '''

                sh '''
                    echo "Updated YAML file contents:"
                    cat Multi-Tier-BankApp-CD/bankapp/bankapp-ds.yml
                '''

                sh '''
                    echo 'Configuring Git user...'
                    cd Multi-Tier-BankApp-CD
                    git config user.email "zackaribantar@yahoo.com"
                    git config user.name "Zbantar"

                    echo 'Committing and pushing changes...'
                    git add bankapp/bankapp-ds.yml
                    git commit -m "Update bankapp image tag to ${IMAGE_TAG}"
                    git push origin main
                '''
            }
        }
     }
   }
*/
 }
}
