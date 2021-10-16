pipeline {
    agent any

    tools {
        maven 'Maven'
        nodejs 'NodeJS'
    }
  
    stages {
        stage('Iniciando...'){
            steps{
             sh '''
              echo "PATH = ${PATH}"
              echo "M2_HOME = ${M2_HOME}"
              '''
            }
        }
        
        stage('Compilando...'){
            steps{
                sh 'mvn clean compile -e'
            }
        }
        
        stage('Testing...'){
            steps{
                sh 'mvn clean test -e'
            }
        }
        
        stage('Análisis de código estático (SCA)...'){
            steps{
                sh 'mvn org.owasp:dependency-check-maven:check'
                
                archiveArtifacts artifacts: 'target/dependency-check-report.html', followSymlinks: false
            }
        }
        
        stage('Checkeo de seguridad de aplicaciones estáticas (SAST)...'){
            steps{
                script{
                    def scannerHome = tool 'SonarQube'
                    
                    withSonarQubeEnv('SonarQube'){
                        sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=devsecops -Dsonar.sources=. -Dsonar.java.binaries=target/classes   -Dsonar.host.url=http://192.168.70.146:9001 -Dsonar.login=4cb8af5218a018b94fb22a8b49889c8464e921a6"
                    }
                }
            }
        }
        
    }
    
}
