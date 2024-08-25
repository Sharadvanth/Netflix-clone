pipeline {
    agent any
    
    tools {
        jdk 'jdk17'
        nodejs 'node22'
    }
    
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }

    stages {
        stage('Git_Checkout') {
            steps {
                git branch: 'main', 
                changelog: false, 
                poll: false, 
                url: 'https://github.com/Sharadvanth/Netflix-clone.git'
            }
        }
        stage('Sonarqube_Analysis') {
            steps {
                  withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix \
                    -Dsonar.projectKey=Netflix '''
                  }
            }
        }
        stage('Sonarqube_QualityGate') {
            steps {
                    script {
                     waitForQualityGate abortPipeline: false, credentialsId: 'sonarqube'
                        }
                  }
        }
        stage('Install_Dependencies') {
            steps {
                    sh 'npm install'
                  }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
          }
        }
        stage('Docker Build & Push') {
            steps {
              script{
                  // This step should not normally be used in your script. Consult the inline help for details.
            withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        // some block
                        sh 'docker build --build-arg TMDB_V3_API_KEY=891424b08e48c0121805f950390be516 -t netflix .'
                        sh 'docker tag netflix sharadvanth/netflix:latest'
                        sh 'docker push sharadvanth/netflix:latest'
                    }
              }
          }
        }
        stage('TRIVY Image SCAN') {
            steps {
                sh "trivy image sharadvanth/netflix:latest > trivyimage.txt"
          }
        }
        
        stage('Deploy to Container') {
            steps {
                sh "docker run -d -p 8081:80 sharadvanth/netflix:latest"
          }
        }
    
        }
        
    post {
     always {
        emailext (
            subject: "Pipeline status: ${BUILD_NUMBER}",
            body: ''' <html>
                        <body>
                            <p>Build Status: ${BUILD_STATUS} </p>
                            <p>Build Number: ${BUILD_NUMBER} </p>
                            <p>Check the <a href='"${BUILD_URL}"> console output </a>.</p>
                        </body>
                      </html> ''',
                    
                    to: 'sharadvanth1@gmail.com',
                    from: 'jenkinsbuild@gmail.com',
                    replyTo: 'jenkinsbuild@gmail.com',
                    mimeType: 'text/html'
                            
            )

        } 
    }

}
