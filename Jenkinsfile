pipeline {
    agent any
    tools {
      nodejs 'node_20.x'
    }

    stages {
        stage('GitHub Clone') {
            steps {
                echo "Hello"
                    git branch: 'master',changelog: false, poll: false, url: 'https://github.com/vinay-3708/juice-shop.git'
            }
        }
        stage('Build') {
            steps {
               sh '''
                    node -v
                    npm -v
                    ls -al
                    npm install
                    npm install -g @angular/cli
                    CHROME_BIN=/usr/bin/google-chrome-stable
                    echo $CHROME_BIN
                    npm test
               '''
            }
        }
        stage('SonarQube Scans') {
            steps {
                withSonarQubeEnv(credentialsId: 'SonarQube-Token', installationName: 'SonarQube') {
                    sh '''
                        sonar-scanner -Dsonar.projectKey=example1 -Dsonar.sources=. -Dsonar.typescript.lcov.reportPaths=build/reports/coverage/server-tests/lcov.info
                    '''
                }
            }
        }
        stage('Docker Deployment') {
            steps {
                script {
                    def container_id=sh(script:"docker ps | grep juice-shop-app | awk -F ' ' '{print \$1}'", returnStdout: true)
                
                    sh """
                        docker build -t juice-shop-app:${BUILD_NUMBER} .
                        docker stop ${container_id}
                        docker rm ${container_id}
                        docker run --name=juice-shop-app -d -p 80:3000 juice-shop-app:${BUILD_NUMBER}
                        echo "Waiting for 2 mins to become the app fully up & Running......."
                        sleep 120
                        
                    """
                }
            }
        }
        stage('OWASP ZAP Scans') {
            steps {
                sh """
                    java -jar /juice-shop/zap/ZAP_2.13.0/zap-2.13.0.jar -cmd -port 8081 -quickurl http://54.173.123.134/ -quickout ./owasp_zap.html -quickprogress
                """
            }
        }
    }
    post {
        always {
            archiveArtifacts artifacts: 'owasp_zap.html', followSymlinks: false
            cleanWs()
        }
    }
}
