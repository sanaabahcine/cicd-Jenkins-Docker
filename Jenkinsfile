def COLOR_MAP = [
    'SUCCESS': 'good', 
    'FAILURE': 'danger',
]

pipeline {

    agent any

	tools {
            jdk 'OracleJDK8'
	    maven 'MAVEN3'	
		
    }

    environment {
        registry = "sanaeabahcine371/vproappdock"
        registryCredential = 'dockerhub'
    }

    stages{

        stage('BUILD'){
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }

      
  



        stage('CODE ANALYSIS with SONARQUBE') {

            environment {
                scannerHome = tool 'sonar4.7'
            }

            steps {
                withSonarQubeEnv('sonar-pro') {
                    sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile-repo \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }

            }
        }

        stage('Building image') {
            steps{
              script {
                dockerImage = docker.build registry + ":$BUILD_NUMBER"
              }
            }
        }
        
        stage('Deploy Image') {
          steps{
            script {
              docker.withRegistry( '', registryCredential ) {
                dockerImage.push("$BUILD_NUMBER")
                dockerImage.push('latest')
              }
            }
          }
        }

        stage('Remove Unused docker image') {
          steps{
            sh "docker rmi $registry:$BUILD_NUMBER"
          }
        }

 
    }


}
