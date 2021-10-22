pipeline {
    agent any

    tools {
        maven 'Maven'
        nodejs 'NodeJS'
    }
  
    stages {
        stage('Inicializando...'){
		
            steps{
            figlet 'Initial'
             sh '''
              echo "PATH = ${PATH}"
              echo "M2_HOME = ${M2_HOME}"
              '''
            }
        }
        
        stage('Compilación...'){
		
            steps{
	    figlet 'Compile'
                sh 'mvn clean compile -e'
            }
        }
        
        stage('Testing...'){
		
            steps{
	    figlet 'Testing'
                sh 'mvn clean test -e'
            }
        }
        
        stage('Análisis de código estático (SCA)...'){
		
            steps{
		figlet 'SCA'
                sh 'mvn org.owasp:dependency-check-maven:check'
                
                archiveArtifacts artifacts: 'target/dependency-check-report.html', followSymlinks: false
            }
        }

        
        stage('Checkeo de seguridad de aplicaciones estáticas (SAST)...'){
		
            steps{
		figlet 'SAST'
                script{
                    def scannerHome = tool 'SonarQube'
                    
                    withSonarQubeEnv('SonarQube'){
                        sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=devsecops -Dsonar.sources=. -Dsonar.java.binaries=target/classes   -Dsonar.host.url=http://192.168.70.146:9001 -Dsonar.login=4cb8af5218a018b94fb22a8b49889c8464e921a6"
                    }
                }
            }
        }
	
        stage('Seguridad de contenedores'){
    		steps{
    			figlet 'Seguridad de contenedores'
				script{
    				env.DOCKER = tool "Docker"
					env.DOCKER_EXEC = "${DOCKER}/bin/docker"
					sh '''
					${DOCKER_EXEC} run --rm -v $(pwd):/root/.cache/ aquasec/trivy openjdk:8-jdk-alpine
					'''
					sh '${DOCKER_EXEC} rmi aquasec/trivy'
					}
				}
            }	    
        
    

        stage('Owasp Zap DAST'){
     		steps{
			figlet 'Owasp Zap DAST'
			script{
        		    env.DOCKER = tool "Docker"
			    env.DOCKER_EXEC = "${DOCKER}/bin/docker"
			    env.TARGET = 'http://zero.webappsecurity.com'
			        		    
			    sh '${DOCKER_EXEC} rm -f zap2'
        		    sh "${DOCKER_EXEC} pull owasp/zap2docker-stable"
                                sh '${DOCKER_EXEC} run --add-host="localhost:192.168.70.146" --rm -e LC_ALL=C.UTF-8 -e LANG=C.UTF-8 --name zap2 -u zap -p 8093:8093 -d owasp/zap2docker-stable zap.sh -daemon -port 8093 -host 0.0.0.0 -config api.disablekey=true'
                                sh '${DOCKER_EXEC} run --add-host="localhost:192.168.70.146" -v /home/devsecops1313/Desktop/Jenkins/jenkins_home/tools:/zap/wrk/:rw -u zap --rm -i owasp/zap2docker-stable zap-full-scan.py -t "http://zero.webappsecurity.com" -I -r zap_full-scan_report.html -l PASS'	
					publishHTML([
        					allowMissing: false,
        				    alwaysLinkToLastBuild: false,
        				    keepAll: false,
        				    reportDir: '/var/jenkins_home/tools/',
        				    reportFiles: 'zap_full-scan_report.html',
        				    reportName: 'HTML Report',
        				    reportTitles: ''])
							}
						}
					}
        
            
		stage('Webhook Slack'){
    		steps{
			figlet 'Webhook Slack'
    			slackSend channel: 'devsecops-channel',
				color: 'good',
				message: "Se ha terminado una ejecucion del pipeline."
				}
			}
		}
}
