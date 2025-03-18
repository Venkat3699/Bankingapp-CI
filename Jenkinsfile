pipeline {
    agent any
    
    tools {
        maven 'maven3'
    }
    
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }

    stages {
        stage('Clean WorkSpace') {
            steps {
                cleanWs()
            }
        }
        
        stage('Code CheckOut') {
            steps {
                git branch: 'master', url: 'https://github.com/Venkat3699/Bankingapp-CI.git'
            }
        }
        
        stage('Code Compile') {
            steps {
                sh 'mvn compile'
            }
        }
        
        stage('Unit Test') {
            steps {
                sh 'mvn test -DskipTests=true'
            }
        }
        
        stage('Trivy File System Scan') {
            steps {
                sh 'trivy fs --format table -o fs.html .'
            }
        }
        
        stage ("Sonar Scan") {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh " $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=bankapp -Dsonar.projectKey=bankapp -Dsonar.java.binaries=target "
                }
            }
        }
        
        stage('Building & Publish to Nexus') {
            steps {
                script {
                    withMaven(globalMavenSettingsConfig: 'maven-setting-nexus', jdk: '', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                        sh "mvn deploy -DskipTests=true"
                    }
                }
            }
        }
        
        stage('Docker Build & Tag') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker build -t ravisree900/bankapp:${env.BUILD_NUMBER} ."
                    }
                }
            }
        }
        
        stage('Docker Image Scan') {
            steps {
                sh "trivy image --format table -o dimage.html ravisree900/bankapp:${env.BUILD_NUMBER}"
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker push ravisree900/bankapp:${env.BUILD_NUMBER}"
                    }
                }
            }
        }

        stage('Update manifest file') {
            steps {
                script {
                    withCredentials([gitUsernamePassword(credentialsId: 'github-cred', gitToolName: 'Default')]) {
                        sh '''
                            git clone https://github.com/Venkat3699/Bankingapp-CD.git
                            cd Bankingapp-CD

                            # List files to confirm the presence of bankapp-ds.yml
                            ls -l bankapp

                            # Get the absolute path of the current directory
                            repo_dir=$(pwd)

                            # Change the image name using sed
                            sed -i 's|image: adijaiswal/bankapp:.*|image: ravisree900/bankapp:'"$BUILD_NUMBER"'|' ${repo_dir}/bankapp/bankapp-ds.yml
                        '''

                        // confirm the Changes
                        sh '''
                            echo "updated manifest files"
                            cat Bankingapp-CD/bankapp/bankapp-ds.yml
                        '''
                        // Configuring Git for commiting the changes
                        sh '''
                            cd Bankingapp-CD
                            git config user.email "ravindrareddy.mukka9@gmail.com"
                            git config user.name "Venkat3699"
                        '''
                        // Commit & Push the changes into GitHub Repository

                        sh '''
                            cd Bankingapp-CD
                            ls
                            git add bankapp/bankapp-ds.yml
                            git commit -m "Updated image tag"
                            git push origin master
                        '''
                    }
                }
            }
        }
    }
}
