pipeline {
    agent any

    parameters {
        string(name: 'IMAGE_TAG', defaultValue: 'latest', description: 'Docker tag')
    }

    tools {
        maven "maven3"
    }

    environment {
        SCANNER_HOME = tool "sonar-scanner"
    }

    stages {

        stage('git clone') {
            steps {
                echo 'Cloning from the git repo'
                git branch: 'main', credentialsId: 'git-cred', url: 'https://github.com/Zbantar/Multi-Tier-BankApp-CI.git'
            }
        }

        stage('compiling') {
            steps {
                sh "mvn compile"
            }
        }

        stage('maven test') {
            steps {
                sh "mvn test -DskipTests=true"
            }
        }

        stage('trivy scan') {
            steps {
                sh "trivy fs --format table -o fs.html ."
            }
        }

        stage('codeQuality with sonarqube') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh """
                        ${SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectName=bankapp \
                        -Dsonar.projectKey=bankapp \
                        -Dsonar.java.binaries=target
                    """
                }
            }
        }

        stage('nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-settings', maven: 'maven3') {
                    sh "mvn deploy -DskipTests=true"
                }
            }
        }

        stage('docker build and tag') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', url: 'https://index.docker.io/v1/') {
                        sh "docker build -t zbantar/myapp:${params.IMAGE_TAG} ."
                    }
                }
            }
        }

        stage('docker push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', url: 'https://index.docker.io/v1/') {
                        sh "docker push zbantar/myapp:${params.IMAGE_TAG}"
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

                            echo "Cloning deployment repo..."
                            git clone https://github.com/Zbantar/Multi-Tier-BankApp-CD.git
                        '''

                        sh '''
                            echo "Checking repo structure..."
                            cd Multi-Tier-BankApp-CD
                            ls -l bankapp
                        '''

                        sh '''
                            echo "Before updating YAML:"
                            cat Multi-Tier-BankApp-CD/bankapp/bankapp-ds.yml
                        '''

                        sh """
                            echo "Updating YAML file..."
                            cd Multi-Tier-BankApp-CD
                            sed -i "s|image: .*|image: zbantar/myapp:${IMAGE_TAG}|g" bankapp/bankapp-ds.yml
                        """

                        sh '''
                            echo "After updating YAML:"
                            cat Multi-Tier-BankApp-CD/bankapp/bankapp-ds.yml
                        '''

                        sh """
                            echo "Configuring Git user..."
                            cd Multi-Tier-BankApp-CD
                            git config user.email "zackaribantar@yahoo.com"
                            git config user.name "Zbantar"

                            echo "Adding file..."
                            git add bankapp/bankapp-ds.yml

                            echo "Checking for changes..."
                            if git diff --cached --quiet; then
                                echo "No changes detected. Skipping commit and push."
                            else
                                echo "Committing and pushing..."
                                git commit -m "Update bankapp image tag to ${IMAGE_TAG}"
                                git push origin main
                            fi
                        """
                    }
                }
            }
        }

    }
}
