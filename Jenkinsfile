pipeline {
  agent any
  environment {
    registryCredential = 'MyDockerHubCreds'
	registry = "semigr/goappsrepo"
  }
  options {
    buildDiscarder(logRotator(numToKeepStr: '5'))
  }
  stages {
    stage('Build') {
      steps {
        sh 'docker build -t $registry + ":latest" .'
      }
    }
    stage('Publish') {
        environment {
            registryCredential = 'MyDockerHubCreds'
        }
        steps{
            script {
                def appimage = docker.build registry + ":$BUILD_NUMBER"
                docker.withRegistry( '', registryCredential ) {
                    appimage.push()
                    appimage.push('latest')
                }
            }
        }
    }
    stage ('Deploy') {
        steps {
            script{
                def image_id = registry + ":$BUILD_NUMBER"
				sh "sed -i s#JUSTIMAGEPLACEHOLDER#${image_id}#g deployment.yml"
	            sh "source /var/lib/jenkins/.bash_profile > /dev/null && kubectl apply -f deployment.yml"
                sh "source /var/lib/jenkins/.bash_profile > /dev/null && kubectl apply -f service.yml"
            }
        }
    }
  }
  post {
    always {
      sh 'docker logout'
    }
  }
}